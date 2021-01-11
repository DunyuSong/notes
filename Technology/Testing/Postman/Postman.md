# Postman

## 如何使用Postman 的Runner进行参数化接口

>   参考：https://learning.postman.com/docs/running-collections/working-with-data-files/

### 方式一：使用JSON数据

1. 将接口保存到Collection中。

2. 点击Runner，新建一个运行器，将接口拖到Run order 列表中。

3. 右边的Data选择一个json文件，格式如下：

    ```json
    [{
      "path": "post1",
      "value": "dd.shan8"
    }, {
      "path": "post2",
      "value": "dd.shan7"
    }, {
      "path": "post3",
      "value": "3"
    }, {
      "path": "post4",
      "value": "dd.shan5"
    }]
    ```

4. 在接口中使用文件中的数据，直接在接口中将需要替换的值使用{{path}}或{{value}}替换即可引用到文件中的值。

5. 如果需要在接口脚本Pre-request Script或Tests中使用，则使用postaman内置函数console.log(pm.iterationData.get("value"))即可。
### 方式二：使用csv文件
1. Runner中Data选择一个csv文件，格式如下：
```
username,password
dd.shan8,Autotest#123
dd.shan7,Autotest#123
dd.shan6,Autotest#123
dd.shan5,Autotest#123
```
2. 在接口中使用文件中的数据，直接在接口中将需要替换的值使用{{username}}或{{password}}替换即可引用到文件中的值。
