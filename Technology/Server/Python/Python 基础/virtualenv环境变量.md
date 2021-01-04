## virtualenv 安装和配置
- 隔离不同的开发环境。python 默认所有包都是放在同一个目录下的，这样就会造成一个问题，不同项目之前使用的程序版本不同容易造成混乱，python是通过虚拟环境来解决此问题的。（在Java中不同的jar包通过Maven会放置到不同的目录，然后在pom文件中来配置依赖。）
- 通过Windows的cmd命令来安装配置文件
``` cmd
安装虚拟环境工具：pip install virtualenv
新建虚拟环境：virtualenv testvir
进入虚拟环境: 
    cd testvir
    cd Script
    activate.bat
查看当前虚拟环境安装的库:pip list
退出虚拟环境：deactivate.bat
```
使用以上方式就可以新建虚拟环境了。但是存在一个问题，我们必须要记住虚拟环境的安装目录，如何解决？使用 virtualevnwrapper-win
### virtualevnwrapper 使用
- 安装 virtualevnwrapper-win （Linux下不用输入-win）.virtualevnwrapper会将我们所有的虚拟环境放置到同一个目录(Envs)中，这样我们就不用记虚拟环境的目录了。
```cmd
安装virtualevnwrapper：pip install virtualevnwrapper-win
新建虚拟环境：mkvirtualenv testvir2
退出虚拟环境：deactivate.bat
删除虚拟环境：rmvirtualenv testvir2
```
- 查看虚拟环境：workon
- 进入虚拟环境：workon testvir2
- 退出虚拟环境：deactivate.bat