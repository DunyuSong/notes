# Redis (Remote Dictionary Server)
## Liunx 配置readis
### 安装
1. cd /opt
2. wget http://download.redis.io/releases/redis-5.0.5.tar.gz
3. tar zxvf redis-5.0.5.tar.gz
4. cd redis-5.0.5
5. make （注意：如果没有安装GCC编译器需要先安装：yum install gcc-c++，然后 make distclean 之后再 make。）
6. make install 

### 配置
1. cd /opt/redis-5.0.5
2. vim redis.conf
3. 查找：/daemonize
4. 修改：daemonize no ===> daemonize yes (设为为以后台服务启动)

### 启动 Redis
1. 进入命令路径：cd /usr/local/redis/bin
2. 启动服务：   redis-server /opt/redis-5.0.5/redis.conf
3. 连接服务：   redis-cli -p 6379   （连接成功后会进入Redis命令窗口）
4. 测试服务：   ping （出现PONG表示成功）
5. set key1 HelloWorld
6. get key1
7. 关闭服务：shutdown
8. 退出连接：exit

### 常用命令
1. 查看Redis进程是否启动：ps -ef|grep redis

## Redis 持久化
两种方式：RDB、AOF
### RDB（Redis DataBase）
- RDB方式会根据配置文件 /opt/redis-5.0.5/redis.conf 中 SNAPSHOTTING 配置的方式进行持久化数据到dump.rdb文件中，以便于Redis启动时从硬盘中恢复数据到内存中。
#### 是什么
1. 在指定的时间间隔内将内存中的数据集快照写入磁盘。
2. Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。RDB的缺点是最后一次持久化后的数据可能丢失。

#### 触发条件
- RDB是整个内存的压缩过的Snapshot，RDB的数据结构，可以配置复合的快照触发条件，
- 默认规则
是1分钟内改了1万次，
或5分钟内改了10次，
或15分钟内改了1次。
- 立即触发：当FLUSHALL、SHUTDOWN、sava、bgsave时会立即触发Redis的备份。

#### 优势
- 适合大规模的数据恢复
- 对数据完整性和一致性要求不高

#### 劣势
- 在一定间隔时间做一次备份，所以如果redis意外down掉的话，就会丢失最后一次快照后的所有修改
- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

### AOF（Append Only File）
- AOF方式会根据配置文件redis.conf中  APPEND ONLY MODE 配置的方式进行持久化数据到 appendonly.aof 文件中，aof默认会以每秒同步的规则进行将用户所有的写、更新操作完全同步到.aof文件中，以便于恢复用户所有的写操作。

#### 使用方式
- 开启aof持久化：redis.conf 中默认是关闭aof持久化的：appendonly no，改成yes即开启。
- 修复.aof文件：Redis-check-aof --fix 进行修复
- 恢复.aof文件：重启redis然后重新加载

### RDB VS AOF
1. RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。
2. AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大。
3. 只做缓存：如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 同时开启两种持久化方式
1. 在这种情况下,当redis重启的时候会优先载入AOF文件来恢复原始的数据。因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整. 
2. RDB的数据不实时，同时使用两者时服务器重启也只会找AOF文件。那要不要只使用AOF呢？作者建议不要，因为RDB更适合用于备份数据库(AOF在不断变化不好备份)，快速重启，而且不会有AOF可能潜在的bug，留着作为一个万一的手段。
- 性能建议
1. 因为RDB文件只用作后备用途，建议只在Slave上持久化RDB文件，而且只要15分钟备份一次就够了，只保留save 900 1这条规则。
2. 如果Enalbe AOF，好处是在最恶劣情况下也只会丢失不超过两秒数据，启动脚本较简单只load自己的AOF文件就可以了。代价一是带来了持续的IO，二是AOF rewrite的最后将rewrite过程中产生的新数据写到新文件造成的阻塞几乎是不可避免的。只要硬盘许可，应该尽量减少AOF rewrite的频率，AOF重写的基础大小默认值64M太小了，可以设到5G以上。默认超过原大小100%大小时重写可以改到适当的数值。
3. 如果不Enable AOF ，仅靠Master-Slave Replication 实现高可用性也可以。能省掉一大笔IO也减少了rewrite时带来的系统波动。代价是如果Master/Slave同时倒掉，会丢失十几分钟的数据，启动脚本也要比较两个Master/Slave中的RDB文件，载入较新的那个。

## 事务
- 悲观错：认为操作都可能会带来数据问题所以直接上锁。比如：锁表、锁行等
- 乐观错：认为操作不会带来数据问题，在更新、插入的时候会带上一个版本号，当发现版本号不一致时才会报错。
### 事务的常用命令
1. 全体连坐：一个错误，都不执行。这种一般是在执行命令期间执行了非法命令。
2. 冤头债主：事务提交后，对执行错误的抛出错误异常，其他的正常执行。
结合以上两点特性，所以Redis的应该算是部分支持事务。


## Redis的复制(Master/Slave)
- 也就是我们所说的主从复制，主机数据更新后根据配置和策略，自动同步到备机的master/slaver机制，Master以写为主，Slave以读为主。
### 使用
- 从库配置：slaveof 主库IP 主库端口
- 如果master挂了，那么 slaveB、那么 slaveC 还是默认保持 slav e的角色，不会从slave转为master，除非slaveB机器手动执行“slaveof on one”命令(反客为主)，那么会将 slaveB 转换为 master 角色。如果此时 slaveC 机器可以选择继续等待之前的master恢复或者选择重新 slavofB 机器作为自己的master，那么此时这两台机器单独组成一个主从关系。

### sentinel 哨兵模式
- 反客为主的自动版，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库
#### 使用
1. 新建sentinel.conf文件，名字绝不能错
2. 配置哨兵,填写内容： sentinel monitor 被监控数据库名字(自己起名字) 127.0.0.1 6379 1
3. 上面最后一个数字1，表示主机挂掉后salve投票看让谁接替成为主机，得票数多少后成为主机
4. 启动哨兵进行自动监控：Redis-sentinel /myredis/sentinel.conf 
5. 哨兵会一直监控着“127.0.0.1 6379”这台master主机，当发现master挂了之后会自动进行从下面的salve中选择一台机器作为mater，然后将其他salve重新挂载在自己下面作为salve。如果此时之前挂掉的master又恢复了，那么会将之前挂掉的master角色变为salve，然后挂载在新的master下面。
6. 哨兵可以监控多台master，只需要在配置文件中配置多台需要监控的机器即可。
7. 