# HTTPClient工具的封装
- 官网：https://github.com/Arronlong/httpclientUtil 
- https://blog.csdn.net/xiaoxian8023/article/details/49883113

## 发送Get请求
```java
Header[] headers = HttpHeader.custom().contentType("application/json").build();
HttpConfig config = HttpConfig.custom().headers(headers).url(url);                    //设置请求的url
HttpClientUtil.get(config);
```
## 发送Post请求，参数为Json
- 需要使用json()方法发送Json数据：HttpConfig.custom().url(url).json(json); 
```java
    public void testContentCheck() throws HttpProcessException {
        String url = BASE_URL + "/v1/article/check";

        String json = "{\n" +
                "    \"infoId\": \"II0WZTY9YT6O3ZQ\",\n" +
                "    \"infoType\": \"article\",\n" +
                "    \"title\": \"重拳反腐无禁区、全面监督无死角 世界感知中国打击腐败决心\",\n" +
                "    \"content\": \"综述：重拳反腐无禁区 全面监督无死角——世界感知中国打击腐败之决心\",\n" +
                "    \"picList\": [{\n" +
                "        \"url\": \"http://recc.nosdn.127.net/irec-20180711-d4a7e8986acc0244a738999103f38ddc.jpg\"\n" +
                "    }, {\n" +
                "        \"url\": \"http://recc.nosdn.127.net/irec-20180529-54fb2021979d9224f32dedfd9e140248.jpg\"\n" +
                "    }]\n" +
                "}";

        //步骤2：配置请求参数（网址、请求参数、编码、client）
        Header[] headers = HttpHeader.custom().contentType("application/json;charset=UTF-8").build();
        HttpConfig config = HttpConfig.custom().headers(headers).url(url).json(json);   //发送json请求数据

        String result = HttpClientUtil.post(config);
        int status = HttpClientUtil.status(config);
        System.out.println(status);
        System.out.println(result);
    }
```