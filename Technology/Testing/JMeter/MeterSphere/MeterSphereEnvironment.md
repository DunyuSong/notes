# Environment
## Test Environment

1. [访问地址](http://192.168.163.41:8081/login)
2. 登入帐号密码：admin:metersphere
3. 机器：192.168.163.41; root:autotest#123; 
4. 安装方式：一键安装，全docker。模式：allinone，主副一体。
5. 安装目录:/opt
6. 接口文档: http://localhost:8081/swagger-ui/index.html?configUrl=/v3/api-docs/swagger-config

## Local Development Environment

1.  本地开发环境CentOS 7。 配置文件:/opt/metersphere/conf/metersphere.properties
2.  本地开发环境CentOS 7。 安装JMeter5.3(拷贝里面所有文件到/opt/jmeter目录下，并且用代码工程中的src/main/resources/jmeter/bin下的配置文件覆盖下载的JMeter配置文件/opt/jmeter/bin
    )
3.  服务器CentOS  7(192.168.163.41): 在 /usr/local 搭建了 JDK 1.8、、kafka_2.13-2.6.0、zookeeper-3.6.2、maven-3.6.3、docker、jenkins-2.263
4.  数据库192.168.163.43: 搭建了MySQL 5.7。创建了数据库metersphere_dev
5.  在服务器上创建Topic
```shell
创建2个topic
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic JMETER_METRICS
bin/kafka-topics.sh --create --zookeeper localhost:2181 --replication-factor 1 --partitions 1 --topic JMETER_LOGS
查看创建的topic
bin/kafka-topics.sh --list --zookeeper localhost:2181
```
6. 修改kafka的配置文件 config/server.properties。 添加端口访问
```properties
# 允许外部端口连接                                            
listeners=PLAINTEXT://0.0.0.0:9092
# 外部代理地址                                                
advertised.listeners=PLAINTEXT://192.168.163.41:9092
```
## Online Eenvionment
### DataBase mysql 5.7.26
- 192.168.9.31, root:autotest#123
- lib name: metersphere
### Front
- 192.168.9.32, root:autotest#123
- 
### Server
- 192.168.9.33, root:autotest#123
- Server: /opt/
### Node-Controller
- 192.168.9.34, root:autotest#123
    - 16G
- 192.168.9.35
- 192.168.9.36, root:autotest#123
    - 16G
- 192.168.9.37, root:autotest#123
    - 16G
- 192.168.9.38
- 192.168.9.39
- 192.168.9.40
- 192.168.9.41
- 192.168.9.42
- 192.168.9.43
- 192.168.9.44
## Debug 

- 查看接口运行日志： docker logs -f ms-server --tail 200
- 日志路径： /opt/metersphere/logs/
- 修改环境配置：vim /opt/metersphere/.env
- 重新加载服务：msctl reload