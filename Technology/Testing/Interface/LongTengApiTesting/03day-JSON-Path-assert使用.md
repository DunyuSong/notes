
### 如何测试每次返回结果顺序不同结果？
- 采用循环变量方式

### 如何过滤到指定数据
- 接口部分数据可以实时在变化，例如时间戳。
- 

## JSON xPath的使用
- https://blog.csdn.net/koflance/article/details/63262484
- https://github.com/json-path/JsonPath
- 实时验证jsonpath语法效果：http://jsonpath.com/
```pom
      <!-- JSON XPath 的使用 -->
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path</artifactId>
            <version>2.4.0</version>
        </dependency>
        <dependency>
            <groupId>com.jayway.jsonpath</groupId>
            <artifactId>json-path-assert</artifactId>
            <version>2.4.0</version>
        </dependency>

```
- json数据
```java
    private String getJson() {
        return "{ \"bookstore\": {\n" +
                "    \"book\": [ \n" +
                "      { \"category\": \"reference\",\n" +
                "        \"author\": \"Nigel Rees\",\n" +
                "        \"title\": \"Sayings of the Century\",\n" +
                "        \"price\": 8.95\n" +
                "      },\n" +
                "      { \"category\": \"fiction\",\n" +
                "        \"author\": \"Evelyn Waugh\",\n" +
                "        \"title\": \"Sword of Honour\",\n" +
                "        \"price\": 12.99,\n" +
                "        \"isbn\": \"0-553-21311-3\"\n" +
                "      },{\"author\":\"jiaou\"," +
                "\"price\":18" +
                "}\n" +
                "    ],\n" +
                "    \"bicycle\": {\n" +
                "      \"color\": \"red\",\n" +
                "      \"price\": 19.95\n" +
                "    }\n" +
                "  }\n" +
                "}";
    }
```
### 获取book数组下所有的author的值

```java
 public void getAllAttribute() {
        String json = getJson();
        List<String> authors = JsonPath.read(json, "$.bookstore.book[*].author");
        for (String s : authors) {
            System.out.println(s);
        }
        //获取book对象下的第一个author属性
        //{ "category": "reference",\"author\": \"Nigel Rees\",\n" \"title\": \"Sayings of the Century\",\n" +\"price\": 8.95\n" +"      }
        String author = JsonPath.read(json, "$.bookstore.book[0].author");
        System.out.println(author);
    }
```
1. $.bookstore：代表获取根路径
2. $.bookstore.book[*]：代表获取bookstores对象下的book对象，因为book是数组所有使用[]，这里我们需要获取所有book对象所有使用 * 号。
3. $.bookstore.book[*].author：获取book[\*]下的所有author属性。因为有多个所以返回的应该是List<String>对象。
4. $.bookstore.book[0].author：获取book对象下的第一个属性为author的值。因为是只有一个值所以返回的应该值应该是String对象。

### 获取指定属性的对象
```java
    /*
     * @Description:获获取book属性中category=reference的对象
     * */
    @Test
    public void getObjectByAttribute() {
        String json = getJson();
        List<Object> books = JsonPath.read(json, "$.bookstore.book[?(@.category == 'reference')]");
        json = JSON.toJSONString(books);
        System.out.println(json);
    }
```
1.[? @.category == 'reference']:获取属性为reference的对象。
- 第二种方式：使用过滤器
```java
    /*
     * @Description:获获取book属性中category=reference的对象filter表达式
     * */
    @Test
    public void getObjectOfAttributeByFilter() {
        String json = getJson();
        //filter 过滤器
        List<Object> books = JsonPath.read(json, "$.bookstore.book[?]", filter(where("category").is("reference")));
        json = JSON.toJSONString(books);
        System.out.println(json);
    }

```
### 多条件查询
```java
    /*
     * @Description:链式过滤器，多个条件
     * */
    @Test
    public void chainedFilters() {
        String json = getJson();
        Filter filter = filter(where("isbn").exists(true).and("category").in("fiction", "reference"));
        List<Object> books = JsonPath.read(json, "$.bookstore.book[?]", filter);
        json = JSON.toJSONString(books);
        System.out.println(json);
    }
```
## JSON 断言
- https://github.com/json-path/JsonPath/tree/master/json-path-assert
- 

