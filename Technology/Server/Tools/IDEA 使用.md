# IDEA 配置
- 破解：https://tech.souyunku.com/?p=15076
## 快捷键
- Ctrl + Alt + L 格式化代码
- Shift + Shift  搜索类、资源、配置项、方法等
- Ctrl + H 查看类的继承关系
- sout 打印语句

## Debug源码
### 如何查看通过接口调用方法的具体实现方法
1. 在测试代码打上断点，点击debug运行。
2. 在运行到接口.方法时查看此接口的具体实现点击“Step Into(F7)”
3. 
### 快捷键
- F8(Step over)：单步执行
- Shift + F8(Step Out): 跳出当前方法
- F9(Resume Program)：跳转到下一个断点
- F7(Step into): 进入方法内部



## 配置

### 编码格式
#### 解决log4j在idea里乱码
1. IDEA安装目录的bin目录下找到名为"idea64.exe.vmoptions"的文件。
2. 末尾追加一行设置（-Dfile.encoding=UTF-8）。
3. 重启idea。log4j不在乱码，但是发现tomcat启动日志乱码了（修改IDEA的配置文件之前是不乱码的）。
4. Run-->Edit Configurations-->tomcat9-->Server-->VM optoins:添加"-Dfile.encoding=UTF-8".

#### 设置文件编码
1.File--Settings--Editor--File Encodings--全部设置为UTF-8。

### Maven
1. File-->setting-->Build-->Build Tools-->Maven-->Maven home directory选择本地maven目录。<br/>
![](http://cloudnotes.nos-eastchina1.126.net/20181105035758-269825.jpg)

### Git
1. 需先下载[Git](https://git-scm.com/downloads)
2. File-->New-->Project form version contorl-->Git
### SVN
1. Setting -->Subversin--> 选择svn的安装路径:\Program Files\TortoiseSVN\bin\svn.exe
### 类注释模板
1. 配置类模板：File-->settings-->Editor-->File and Code Templates-->Files-->选择Class-->输入一下代码
```
/**
* @Project: ${PROJECT_NAME}
*
* @Author: Miller
*
* @Time: ${YEAR}-${MONTH}-${DAY} ${HOUR}:${MINUTE}
* 
* @Email: miller.shan@aliyun.com;miller.shan.dd@gmail.com
*
* @Description: ${description}
**/
```
![](http://cloudnotes.nos-eastchina1.126.net/20181105040521-453960.jpg)
> 接口和枚举方法相同。
2. 生成方法注释
