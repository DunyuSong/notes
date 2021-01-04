# JSON数据读写
### $ref引用错误
- 使用循环构造json数据是，会发生$ref"解析错误。
- 解决方法：禁止循环引用检测的。JSON.toJSONString(resp, SerializerFeature.DisableCircularReferenceDetect);