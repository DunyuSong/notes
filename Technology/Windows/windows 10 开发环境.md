# windows 10 开发环境
## docker安装
1. dockert在windows上安装上会缺少很多东西，不建议在windows上安装。但是可以在windows上远程链接到安装了docket的机器上进行开发。
2. Docker开启Remote API 访问 2375端口
```
vim usr/lib/systemd/system/docker.service
修改配置
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock -H tcp://0.0.0.0:2375
重启服务
systemctl daemon-reload
systemctl restart docker
```
3. IDEA安装Docker插件。Setting->Plugins中搜索安装即可
4. IDEA配置Docker远程连接，File--Settings--Build--Docker--Connect to Docker daemon with:使用TCP socket里面配置"tcp://192.168.163.41:2375", 点击apply，下面显示Connection successful及表示已经连接到远程机器的2375端口了。就可以进行docker的部署，运行了。
5. 运行Docker程序：Run/Debug Configurations--添加Dockerfile运行方式。
```
Server:Docker
Dockerfile:Dockefile
Context folder:.
image tag:自己写
勾选Run built image,
Container name:自己写
publish exposed ports to the host interfaces:Specify,点击文件夹图标添加Prot Bindings。 Host port：是对外访问的端口。Container port:这个是你的应用jar的端口；protocol：tcp；Host IP：localhost
Bind mounts: Host path:/var/run/docker.sock; Container path:/var/run/docker.sock。这里主要是为了挂在docker引擎，否则无法运行。
Before launch: 可以选择一个需要的命令，比如 Run Maven Goal:clean package -U -DskipTests.
```
6. 然后就可以通过idea运行docker了。
7. 可以ssh远程到服务器上就可以看到运行的容器了docker ps