
## 安装
### CentOS 下 安装 tomcat 8
1. 复制下载链接tar.gz源码:(https://tomcat.apache.org/download-80.cgi)，右键复制链接地址
2. 指定下载的目录：cd ~
3. 下载:wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-8/v8.5.42/bin/apache-tomcat-8.5.42.tar.gz
4. 创建解压目录：mkdir /opt/tomcat
5. 解压到指定目录：tar -zxvf apache-tomcat-8.5.42.tar.gz -C /opt/tomcat --strip-components=1
6. 
> 参考：https://www.vultr.com/docs/how-to-install-apache-tomcat-8-on-centos-7
## 启动tomcat


### Windows安装
- 下载zip包后解压配置环境变量"CATALINA_HOME":"D:\apache-tomcat-8.0.24"，JAVA_HOME如果没有配置也需要配置一下。-->starup.bat 双击就启动了。
    - 问题：提示服务没有开启：service.bat install
- 启动Tomcat：cmd-->startup.bat
- 访问Tomcat: http://localhost:8080。显示Tomcat页面表示服务正常运行。
## 配置
### 如何共享目录
- tomcat默认访问的是webapps/ROOT目录， 在ROOT目录下创建一个test目录，修改 conf/web.xml 文件的false为true即可
```
        <init-param>
            <param-name>listings</param-name>
	    <param-value>true</param-value>
        </init-param>
```
### Tomcat9命令行启动后中文乱码
- 修改"D:\software\apache-tomcat-9.0.17\conf\logging.properties"中“java.util.logging.ConsoleHandler.encoding = UTF-8”为“java.util.logging.ConsoleHandler.encoding = GBK”。
- 

