- 原地址：https://www.cnblogs.com/liaojie970/p/5527105.html 
### mysql JDBC URL格式如下：

- jdbc:mysql://[host:port],[host:port].../[database][?参数名1][=参数值1][&参数名2][=参数值2]...
- 对应中文环境，通常mysql连接URL可以设置为：

jdbc:mysql://localhost:3306/test?user=root&password=&useUnicode=true&characterEncoding=gbk&autoReconnect=true&failOverReadOnly=false

- 在使用数据库连接池的情况下，最好设置如下两个参数：

autoReconnect=true&failOverReadOnly=false

- 需要注意的是，在xml配置文件中，url中的&符号需要转义成“&amp;”。比如在tomcat的server.xml中配置数据库连接池时，mysql jdbc url样例如下：

jdbc:mysql://localhost:3306/test?user=root&amp;password=&amp;useUnicode=true&amp;characterEncoding=gbk

&amp;autoReconnect=true&amp;failOverReadOnly=false

