## token
- token主要用于接口之间调用的校验，由服务端提供。

## REST
- rest是HTTP协议的另一个编写格式风格的体现。其实就是HTTP协议。
- rest风格URL：http://localhost:8881/DemoController/getUserByIdForRest/{id}，不同之处在于调用之直接将参数带在URL上。rest风格URL：http://localhost:8881/DemoController/getUserByIdForRest/0


## 使用Apache的org.apache.http包发送请求
- 无论是使用HttpGet，还是使用HttpPost，都必须通过如下3步来访问HTTP资源。
1. 创建HttpGet或HttpPost对象，将要请求的URL通过构造方法传入HttpGet或HttpPost对象。
2. 如果需要发送请求参数，可调用HttpGet、HttpPost共同的setParams(HetpParams params)方法来添加请求参数；对于HttpPost对象而言，也可调用setEntity(HttpEntity entity)方法来设置请求参数。
2. 使用 CloseableHttpClient 类的execute()方法发送HTTP GET或HTTP POST请求，并返回 HttpResponse 对象。
3. 通过 HttpResponse 接口的 getAllHeaders() 方法返回响应头信息。
3. 通过 HttpResponse 接口的 getEntity() 方法返回响应体信息。
- 如果使用HttpPost方法提交HTTP POST请求，则需要使用 HttpPost 类的setHeader()方法设置请求头参数，使用 setEntity() 方法设置请求体参数。

