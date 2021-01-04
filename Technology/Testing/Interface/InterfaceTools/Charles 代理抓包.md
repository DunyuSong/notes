# Charles

## 2018年9月18日_Charles安装
- 官方下载：https://www.charlesproxy.com/download/
- charles系列破解激活办法（最高charles4.2都可以激活）
Charles Proxy License，适用于Charles任意版本的注册码
```
Registered Name: https://zhile.io
License Key: 48891cf209c6d32bf4
```
- HTTPS抓包配置：https://www.jianshu.com/p/7a88617ce80b 
1. 配置HTTP代理，这步与抓取HTTP请求是一样的.<br/>
![](http://cloudnotes.nos-eastchina1.126.net/20180918111600-506999.jpg)
2. 配置SSL代理，在charles的 Proxy选项选择SSL Proxing Settings，点add添加需要监视的域名，支持 *号通配符，端口一般都是443。<br/>
![](http://cloudnotes.nos-eastchina1.126.net/20180918111817-848078.jpg)
3. 在手机无线中配置手动代理，输入安装Charles的电脑的网络地址，端口填8888。
4. 在手机上安装Charles的根证书：书下载网址： chls.pro/ssl 。
5. 电脑端的根证书安装，Charles的Help菜单--SSL Proxying-->Install Charles Root Certicate.

## Charles安装(2018年5月14日)
- 官网下载地址：https://www.charlesproxy.com/
官方版为试用版，启动时有10秒等待，每隔30min会有提示，这个版本emmmmm~试用一下就好
- 绿色版下载（网上有很多相关下载地址）
    - For Windows：https://www.7down.com/soft/133829.html
    - For Mac：http://www.xue51.com/mac/2527.html

- 本人私藏版下载（For Windows V4.0.2）
    - 链接: https://pan.baidu.com/s/1OeBJUB4fbfOPSkkI-acLzQ 
    - 密码: zba5
    - 【注】安装完后，替换”安装路径->Charles\lib”文件夹下的charles.jar文件成破解版jar文件，如果再次启动未弹出30天试用的提示，说明破解成功。

### Charles代理配置使用
【注】这一步的目的是为了移动设备连接到Charles，这样移动设备发起的所有请求才能在Charles中看到。以下所有演示截图皆来自Android设备，iOS设备大同小异。

1. 使用Charles工具查看PC本地IP和端口号（端口号默认为8888，也可自行修改），选择“Help->Local IP Address”
2. 查看默认端口号“Proxy->Proxy Settings”
3. 长按Android设备当前连接的WiFi，选择”Modify network”->”Advanced options”->“Proxy”->”Manual”，输入”Proxy hostname”和”Proxy port”（即上一步查看的IP地址和端口号）然后点击保存，Charles会弹出connection确认弹窗，选择”Allow”。
4. 注意，如果首次连接时，Charles未出现该提示，手机随便开个一个网站后Charles的PC客户端会提示是否添加，选择允许即可。或者手动进入Charles的设置选项，添加当前手机的IP，选择“Proxy->Access Control Settings”，点击“Add”手动添加IP.

###  移动设备证书安装
上述操作只是完成了代理配置的一半，想要Charles抓到包还需要在移动设备上安装证书。
1. 手机浏览器访问链接http://www.charlesproxy.com/getssl/（链接名字长，注意不要写错），也可以在Charles中查看下载证书的地址（help--SSL Proxying--install charles root Certificate On Mobile Deveice...）
2. 安装动态证书（证书名称随意填写即可），凭据用途默认即可，证书安装时需要设置系统锁，设置锁屏成功后证书会提示证书安装成功。到此，你的设备和Charles就建立了连接，可以尝试在设备上访问一个地址，在Charles左侧视图就能实时看到请求啦~但你会发现，有些请求为什么会显示成红叉unknown呢？请接着往下看~


## 基础功能使用介绍
### HTTPS解密抓包
1. 对于https的请求，解密抓包时需要电脑安装证书，选择“Help->SSL Proxying->Install Charles Root->Certificate ”：点击“安装证书”,默认下一步即可完成。
2. 证书安装完成后,需要将解密的URL添加到”SSL Proxying”中,”选择左侧的Structure视图中的HTTPS请求右键--Enable SSL Proxying，此时再查看https的请求,就显示正常了：
参考:https://www.cnblogs.com/ceshijiagoushi/p/6812493.html 

### Focus功能
对请求进行分组，当请求过多时左侧域名显示杂乱，可以右键”focus”你需要的请求，这样，URL就会被大致分组，想找的URL显而易见。

### Repeat功能
可以实现一次重复请求


## 删除 chrales配置文件
C:\Users\shandongdong\AppData\Roaming\Charles


## 参考：
-