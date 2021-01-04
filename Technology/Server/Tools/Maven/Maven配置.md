### 修改maven自动下载的jar包位置
打开\apache-maven-3.6.0\conf\settings.xml,添加下载目录“ <localRepository>D:/software/apache-maven-3.6.0/repo</localRepository>”
### maven下载镜像配置
打开\apache-maven-3.6.0\conf\settings.xml在<mirrors>标签内添加
```xml
	<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>       
	</mirror>
```