# Jedis
## 使用Java远程调用
1. 配置依赖
```xml
        <dependency>
            <groupId>redis.clients</groupId>
            <artifactId>jedis</artifactId>
            <version>3.1.0</version>
        </dependency>
```
2. 测试连通性
```java
public class Jedis01 {
    public static void main(String[] args) {
        Jedis jedis = new Jedis("192.168.163.42", 6379);
        System.out.println(jedis.ping());
    }
}
```
3. 默认配置可能连接不上参考下面错误解决方法。
4. 




# 常见错误
### Connection refused 拒绝访问
问题：redis.clients.jedis.exceptions.JedisConnectionException: java.net.ConnectException: Connection refused
解决：修改redis.conf配置文件,注释掉此行。"# bind 127.0.0.1"
### DENIED Redis is running in protected mode （保护模式）
问题：redis开启了protected mode，这也是Redis3.2加入的新特性，开启保护模式的redis只允许本机登录，同样设置在配置文件redis.conf中
解决：修改redis.conf配置文件中“protected-mode no” 为 “protected-mode yes”。
### SocketTimeoutException 连接超时
问题：可能是redis的6379端口无法访问，请先在cmd中输入命令 telnet 127.0.0.1 6379 如果不能访问，需要在远程机器关掉防火墙或者添加允许通过
解决：vim /etc/sysconfig/iptables  添加“-A INPUT -m state --state NEW -m tcp -p tcp --dport 6379  -j ACCEPT”
