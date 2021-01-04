# Asynchronous JavaScript and XML
- 异步的JavaScript与xml。
- Ajax中一个重要的对象 XMLHttpRequest。
## 发送请求的步骤
1. 准备工作：使用xmlHttpRequest.open("GET", "ServletAjax", true);方法准备向服务器发送异步请求。
2. 关联回调函数。xmlHttpRequest.onreadystatechange = ajaxCallBack;   //当状态改变时使用ajaxCallBack函数来处理
3. 向服务器发送数据。xmlHttpRequest.send(null);  //如果以POST方法发送请求，那么写具体的参数
4. 在回调函数中处理响应结果。

### 使用Ajax发送GET请求
- 编写服务端代码ServletAjax.java
```java
@WebServlet(name = "ServletAjax", urlPatterns = "/ServletAjax")
public class ServletAjax extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        PrintWriter out = response.getWriter();
        out.println("Hello World Ajax!!!");
        System.out.println(this.getClass().getSimpleName() + " doGet invoked!");
        out.flush();
        out.close();
    }
}
```
- 客户端ajax.jsp页面
```html
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>

    <script type="text/javascript">
        var xmlHttpRequest = null;      //声明一个xmlHttpRequest对象
        function ajaxSubmit() {
            if (window.ActiveXObject)    //如果这个对象存在，那么是IE浏览器
            {
                xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
            } else if (window.XMLHttpRequest)  //IE之外的浏览器
            {
                xmlHttpRequest = new XMLHttpRequest();
            } else {
                //其他小众浏览器，不处理
            }

            //向服务器发送请求
            if (null != xmlHttpRequest) {
                xmlHttpRequest.open("GET", "../../ServletAjax", true);
                //关联回调函数
                xmlHttpRequest.onreadystatechange = ajaxCallBack;   //当状态改变时使用ajaxCallBack函数来处理

                //真正向服务器发送数据
                xmlHttpRequest.send(null);  //如果以POST方法发送请求，那么写具体的参数
            }
        }
        //处理响应
        function ajaxCallBack() {
            //1：构造数据对象。2：发送请求完成。3：响应达到还未处理完。4：响应处理完毕
            if (xmlHttpRequest.readyState == 4) {
                //http响应状态码
                if (xmlHttpRequest.status == 200) {
                    var responseText = xmlHttpRequest.responseText; //获取服务器响应的文本
                    document.getElementById("div1").innerHTML = responseText;   //
                }
            }
        }
    </script>
</head>
<body>
<h1>单击按钮后从服务端拿到数据然后异步展示到客户端</h1>
<input type="button" value="get content from servlet" onclick="ajaxSubmit();">
<div id="div1"></div>
</body>
</html>
```
### 使用Ajax发送带参数的GET请求
- 修改ajax.jsp。完成两个数相加的功能。
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>Title</title>

    <script type="text/javascript">
        var xmlHttpRequest = null;      //声明一个xmlHttpRequest对象
        function ajaxSubmit() {
            if (window.ActiveXObject)    //如果这个对象存在，那么是IE浏览器
            {
                xmlHttpRequest = new ActiveXObject("Microsoft.XMLHTTP");
            } else if (window.XMLHttpRequest)  //IE之外的浏览器
            {
                xmlHttpRequest = new XMLHttpRequest();
            } else {
                //其他小众浏览器，不处理
            }

            //向服务器发送请求
            if (null != xmlHttpRequest) {
                var v1 = document.getElementById("value1ID").value;
                var v2 = document.getElementById("value2ID").value;

                xmlHttpRequest.open("GET", "../../ServletAjax?v1=" + v1 + "&v2=" + v2, true);
                //关联回调函数
                xmlHttpRequest.onreadystatechange = ajaxCallBack;   //当状态改变时使用ajaxCallBack函数来处理

                //真正向服务器发送数据
                xmlHttpRequest.send(null);  //如果以POST方法发送请求，那么写具体的参数
            }
        }

        //处理响应
        function ajaxCallBack() {
            //1：构造数据对象。2：发送请求完成。3：响应达到还未处理完。4：响应处理完毕
            if (xmlHttpRequest.readyState == 4) {
                //http响应状态码
                if (xmlHttpRequest.status == 200) {
                    var responseText = xmlHttpRequest.responseText; //获取服务器响应的文本
                    document.getElementById("div1").innerHTML = responseText;   //
                }
            }
        }
    </script>
</head>
<body>
<h1>单击按钮后从服务端拿到数据然后异步展示到客户端</h1>
<input type="button" value="get content from servlet" onclick="ajaxSubmit();"><br/>
<input type="text" name="value1" id="value1ID"><br/>
<input type="text" name="value2" id="value2ID"><br/>
<div id="div1"></div>
</body>
</html>
```
-修改服务端代码ServletAjax
```java
@WebServlet(name = "ServletAjax", urlPatterns = "/ServletAjax")
public class ServletAjax extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println(this.getClass().getSimpleName() + "doPost invoked!");
        process(request, response);
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        System.out.println(this.getClass().getSimpleName() + "doGet invoked!");
        process(request, response);
    }
    private void process(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String v1 = request.getParameter("v1");
        String v2 = request.getParameter("v2");
        String v3 = String.valueOf(Integer.valueOf(v1) + Integer.valueOf(v2));
        PrintWriter out = response.getWriter();
        //设置不缓存
        response.setHeader("param", "no-cache");
        response.setHeader("cache-control", "no-cache");

        out.println(v3);
        out.flush();
        out.close();
    }
}
```
### 使用Ajax发送带参数的 POST 请求
- 修改客户端ajax.jsp文件
```html
            //向服务器发送请求
            if (null != xmlHttpRequest) {
                var v1 = document.getElementById("value1ID").value;
                var v2 = document.getElementById("value2ID").value;

                xmlHttpRequest.open("POST", "../../ServletAjax", true);
                //关联回调函数
                xmlHttpRequest.onreadystatechange = ajaxCallBack;   //当状态改变时使用ajaxCallBack函数来处理
                //如果是以POST方式发送数据需要在请求头设置参数
                xmlHttpRequest.setRequestHeader("Content-Type", "application/x-www-form-urlencoded")
                //真正向服务器发送数据
                xmlHttpRequest.send("v1=" + v1 + "&v2=" + v2);  //如果以POST方法发送请求，那么写具体的参数
            }
```

