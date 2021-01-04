# Quartz 学习笔记
## 学习资料
- [Quartz官方文档翻译](https://www.w3cschool.cn/quartz_doc/?)
- [Quartz专栏](https://aflyun.blog.csdn.net/column/info/14251), [Spring+SpringMVC+Mybatis+Mysql)和Quartz集成](https://blog.csdn.net/u010648555/article/details/60767633)
- [quartz （从原理到应用）详解篇](https://my.oschina.net/songhongxu/blog/802574)
- [x] ~~[基于spring-boot 2.x + quartz 的CRUD任务管理系](https://gitee.com/52itstyle/spring-boot-task)~~
- [x] ~~[SpringBoot + Spring MVC + Quartz + MySQL 学习项目](https://github.com/andy-zhang-guohua/springboot-quartz-mysql)~~~
- [Spring整合Quartz基于数据库的分布式定时任务，可动态添加、删除、修改定时任务](https://github.com/ameizi/spring-quartz-cluster-sample)

## 问题记录
### Quartz数据是如何入库的？
quartz 数据入库没有对应的mapper.xml，而是在代码中直接写死了。可以通过sched.scheduleJob(job, trigger)进去查看具体保存job和trigger的源码“org.quartz.impl.jdbcjobstore.StdJDBCDelegate#insertJobDetail”方法查看，所以我们建表语句必须使用quartz的建表语句，否则字段对应不上。
### 为什么服务器重启了Quartz的任务还能恢复执行？
quartz 自带有恢复机制，即使你把tomcat服务器重启了，已经开始的任务在下次tomcat启动的时候还是会重新执行，quartz会计算出服务器重启期间需要执行的次数。猜测是在启动的时候进行了qrtz_表前缀的扫描表，然后根据qrtz_triggers.trigger_state状态来计算，具体状态可以查看org.quartz.impl.jdbcjobstore.JobStoreSupport#getTriggerState(java.sql.Connection, org.quartz.TriggerKey)查看。REQUESTS_RECOVERY== 1
### Job是单实例的吗？
1.  我们写的每一个job都继承自Job，public class HelloWorldJob implements Job {}, 当 Scheduler.scheduleJob(job, trigger);时HelloWorldJob类每次都会实例化一次，可以通过添加构造方法查看。
2.  每一个Job都是单独的一个实例，包括了所需的数据，如果我们在job中使用“new JobDataMap”方式给trigger添加了参数，那么每个Job都是独立保存自己的数据的。可以通过表qrtz_triggers.job_data查看每个job保存的数据。
3.  业务代码一般写在Job.execute(){// 具体业务处理}方法中，如果需要数据传递，可以通过JobDataMap的方式，但是这里一般是存放基本数据类型。
### 如何进行同步执行？
1. 同步执行：默认Job都是异步进行执行的，如果需要使用同步方式执行那么需要在Job类上添加注解@DisallowConcurrentExecution @PersistJobDataAfterExecution。
2. 说明：@DisallowConcurrentExecution：将该注解加到job类上，告诉Quartz**不要并发**地执行同一个job定义（这里指特定的job类）的多个实例。该限制是针对JobDetail的，而不是job类的。@PersistJobDataAfterExecution：有状态的job，多次调用job的时候保存JobDataMap的数据。将该注解加在job类上，告诉Quartz在成功执行了job类的execute方法后（没有发生任何异常），更新JobDetail中JobDataMap的数据， 使得该job（即JobDetail）在下一次执行的时候，JobDataMap中是更新后的数据，而不是更新前的旧数据。 
3. 注意：如果你使用了@PersistJobDataAfterExecution注解，我们强烈建议你同时使用@DisallowConcurrentExecution注解，因为当同一个job（JobDetail）的两个实例被并发执行时，由于竞争，JobDataMap中存储的数据很可能是不确定的。

### 任务的运行状态
1. 获取任务状态：scheduler.getTriggerState(trigger.getKey());
2. 存储表字段：qrtz_trigger.trigger_state
3. 状态说明：
```
    // NONE 无或删除,
    // NORMAL 正常,
    // PAUSED 暂停,
    // COMPLETE 完成,
    // ERROR 错误,
    // BLOCKED 阻塞
```
- 代码"org.quartz.impl.jdbcjobstore.JobStoreSupport#getTriggerState(java.sql.Connection, org.quartz.TriggerKey)"
```java
    public TriggerState getTriggerState(Connection conn, TriggerKey key) throws JobPersistenceException {
        try {
            String ts = this.getDelegate().selectTriggerState(conn, key);
            if (ts == null) {
                return TriggerState.NONE;
            } else if (ts.equals("DELETED")) {
                return TriggerState.NONE;
            } else if (ts.equals("COMPLETE")) {
                return TriggerState.COMPLETE;
            } else if (ts.equals("PAUSED")) {
                return TriggerState.PAUSED;
            } else if (ts.equals("PAUSED_BLOCKED")) {
                return TriggerState.PAUSED;
            } else if (ts.equals("ERROR")) {
                return TriggerState.ERROR;
            } else {
                return ts.equals("BLOCKED") ? TriggerState.BLOCKED : TriggerState.NORMAL;
            }
        } catch (SQLException var4) {
            throw new JobPersistenceException("Couldn't determine state of trigger (" + key + "): " + var4.getMessage(), var4);
        }
    }
```
### 任务数大于线程数如何处理？
- quartz可以通过配置文件指定线程数量“org.quartz.threadPool.threadCount=2”
- 线程数量，同一时刻允许触发的执行Job的线程数。
- 例如：总共有10个Job，配置线程数为2，每个线程执行需要耗时2秒。那么现象为每一秒执行2个Job任务，每个job执行需要耗时2秒，等待2个job执行完之后，触发后面2个job的执行，后面两个job与前两个job相差时间为2秒。
- 可通过线程ID查看Thread.currentThread().getId()。需要注意的时，如果任务是不停歇重复执行时，当任务较多时会出现抢占CPU资源的问题，此时无法确定哪个线程会被先执行。比如有2个线程，10个任务，此时可能会出现有些任务一直在执行，有些任务永远执行不到。

### 普通触发器无法查询状态？
1. 通过SimpleTrigger创建的触发器在数据库里面是无法直接查找到的。
2. 刚刚触发的任务可能能看到，但是过一会任务执行了数据库的记录也没有了。
3. 如果是选择用立即触发会发现看不到执行的日志，通过添加等待来解决。
```java
   /**
     * 添加一个执行任务, 立即触发来执行，用于手动执行任务
     */
    public void addRunOnceJob(String jobName, String jobGroupName, String triggerName, String triggerGroupName,
                              String clazz, JobDataMap jobExtraDataMap) {
        try {
            Class cls = Class.forName(clazz);
            // 创建一项作业
            JobDetail job = JobBuilder.newJob(cls).withIdentity(jobName, jobGroupName).build();

            // 触发时间点
            SimpleScheduleBuilder simpleScheduleBuilder = SimpleScheduleBuilder.simpleSchedule().withRepeatCount(0);
            Date startDate = new Date();
            // 延迟0.5秒执行,防止出现priv_fire_time为-1的情况
            startDate.setTime(startDate.getTime() + 500);
            TriggerBuilder<SimpleTrigger> simpleTriggerTriggerBuilder = newTrigger().withIdentity(triggerName, triggerGroupName).withSchedule(simpleScheduleBuilder).startAt(startDate);
            if (null != jobExtraDataMap) {
                simpleTriggerTriggerBuilder.usingJobData(jobExtraDataMap);
            }
            Trigger trigger = simpleTriggerTriggerBuilder.build();

            // 指定时间开始触发，不重复
//            SimpleTrigger trigger =  SimpleScheduleBuilder.simpleSchedule().withRepeatCount(0).build();
//                    .withIdentity(triggerName, triggerGroupName)
//                    .startAt(new Date())
//                    .forJob(jobName, jobGroupName)
//                    .usingJobData(jobExtraDataMap)
//                    .build();

            scheduler.scheduleJob(job, trigger);
            // 启动
            if (!scheduler.isShutdown()) {
                scheduler.start();
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

```

## 工具
- [Cron在线表达式生成工具](https://www.pppet.net/)
