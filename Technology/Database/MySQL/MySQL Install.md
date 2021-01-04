## CentOS6.8 上安装 MySQL 5.7
1. yum localinstall https://dev.mysql.com/get/mysql57-community-release-el6-9.noarch.rpm
2. yum install mysql-community-server
3. 安装MySQL会默认创建一个临时密码，通过命令查看：grep 'A temporary password' /var/log/mysqld.log |tail -1。大概后面会出现密码“root@localhost: Nm(!pKkkjo68e”
4. 启动mysql：service mysqld start
5. 登录MySQL：mysql -uroot -p
6. 修改MySQL密码等级：mysql> set global validate_password_policy=0;
7. 修改MySQL密码为test123456:mysql> ALTER USER 'root'@'localhost' IDENTIFIED BY 'test123456';
8. 修改MySQL连接权限(这样就可以远程通过IP连接):mysql> use mysql;
9. mysql> update user set host = '%' where user ='root';
10. 刷新MySQL:mysql> flush privileges;
11. 退出:mysql> quit
12. 完成安装后就可以通过客户端连接MySQL了
### 设置mysql不区分大小写
默认mysql在Windows上表名不区分大小写，但是Linux上区分。
1. vim /etc/my.cnf
2. 追加“lower_case_table_names=1”
3. 重启mysqld, service mysqld stop
4. service mysql start
### 参考文档
- https://tecadmin.net/install-mysql-5-7-centos-rhel/
- https://blog.csdn.net/qq_41266954/article/details/80858301


## Windows安装
- 安装5.x：https://www.jb51.net/article/147685.html
- 安装8.X：https://blog.csdn.net/qq_33236248/article/details/80046448
- navicat 连接 mysql 出现Client does not support authentication protocol requested by server解决方案
https://blog.csdn.net/u013700358/article/details/80306560