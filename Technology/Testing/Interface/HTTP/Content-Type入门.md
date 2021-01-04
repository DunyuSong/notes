# Content-Type
## Servlet 中如何根据客户端发送过来的Content-Type进行不同的处理
- ContentType.java
```java
/**
 * @Project: interface-test
 * @Author: ShanDongDong
 * @Time: 2019-07-09 14:38
 * @Email: shandongdong@126.com
 * @Description: Servlet中处理客户端发送过来的Content-Type
 */

import lombok.extern.slf4j.Slf4j;

import javax.servlet.ServletOutputStream;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.BufferedInputStream;
import java.io.ByteArrayInputStream;
import java.io.IOException;

@Slf4j
@WebServlet(description = "模拟处理Content-Type", urlPatterns = {"/ContentType"})
public class ContentType extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws  IOException {
        String strXMLResponse = "<Response><code>";
        String strMessage = "";
        int intCode = 0;
        ServletOutputStream stream = null;
        BufferedInputStream buffer = null;

        try {
            String strContentType = request.getContentType();

            // 处理 text/xml
            if (strContentType.equalsIgnoreCase("text/xml")) {
                strMessage = "text/xml";
            }
            // 处理Content-Type为application/json
            else if (strContentType.equalsIgnoreCase("application/json")) {
                strMessage = "Application/json";
            }
            // 其他格式
            else {
                strMessage = "other MIME type";
                intCode = 1;
            }
            strXMLResponse += intCode + "</code><message>" + strMessage + "</message></Response>";

            // 设置响应数据
            response.setContentType("text/xml");
            response.setContentLength(strXMLResponse.length());

            int intReadBytes = 0;

            stream = response.getOutputStream();

            ByteArrayInputStream bs = new ByteArrayInputStream(strXMLResponse.getBytes());
            buffer = new BufferedInputStream(bs);

            while ((intReadBytes = buffer.read()) != -1) {
                stream.write(intReadBytes);
            }
        } catch (IOException e) {
            log.error(e.getMessage());
        } catch (Exception e) {
            log.error(e.getMessage());
        } finally {
            stream.close();
            buffer.close();
        }
    }
}
```
- 测试Content-Type，可以使用代码或者postman来试验
- 代码方式：TestContentType.java
```java
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

/**
 * @Project: interface-test
 * @Author: ShanDongDong
 * @Time: 2019-07-09 14:40
 * @Email: shandongdong@126.com
 * @Description:
 */
public class TestContentType {
    /**
     * @param args
     */
    public static void main(String[] args) {
        BufferedReader inStream = null;

        try {
            // Connect to servlet
            URL url = new URL("http://localhost:8080/ContentType");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();

            // Initialize the connection
            conn.setDoOutput(true);
            conn.setDoInput(true);
            conn.setRequestMethod("POST");
            conn.setUseCaches(false);
            conn.setRequestProperty("Content-Type", "application/json");
//            conn.setRequestProperty("Content-Type", "text/xml");
//            conn.setRequestProperty("Content-Type", "text/html");
            //conn.setRequestProperty("Connection", "Keep-Alive");

            conn.connect();

            OutputStream out = conn.getOutputStream();

            inStream = new BufferedReader(new InputStreamReader(conn.getInputStream()));

            String strXMLRequest = "<?xml version=\"1.0\" encoding=\"UTF-8\"?><Request></Request>";
            out.write(strXMLRequest.getBytes());
            out.flush();
            out.close();

            String strServerResponse = "";

            System.out.println("Server says: ");
            while ((strServerResponse = inStream.readLine()) != null) {
                System.out.println(strServerResponse);
            } // end while

            inStream.close();
        } catch (IOException e) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        } 
    }
}

```
## SpringMVC 中如何根据客户端发送过来的Content-Type进行不同的处理
```java
@Controller
@RequestMapping(value = "/test")
public class TestController {

    /**
     * 测试SpringMVC处理不同的Content-Type.
     * consumes： 指定处理请求的提交内容类型（Content-Type），例如application/json, text/html;
     * produces:  指定返回的内容类型，仅当request请求头中的(Accept)类型中包含该指定类型才返回.
     * 在 HTTP 协议中
     * ContentType: 用来告诉服务器当前发送的数据是什么格式
     * Accept:      用来告诉服务器，客户端能认识哪些格式,最好返回这些格式中的其中一种
     *
     *
     * consumes 用来限制ContentType
     * produces 用来限制Accept
     */
    @RequestMapping(value = "/testContentType", consumes = MediaType.APPLICATION_JSON_VALUE)
    public String testContentType() {
        return "success";
    }
}
```
### RequestMapping注解中consumes与produces的区别
在 Request 中 
ContentType 用来告诉服务器当前发送的数据是什么格式 
Accept      用来告诉服务器，客户端能认识哪些格式,最好返回这些格式中的其中一种 

