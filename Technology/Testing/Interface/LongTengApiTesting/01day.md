# 接口学习第一天
### 启动服务
- cd ..\接口学习视频\day01\longtengserver\bin
- ./startup.bat
- 浏览器访问：http://localhost:8881/DemoController/getUserById?id=1

### 状态码200
- 状态码200代表与服务器链接建立成功了，但是**不代表接口中的业务处理逻辑也是成功的**，所以我们断言是不能仅仅只是校验接口的状态码。
- 例如下面这个Post请求虽然服务器返回状态码200 OK，但是实际业务逻辑是失败的。
```html
{"code":400,"msg":"创建失败:com.alibaba.fastjson.JSONException: syntax error, expect {, actual int, pos 0","success":false}
```
### 验证点
1. Code状态码。
2. 返回结果的正确性.
3. DB，校验数据库。例如：createUser接口,返回200 {“success”:true}用户创建成功了。那么能保证正确插入到DB中了么？？？？开发的代码就没有BUG写不进去？
4. 缓存。例如Redis。

### Session
Session是服务器的，Cookie是客户端的。Session就是一次会画，客户端向服务发送请求，服务端生产sessionID后返回给客户端，这样服务端就知道此次会话的是哪一个客户端发送过来的了。