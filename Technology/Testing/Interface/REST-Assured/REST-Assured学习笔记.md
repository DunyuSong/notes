# REST-Assured notes
## API 使用
### RestAssured
- 用户配置全局信息。比如URL
- RestAssured.baseURL= "localhost:8080";
### given()
- 作用：初始化、认证、Auth。Initialization / Auth
- 比如：处理请求的 headers、cookies、parameters、body等。查看具体实现类 RequestSpecificationImpl 中包含的方法。 
- given().param(Map).header():处理URL上的(GET)请求参数，如果是POST，需要使用queryParam(Map);
- 
### when()
- 作用：处理资源 Resources。
- 比如：发送具体的请求。：GET(resources path)、POST(resources path)、PUT(resources path)等。
- when().get("/test/getXmlData").asString();    //  以字符串形式返回响应体内容。
- 

### then()
- 作用：验证 Validation、提取响应体和头 Extract
- 比如：获取响应信息,并进行断言。then()返回的对象是“Validatable”。\
```java
        then().
        assertThat().statusCode(200).and(). 
        contentType(ContentType.JSON).and().
        body("title", equalTo("My Title")).
        header("", "").
```
- then().body()，这里可以抽象出断言，body()里面主要接受两个参数，第一个一般为字符串，第二个为具体断言的函数。
    1. body("id", equalTo(5))
    2. body("id", hasItems(1,2,3,4)，这里的括号可以有多个值，但是也可以当做一个参数处理
    3. body("id", is(12.12f))
    4. body("store.book.findAll { it.price < 10 }.title", hasItems("Sayings", "Moby Dick"));
    5. body("store.book.author.collect { it.length() }.sum()", greaterThan(50));

### 小结
- 一般常规用法
```java
given()     --> 设置请求头、请求体、请求参数、路径参数
when()      --> 发送具体的请求
then()      --> 验证响应信息，做断言
范例：
RequestSpecification request = RestAssured.given();
request.params(params);
request.headers(headers);
Response response = request.when().request(Method.GET, url);
response.then().statusCode(200);
```

## 例子
### GET，使用param()处理 URL 上的请求参数
```java
public class TestRestAssuredGETController {
    @BeforeMethod
    public void beforeMethod() {
        RestAssured.baseURI = "http://localhost:8080";
    }

    @Test(description = "测试发送请求路径上携带参数")
    public void testRestAssuredGet() {
        // 构造请求参数
        Map<String, Object> mapParams = new HashMap<>();
        mapParams.put("PAAA", "P111");
        mapParams.put("PBBB", "P222");
        // 构造请求头
        Map<String, Object> mapHeaders = new HashMap<>();
        mapHeaders.put("Content-Type", "application/json");
        mapHeaders.put("Header1", "H111");
        // 构造请求cookies
        Map<String, Object> mapCookies = new HashMap<>();
        mapCookies.put("cookieNameA", "cookiesValue1");
        mapCookies.put("cookieNameB", "cookiesValue2");

        RequestSpecification request = RestAssured.given();

        request.headers(mapHeaders);
        request.params(mapParams);  // GET URL上 携带的參數使用param來处理

        Response response = request.when().request(Method.GET, "/test/testRestAssuredGet");

        // 验证
        response.then()
                .statusCode(200)    //验证响应码
                .and()              // 添加第二个验证
                .body("lotto.winners[0].winnerId", equalTo(23))     // 验证响应体中的某个值
                .contentType(ContentType.JSON);     // 验证响应体;
        System.out.println("result:\t" + response.prettyPrint());
    }
}
```
### POST，使用 queryParams() 处理 URL 上的参数，使用 body() 处理 application/json 的数据格式
- 请求body为raw类型，格式为application/json
```java
   @BeforeMethod
    public void beforeMethod() {
        RestAssured.baseURI = "http://localhost:8080";
    }

    /**
     * 格式一：请求URL上携带参数、请求体为application/json格式
     * URL：http://localhost:8080/test/testRestAssuredPostWithJSON?PAAA=111&PBBB=222&PCCC=333
     * body：raw --> application/json
     */
    @Test(description = "测试发送请求路径上携带参数，请求体为json格式")
    public void testRestAssuredPOSTWithJSON() {
        // 构造请求参数
        Map<String, Object> mapParams = new HashMap<>();
        mapParams.put("PAAA", "P111");
        mapParams.put("PBBB", "P222");
        String bodyContent = "{\"json\": \"any json string\"}";     // 这里放任意的合法JSON数据即可

        RequestSpecification request = RestAssured.given();

        request.queryParams(mapParams);  // POST 请求 URL 上携带的数据使用 queryParams 处理
        request.body(bodyContent);  // POST 请求 body 中携带的数据，使用body() 处理

        Response response = request.when().request(Method.POST, "/test/testRestAssuredPostWithJSON");

        // 验证
        response.then()
                .statusCode(200);    //验证响应码
        System.out.println("result:\t" + response.prettyPrint());
    }
```
### POST, 请求体为 Object 对象
- 直接使用Java中的对象作为参数,请求中的content-type被设置为"application/json"，rest-assured 也就会把对象序列化为JSON。
- 原理：rest-assured 首先会在您的 classpath中 寻找 Jackson，如果没有则使用Gson。如果您把请求中的content-type修改为"application/xml"，rest-assured将会使用JAXB把对象序列化为XML。
- 如果没有指定content-type，rest-assured会按照以下的优先级进行序列化：
    1. 使用Jackson 2将对象序列化为JSON（Faster Jackson (databind)）
    2. 使用Jackson将对象序列化为JSON（databind）
    3. 使用Gson将对象序列化为JSON
    4. 使用JAXB将对象序列化为XML
- 例子:使用自定义对象class Message {private String message;}
```java
    @Test(description = "请求体为 Object 对象")
    public void testRestAssuredPOSTWithObject() {
        Message bodyContent = new Message();
        bodyContent.setMessage("Hello World");

        RequestSpecification request = RestAssured.given();
//        request.contentType("application/json");    // 必须指定为application/json,因为是基于Content-Type的序列化，如果没有则按照优先级序列化

        request.body(bodyContent);
        Response response = request.when().request(Method.POST, "/test/testRestAssuredPostWithJSON");

        response.then().statusCode(200);    //验证响应码
        System.out.println("result:\t" + response.prettyPrint());
    }
```
### POST, 请求体为 Map 对象
```java
    @Test(description = "请求体为 Map 对象")
    public void testRestAssuredPOSTWithMap() {
        Map<String, Object> bodyContent = new HashMap<>();
        bodyContent.put("Hello World", "Welcome");

        RequestSpecification request = RestAssured.given();

        request.body(bodyContent);
        Response response = request.when().request(Method.POST, "/test/testRestAssuredPostWithJSON");

        response.then().statusCode(200);    //验证响应码
        System.out.println("result:\t" + response.prettyPrint());
    }
```
### POST, Query Parameters 与 Path Parameters 使用？
- URL上的参数处理分为两种情况，一种是传统的key=value拼接形式，另一种是RESTFul格式。
- Query Parameters:请求URL中以?开始参数形式存在，比如 http://localhost:8080/test?key1=value1&key2=valu2
- Path Parameters：请求 URL 中的参数以 {arg} 形式存在，比如 http://localhost:8080/test/{key1}/{key2}
- 这两种形式对应REST-Assured的处理方式方式不同
```
request.params(paramsMap);      // 处理 GET  请求中的 ?key=value 形式参数
request.queryParams(paramsMap); // 处理 POST 请求中的 ?key=value 形式参数
request.pathParams(paramsMap);  // 处理 GET  请求中的 /{key} 形式参数。（应该也适用于POST请求，因为这个主要是处理URL参数）
request.formParams(paramsMap);  // 待研究？？ 
request.body(bodyContent);      // 处理 POST 请求中的Raw格式数据
```

### 断言关键字
- 断言的使用，then()。一般常用为请求头、请求体的验证，大部分是map的格式。比如body("JsonPath表达式", 断言关键字 )， 验证内容是否符合条件(断言关键字)

断言关键字 | 断言说明 | 说明
---|---
contentType(JSON)    |   验证内容类型是JSON格式     |
body("title", equalTo("My Title"))  |   验证标题是My Title  |   equalTo为全精准匹配，区分大小写




- 多个预期结果想要检验
```grovy
given().
        param("x", "y").
when().
        get("/lotto").
then().
        statusCode(200).
        body("lotto.lottoId", equalTo(6));
```
- 多个预期结果想要检验,第二种协防。使用语法糖，"and"在一行代码中使用可以增强可读性。
```
given().param("x", "y").and().header("z", "w").when().get("/something").then().assertThat().statusCode(200).and().body("x.y", equalTo("z"));
```
以上代码等价于下面代码：
```grovy
given().
        param("x", "y").
        header("z", "w").
when().
        get("/something").
then().
        statusCode(200).
        body("x.y", equalTo("z"));
```
## 请求
### 发送请求
```java
/**
 * @Project: interfacetest
 * @Author: ShanDongDong
 * @Time: 2019-05-09 17:28
 * @Email: shandongdong@126.com
 * @Description: 处理请求数据
 * 总结：
 * 1. 请求参数：封装到 Map 中统一处理。然后使用given().params(Map)方式统一处理。
 * 2. 请求头：  封装到 Map 中统一处理。然后使用given().headers(Map)方式统一处理。
 * <p>
 * 最佳实践：
 * 1. 获取request, RequestSpecification request = RestAssured.given();
 * 2. 使用request.xx方式配置请求的各种参数。
 */
public class TestRequest {

    private RequestSpecification request;
    private final String URI = "http://localhost:8080";

    public TestRequest() {
        RestAssured.baseURI = URI;
        RestAssured.config = RestAssured.config().encoderConfig(encoderConfig().appendDefaultContentCharsetToContentTypeIfUndefined(false));
        request = RestAssured.given();
    }

    @Test(description = "发送请求时添加头、cookie等")
    public void testAddHeaders() {
        request.cookie("cookieName", "cookieValue", "cookieValue2");
        request.header("headerName", "value1", "value2");
        request.contentType("application/xml");
        // 请求正文
        request.body("some body");

        Response response = request.when().request(Method.GET, "/test/testRequest");

        System.out.println(response.asString());

    }

    /**
     * 发送POST 请求体为  application/json
     */
    @Test(description = "测试发送JOSN数据")
    public void testRequestPostJSON() {

        JSONObject requestParams = new JSONObject();
        requestParams.put("FirstName", "Virender");
        requestParams.put("LastName", "Singh");
        requestParams.put("UserName", "simpleuser001");
        requestParams.put("Password", "password1");
        requestParams.put("Email", "someuser@gmail.com");

        // Add a header stating the Request body is a JSON
        request.header("Content-Type", "application/json");

        // Add the Json to the body of the request
        request.body(requestParams.toJSONString());
        // Post the request and check the response
        request.log().all();    //打印请求日志
        Response response = request.post("/test/testRequest");

        System.out.println("Response:" + response.asString());

        System.out.println("请在Tomcat下查看Request的打印信息.");
    }

    /**
     * 发送POST请求是有两种方式：
     * 第一种：直接URL上带参数。POST http://localhost:8080/test/testRequest?name=HelloWorld
     * 第二种：通过表单提交
     */
    @Test(description = "测试发送请求时的各种情况")
    public void testRequestPost() {
        // 构造请求头
        Map<String, Object> mapHeaders = new HashMap<>();   //发送请求是携带 请求头   给服务器
//        mapHeaders.put("Content-Type", "application/x-www-form-urlencoded");

        // 构造请求参数
        Map<String, Object> mapParams = new HashMap<>();
        mapParams.put("name", "HelloWorld");                //发送请求是携带 请求参数 给服务器，可以看TestRESTAssured.testExtractResponseValue中的打印信息
        mapParams.put("age", "22");

        // 发送请求，返回Response对象
        Response response =
                given().
                        headers(mapHeaders).        // 设置请求头
                        queryParam("queryPa", mapParams).       //第一种URL携带参数
                        formParam("form1", mapParams).          //第二种表单提交
                        log().all().
                        when().request(Method.POST, "/test/testRequest").
                        andReturn();
        System.out.println("Response:" + response.asString());

        System.out.println("请在Tomcat下查看Request的打印信息.");
    }

    /**
     * 指定请求数据的各种情况：
     * 1. 请求方法。GET、POST、PUT、DELETE等。when().request("GET", "/test/testRequest").
     * 2.
     */
    @Test(description = "测试发送请求时的各种情况")
    public void testRequestGet() {
        // 构造请求参数
        Map<String, Object> mapParams = new HashMap<>();
        mapParams.put("name", "HelloWorld");                //发送请求是携带 请求参数 给服务器，可以看TestRESTAssured.testExtractResponseValue中的打印信息
        mapParams.put("age", "22");                //发送请求是携带 请求参数 给服务器，可以看TestRESTAssured.testExtractResponseValue中的打印信息
        mapParams.put("address", "HZ");                //发送请求是携带 请求参数 给服务器，可以看TestRESTAssured.testExtractResponseValue中的打印信息
        // 构造请求头
        Map<String, Object> mapHeaders = new HashMap<>();   //发送请求是携带 请求头   给服务器
        mapHeaders.put("Content-Type", "application/json");
        mapHeaders.put("Header1", "Welcome");
        // 构造请求cookies
        Map<String, Object> mapCookies = new HashMap<>();
        mapCookies.put("cookieNameA", "cookiesValue1");
        mapCookies.put("cookieNameB", "cookiesValue2");

        // 发送请求，返回Response对象
        Response response =
                given().
                        headers(mapHeaders).        // 设置请求头
                        params(mapParams).          // 设置请求参数
                        cookies(mapCookies).        // 设置请求 Cookies
//                        when().get("/test/testResponse").                       // 第一种方式：发送请求, 返回Response对象
//                        when().request(Method.GET, "/test/testResponse").       // 第二种方式：发送请求，返回Response对象
        when().request("GET", "/test/testRequest").              // 第三种方式：发送请求， 返回Response对象
                        andReturn();
        response.then().statusCode(200);
        System.out.println("Response:" + response.asString());

        System.out.println("请在Tomcat下查看Request的打印信息.");
    }
}

```
## 响应
### 获取响应体的几种方式
```java
InputStream stream = get("/lotto").asInputStream(); // Don't forget to close this one when you're done
byte[] byteArray = get("/lotto").asByteArray();
String json = get("/lotto").asString();
```
### 从响应体中提取值，作为下一个请求的请求内容
```java

/**
 * @Project: interfacetest
 * @Author: ShanDongDong
 * @Time: 2019-05-09 10:27
 * @Email: shandongdong@126.com
 * @Description: 测试Response相关
 * 总结：
 * 1.响应信息：封装到 Response 对象中统一存储处理，为后续提取数据做标准化，提取响应分为两种：
 * 第一种：提取响应头，response.header(headerName）
 * 第二种：提取响应体，response.path(JsonPath)
 * 2 断言一般分为两种：
 * 一个参数：contentType(ContentType.JSON).这种单值的可以根据断言类型来区分，如果是响应头的断言则使用这种。
 * 两个参数：body("title", equalTo("My Title"))，第一个形参为JOSNPath，第二个形参为函数。
 */
public class TestResponse {

    @Test(description = "测试获取响应体中的各种情况。提取多个值，作为下一个接口的入参")
    public void testResponse() {
        // 构造请求头
        Map<String, Object> mapHeaders = new HashMap<>();   //发送请求是携带 请求头   给服务器
        mapHeaders.put("Content-Type", "application/json");

        // 发送请求，返回Response对象
        Response response =
                given().
                        headers(mapHeaders).        // 设置请求头
                        when().request("GET", "/test/testResponse").              // 发送请求， 返回Response对象
                        then().
                        contentType(ContentType.JSON).
                        body("title", equalTo("My Title")).
                        extract().
                        response();  //返回Response的整个返回值

        // 从Response中使用JsonPath 提取多个值
        String nextTitleLink = response.path("_links.next.href");
        String headerValue = response.header("Content-type");

        System.out.println("path:" + nextTitleLink);
        System.out.println("header:" + headerValue);

        // 从Response 中获取请求头
        Headers headers = response.getHeaders();
        for (Header header : headers) {
            System.out.println("headerName:" + header.getName() + ", headerValue:" + header.getValue());
        }

        // 获取所有 cookie 键值对
        Map<String, String> cookies = response.getCookies();
        for (String str : cookies.keySet()) {
            System.out.println("cookieName:" + str + ", cookieValue:" + response.getCookie(str));


        }
        Cookies cookies1 = response.getDetailedCookies();   //获取所有详细的响应cookies
        System.out.println("DetailedCookies:" + cookies1);

        // 获取状态行信息
        String statusLine = response.getStatusLine();       // 结果:HTTP/1.1 200
        System.out.println("statusLine:" + statusLine);     // 200

        // 获取状态码信息
        int statusCode = response.getStatusCode();
        System.out.println("statusCode:" + statusCode);

    }

    @Test(description = "使用JsonPath提取响应值")
    public void testExtractResponseValueUseJsonPath() {
        // 获取响应体的三种方式
        InputStream stream = get("/test/lotto").asInputStream(); // Don't forget to close this one when you're done
        byte[] byteArray = get("/test/lotto").asByteArray();
        String json = get("/test/lotto").asString();

        //从响应信息中提取值.独立地使用了JsonPath，而没有依赖rest-assured
        JsonPath.config = new JsonPathConfig("UTF-8");  //静态配置好JsonPath，这样所有的JsonPath实例都会共享这个配置：

        int lottoId = from(json).getInt("lotto.lottoId");
        List<Integer> winnerIds = from(json).get("lotto.winners.winnerId");

        System.out.println("lottoId:" + lottoId);
        System.out.println("winnerIds:" + winnerIds);


    }

}
```

## 日志
### 打印请求与响应的日志
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
        response.then().log().ifStatusCodeIsEqualTo(302);   //状态为指定条件时才打印日志
        System.out.println("响应的状态码：" + response.statusCode());
    }
```