consumes 用来限制ContentType 
produces 用来限制Accept  

举例: 
有个用户给我发了一个请求, 

请求头中 
     ContentType =application/json 
     Accept      =  */* 
就是说用户发送的json格式的数据，可以接收任意格式的数据返回 

但是我的接口中定义了consumes={"application/xml"},produces={"application/xml"} 
我只接收 application/xml 格式，也只返回xml格式 

很明显，用户调不通这个接口 

所以我改下consumes={"application/xml","application/json"},produces={"application/xml"} 
注: 除了格式支持，还需要与数据对应的http转换器（HttpMessageConverter）此处先跳过     


MediaType 其实就是 application/xml,application/json 等类型格式 

接下来看下源码 
- AbstractHandlerMethodMapping.java
```java
protected HandlerMethod lookupHandlerMethod(String lookupPath, HttpServletRequest request) throws Exception {  
        List<Match> matches = new ArrayList<Match>();  
                //这里找到所有对应path的方法  
        List<T> directPathMatches = this.mappingRegistry.getMappingsByUrl(lookupPath);  
        if (directPathMatches != null) {  
            addMatchingMappings(directPathMatches, matches, request);  
        }  
        if (matches.isEmpty()) {  
            // No choice but to go through all mappings...  
            addMatchingMappings(this.mappingRegistry.getMappings().keySet(), matches, request);  
        }  
  
        if (!matches.isEmpty()) {  
            Comparator<Match> comparator = new MatchComparator(getMappingComparator(request));  
            Collections.sort(matches, comparator);  
            if (logger.isTraceEnabled()) {  
                logger.trace("Found " + matches.size() + " matching mapping(s) for [" +  
                        lookupPath + "] : " + matches);  
            }  
            Match bestMatch = matches.get(0);  
            if (matches.size() > 1) {  
                if (CorsUtils.isPreFlightRequest(request)) {  
                    return PREFLIGHT_AMBIGUOUS_MATCH;  
                }  
                Match secondBestMatch = matches.get(1);  
                if (comparator.compare(bestMatch, secondBestMatch) == 0) {  
                    Method m1 = bestMatch.handlerMethod.getMethod();  
                    Method m2 = secondBestMatch.handlerMethod.getMethod();  
                    throw new IllegalStateException("Ambiguous handler methods mapped for HTTP path '" +  
                            request.getRequestURL() + "': {" + m1 + ", " + m2 + "}");  
                }  
            }  
            handleMatch(bestMatch.mapping, lookupPath, request);  
            return bestMatch.handlerMethod;  
        }  
        else {  
            return handleNoMatch(this.mappingRegistry.getMappings().keySet(), lookupPath, request);  
        }  
    }  
  
  
    private void addMatchingMappings(Collection<T> mappings, List<Match> matches, HttpServletRequest request) {  
        for (T mapping : mappings) {  
            T match = getMatchingMapping(mapping, request);  
            if (match != null) {  
                matches.add(new Match(match, this.mappingRegistry.getMappings().get(mapping)));  
            }  
        }  
    }  
```
- RequestMappingInfo.java 对header,consumes和produces进行筛选 
```java
public RequestMappingInfo getMatchingCondition(HttpServletRequest request) {  
        RequestMethodsRequestCondition methods = this.methodsCondition.getMatchingCondition(request);  
        ParamsRequestCondition params = this.paramsCondition.getMatchingCondition(request);  
        HeadersRequestCondition headers = this.headersCondition.getMatchingCondition(request);  
[color=red]     ConsumesRequestCondition consumes = this.consumesCondition.getMatchingCondition(request);  
        ProducesRequestCondition produces = this.producesCondition.getMatchingCondition(request);[/color]  
  
        if (methods == null || params == null || headers == null || consumes == null || produces == null) {  
            return null;  
        }  
  
        PatternsRequestCondition patterns = this.patternsCondition.getMatchingCondition(request);  
        if (patterns == null) {  
            return null;  
        }  
  
        RequestConditionHolder custom = this.customConditionHolder.getMatchingCondition(request);  
        if (custom == null) {  
            return null;  
        }  
  
        return new RequestMappingInfo(this.name, patterns,  
                methods, params, headers, consumes, produces, custom.getCondition());  
    }  
```
- ConsumesRequestCondition.java 对consume的过滤 
```java
public ConsumesRequestCondition getMatchingCondition(HttpServletRequest request) {  
    if (CorsUtils.isPreFlightRequest(request)) {  
        return PRE_FLIGHT_MATCH;  
    }  
    if (isEmpty()) {  
        return this;  
    }  
    MediaType contentType;  
    try {  
        contentType = StringUtils.hasLength(request.getContentType()) ?  
                MediaType.parseMediaType(request.getContentType()) :  
                MediaType.APPLICATION_OCTET_STREAM;  
    }  
    catch (InvalidMediaTypeException ex) {  
        return null;  
    }  
    Set<ConsumeMediaTypeExpression> result = new LinkedHashSet<ConsumeMediaTypeExpression>(this.expressions);  
    for (Iterator<ConsumeMediaTypeExpression> iterator = result.iterator(); iterator.hasNext();) {  
        ConsumeMediaTypeExpression expression = iterator.next();  
        if (!expression.match(contentType)) {  
            iterator.remove();  
        }  
    }  
    return (result.isEmpty()) ? null : new ConsumesRequestCondition(result);  
}  
```
- ProducesRequestCondition.java 对Produces的过滤
```java
public ProducesRequestCondition getMatchingCondition(HttpServletRequest request) {  
        if (CorsUtils.isPreFlightRequest(request)) {  
            return PRE_FLIGHT_MATCH;  
        }  
        if (isEmpty()) {  
            return this;  
        }  
        List<MediaType> acceptedMediaTypes;  
        try {  
                        //获取Accept  
            acceptedMediaTypes = getAcceptedMediaTypes(request);  
        }  
        catch (HttpMediaTypeException ex) {  
            return null;  
        }  
        Set<ProduceMediaTypeExpression> result = new LinkedHashSet<ProduceMediaTypeExpression>(expressions);  
        for (Iterator<ProduceMediaTypeExpression> iterator = result.iterator(); iterator.hasNext();) {  
            ProduceMediaTypeExpression expression = iterator.next();  
            if (!expression.match(acceptedMediaTypes)) {  
                iterator.remove();  
            }  
        }  
        if (!result.isEmpty()) {  
            return new ProducesRequestCondition(result, this.contentNegotiationManager);  
        }  
        else if (acceptedMediaTypes.contains(MediaType.ALL)) {  
            return EMPTY_CONDITION;  
        }  
        else {  
            return null;  
        }  
    }  
```
# 参考：
> - https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/POST
> - https://segmentfault.com/a/1190000013056786
