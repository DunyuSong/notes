# REST-Assured 问题笔记
发现问题是先添加日志语句打印出所有日志信息.
1. 在Web服务器上打印所有日志，比如使用Servlet API request.getHeaderNames();遍历所有请求头。
2. request.log().all();             //打印请求日志
3. response.then().log().all();     //打印响应日志;     
```java
    @Test(description = "测试请求体为对象，自动序列化为JSON")
    public void testBodyIsObjectToJSON() {
        RestAssured.config = RestAssured.config().encoderConfig(encoderConfig().appendDefaultContentCharsetToContentTypeIfUndefined(false));
        RequestSpecification request = given();
        Message message = new Message();
        message.setMessage("Hello World");
        request.contentType("application/json");    //必须指定为application/json,因为是基于Content-Type的序列化
        request.body(message);
        System.out.println("--------------------打印请求日志------------------------------");
        request.log().all();    //打印请求日志
        Response response = request.when().request("GET", "/test/testRequest");
        System.out.println("-------------------------打印响应日志-------------------------");
        response.then().log().all();     //打印响应日志
        System.out.println("--------------------------------------------------");
        System.out.println("响应的状态码：" + response.statusCode());
    }
```
### POST JSON(applicatin/json)数据时服务器没有收到
问题现象： 使用PostMan发送请求是服务器能收到发送的数据，但是使用REST-Assured代码服务器方式收不到。
问题原因： 问题出在Content-Type = application/json; charset=UTF-8，您应该避免自动将charset添加到content-type标头：
解决方法： "RestAssured.config = RestAssured.config().encoderConfig(encoderConfig().appendDefaultContentCharsetToContentTypeIfUndefined(false));"


### 不同Content-type调用不同的方法：application/json、application/x-www-form-urlencoded
request.body(jsonAsMap)。接受的请求方式 application/json 时放 body 里即可。
request.formParam("param1", jsonAsMap)。接受的请求方式 application/x-www-form-urlencoded 时放 formParam 里即可。

### You can either send form parameters OR body content in POST, not both!
问题现象：发送 POST 请求是 URL 上携带了参数，请求body为 application/json。
问题原因：
解决方法：使用“ request.queryParams()”,而不是"request.params()""
```json
        // Add a header stating the Request body is a JSON
        request.header("Content-Type", "application/json");

        // 添加请求参数URL上的值key=value&key=value
        request.queryParams(requestParams);
//        request.params(requestParams);  //不能使用这种方式

        // 在请求体上添加JSON数据格式
        request.body(requestBody.toJSONString());
```

### queryParams()、params()、body()区别
- queryParams():  处理POST请求URL上携带的参数。比如/testRequest?name=tom&age=14
- body(): 处理请求体携带的参数。比如类型为“application/json、application/x-www-form-urlencoded”
- params: 处理GET请求URL上携带的参数。比如/testRequest?name=tom&age=14
- 注意的是：queryParams可以和 body()或params同时使用，但是body()和params()不能同时存在,因为GET请求没有body。
- 这两个方法方法一个是用于处理请求URI上的额参数，一个是处理请求体的。但是请求体内又分为很多种，比如multipart/form-data、 application/x-www-form-urlencoded 不同处理方式也不同。form-data可以包含文本和文件类型，x-www-form-urlencoded则只有文本类型。
