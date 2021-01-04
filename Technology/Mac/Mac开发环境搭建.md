### item2 

### Maven
1. 下载安装包tar.gz：http://maven.apache.org/download.cgi
2. 移动安装包指定位置：mv apache-maven-3.6.0-bin.tar.gz /Users/shandongdong/Documents/software/
3. 解压： tar -xzvf apache-maven-3.0.6-bin.tar
4. vim ~/.bash_profile
5. 添加下列两行代码，之后保存并退出ViM：
```
export M2_HOME=/Users/shandongdong/Documents/software/apache-maven-3.6.0
export PATH=$PATH:$M2_HOME/bin
```
6. 输入命令以使bash_profile生效:$ source ~/.bash_profile
7. mvn -v查看Maven是否安装成功
8. 

### Idea 配置git
1. 下载：https://git-scm.com/download/mac
2. 查看软件安装位置：which git
3. idea--VersionContorl--Git to Git--path:/usr/local/bin/git
4. 

### Tomacat
1. 下载tar.gz安装包：https://tomcat.apache.org/download-90.cgi
2. 解压缩：tar zxfv apache-tomcat-9.0.14.tar.gz
3. 重命名：mv apache-tomcat-9.0.14.tar.gz tomcat9
4. 修改目录权限:输入命令：sudo chmod 755 /Users/shandongdong/Documents/software/tomcat9/bin/*.sh
5. 启动Tomcat:在终端中输入cd xxx (xxx是你tomcat/bin放至的路径)，再输入sudo sh startup.sh
6. 打开我们的浏览器，然后网址输入 http://localhost:8080/，如果出现一只猫，则证明配置成功~.
7. 关闭tomcat：同样是在bin 目录下，在终端输入：./shutdown.sh + 回车，就可以了。
8. 