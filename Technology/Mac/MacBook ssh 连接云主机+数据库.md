# MacBook ssh 连接云主机+数据库
## 生成ssh方法
- 参考Github：https://help.github.com/articles/connecting-to-github-with-ssh/
- 方法：
1. ssh-keygen -t rsa -b 4096 -C "your_email@example.com"

## 连接云主机
1. 菜单栏item2--Preferences--profiles--点击“+”号。
2. 在Command连接云主机命令：ssh -i /Users/shandongdong/Documents/ssh/id_rsa -p 1046 shandongdong@10.122.58.122
> 参考：https://www.jianshu.com/p/d75dc43be7a7
## 连接数据库
1. 下载navicat(连接DDB用MySQL Workbench无法连接)：https://xclient.info/s/navicat-for-mysql.html?t=b43ff20a95f7a8f8c2453e68b2650326403e6b99#versions
2. 菜单栏item2--Preferences--profiles--点击“+”号。
2. 在Command连接云主机命令通过隧道方式：ssh -CNPf -L 10073:10.122.58.73:6000 -p 1046 shandongdong@59.111.178.139
> ssh -CNPf -L 隧道端口号:数据库ip:数据库端口号 -p 1046 用户名@跳板机ip
3. 打开navicat连接，Host：127.0.0.1。端口：10073.
## 连接云主机并且配置隧道
- 使用一下命令只能在item中选择send text a start:
```
ssh -i /Users/shandongdong/Documents/ssh/id_rsa -CNPf -L 10073:10.122.58.73:6000 -p 1046 shandongdong@59.111.178.139

ssh -i /Users/shandongdong/Documents/ssh/id_rsa -CNPf -L 10092:10.201.224.92:3306 -p 1046 shandongdong@59.111.178.139
```

## 结束上面命令后开启后的端口程序（lsof：list open file）
- 查看端口：lsof -i:10073
- 杀掉进程：kill -9 10073
- 

