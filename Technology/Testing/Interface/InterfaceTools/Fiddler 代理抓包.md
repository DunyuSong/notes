# Fiddler 抓包工具
## 简介
- Fiddler是一款用于抓取客户端与服务端通讯的工具。[官网](https://www.telerik.com/download/fiddler)
- 当启动fiddler后默认会修改浏览器的代理设置（代理--设置--打开代理设置--Internet属性--链接--区域网设置--代理服务器--会自动勾选代理）。
- 执行原理：打开浏览器代理高级可以发现fiddler默认将浏览器的所有请求转发给fiddler后，fiddler通过监听127.0.0.1:8888端口号获取客户端的请求，然后转发给fiddler工具，再由fiddler工具转发给服务器。(浏览器-->fiddler转发-->服务器)

## 功能
- http://www.51testing.com/zhuanti/fiddler/Fiddler.htm
### 工具介绍
- Repaly：重复发送某一个请求，选择列表其中某一个点击则重复发送此请求一次。
- Go：需要配置状态栏中一起使用，意思是对请求或者响应做断点调试。点击状态栏"All Processes"旁边的空白方框会出现一个箭头。箭头朝上表示在发送阶段会有一个断点，复点则是表示请求回来时会下断点。下完断点后当发送请求时会发现没有返回状态码，然后点击新的请求记录按“Go”则会继续执行下去。
- Stream：流模式和缓冲模式切换。默认是缓冲模式（一个http请求全部响应结束后才会展示请求），按下去则是流模式（实时响应默认，最接近浏览器模式）。
- Decode：将请求的内容解压。
- Any Process：过滤指定进程的请求。使用方法为按住不放，然后拖动到你需要监听的程序界面即可。
- TextWizard：编码/解码文本。
- AutoResponder:将请求模拟成本地。可以选择发送到本地文件。
- Composer:接口测试，模拟发送请求到服务端。

### 常用功能
- 使用fiddler做host配置：Tools-->HOSTS-->Enable-->输入host即可。
- AutoResponder：一般用于线上Bugfix，将线上文件替换为本地文件，然后客户端再次发送请求时实际上会访问本地替换的文件。使用方法：AutoResponder，将请求拖到Rule Editor（这里可以使用正则方式匹配），然后下面选择一个本地文件。那么再次发送次匹配的请求时，会访问本地文件。
- 请求模拟：使用Composer即可模拟Get、Post。。。
- 网速限制：使用FiddlerScript，通过代码来控制。OnBeforeRequest()发送请求之前控制。

### 使用App代理
1. Tools--Options--勾选allow remote computers to connect 和 HTTPS页签中的Capture HTTPS(不勾选的话无法下载证书)
2. 手机端修改Wifi--代理--手动输入服务器主机IP为电脑IP--端口为8888.
3. 手机浏览器访问：电脑IP+端口--会提示下载证书--点击自动下载证书
3. 安装证书后即可监听手机所有请求。