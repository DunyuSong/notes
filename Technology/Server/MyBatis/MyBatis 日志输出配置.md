
### 打印Debug日志，这样就可以查看执行的SQL语句
``` log4j.properties
# 第一个字段INFO/DEBUG指定优先级，只能指定一个字段。第二个字段和第三个字段，用来指定日志输出的目的地。Console代表输出到控制台，File代表输出到文件。如果只有一个输出目的地，我们也可以只指定一个字段。
log4j.rootLogger=INFO,Console,File
#定义日志输出目的地为控制台
log4j.appender.Console=org.apache.log4j.ConsoleAppender
log4j.appender.Console.Target=System.out
#可以灵活地指定日志输出格式，下面一行是指定具体的格式
log4j.appender.Console.layout=org.apache.log4j.PatternLayout
log4j.appender.Console.layout.ConversionPattern=[%c] - %m%n

#打印sql部分
log4j.logger.java.sql=DEBUG
log4j.logger.java.sql.Connection=DEBUG  
log4j.logger.java.sql.Statement=DEBUG  
log4j.logger.java.sql.PreparedStatement=DEBUG  
log4j.logger.java.sql.ResultSet=DEBUG
#配置logger扫描的包路径这样才会打印sql
log4j.logger.com.test.dao=DEBUG
```
- 例子
```log
[com.test.dao.EmployeeMapper.deleteEmployeeById] - ==>  Preparing: delete from tbl_employee where id = ? 
[com.test.dao.EmployeeMapper.deleteEmployeeById] - ==> Parameters: 10005(Integer)
[com.test.dao.EmployeeMapper.deleteEmployeeById] - <==    Updates: 1
```
==>代表发送给出去的，<==表示返回结果