# Moco框架
- 下载地址：https://github.com/dreamhead/moco 

## 基本使用
1. 下载jar包：https://github.com/dreamhead/moco
2. 创建startup1.json文件
```json
[
  {
    "description": "第一个Moco例子",
    "request": {
      "uri": "/demo"
    },
    "response": {
      "text": "This is my first Moco example."
    }
  }
]
```
3. 输入命令：java -jar moco-runner-0.12.0-standalone.jar http -p 7001 -c startup1.json
4. 浏览器输入：http://localhost:7001/demo

### 模拟Get请求
1. 编写json文件
```json
[
  {
    "description":"模拟一个有参数的Get请求",
    "request":
    {
      "uri":"/getwithparam",
      "method":"get",
      "queries":
      {
        "name":"hello world",
        "age": "33"
      }
    },
    "response": {
      "text": "成功返回响应结果"
    }
  }
]
```
2. 编写完后服务会自动重启，如果没有手动重新启一下服务。
3. 浏览器输入：http://localhost:7001/getwithparam?name=hello world&age=33

### 模拟Post请求
1. 编写json文件
```json
[
  {
    "description":"模拟一个Post请求",
    "request":{
      "uri":"/postdemo",
      "method":"post",
      "forms":
      {
        "name":"hello",
        "sex":"man"
      }

    },
    "response":
    {
      "text":"This is my moco post request."
    }
  }
]
```
2. 启动服务。
3. 使用postman发送post请求。Body里面填写2个参数。

### 携带cookies
1.编写json文件
```json
[
  {
    "description":"这个一个带cookies信息的Get请求",
    "request":{
      "uri":"/cookies/get",
      "method":"get",
      "cookies":
      {
        "login":"true"
      }
    },
    "response":
    {
      "text":"这是一个需要携带cookies信息才能访问的请求。"
    }
  },
  {
    "description":"这个一个带cookies信息的Post请求",
    "request":{
      "uri":"/cookies/post",
      "method":"post",
      "cookies":
      {
        "login":"true"
      },
      "json":
      {
        "name":"abc",
        "age":"33"
      }
    },
    "response":
    {
      "status":200,
      "json":{
        "bob":"success",
        "status":1
      }
    }
  }
]
```
2. 启动服务。
3. 使用postman发送get请求。headers里面填写key:Cookie,value:login=true.
4. 使用postman发送get请求。headers里面填写key:Cookie,value:login=true。在Body里面选择raw格式值填写:{"name":"abc","age":"33" }，格式下拉选择JSON(application/json)

### 携带Headers
1.编写json文件
```json
[
  {
    "description": "这个一个带header信息的Post请求",
    "request": {
      "uri": "/cookies/headers",
      "method": "post",
      "headers": {
        "content-type": "application/json"
      },
      "json": {
        "name": "abc",
        "age": "33"
      }
    },
    "response": {
      "status": 200,
      "json": {
        "bob": "success",
        "status": 1
      }
    }
  }
]
```
2. 启动服务。
3.使用postman发送Post请求。headers里面填写key:Content-Type,value:application/json。在Body里面选择raw格式值填写:{"name":"abc","age":"33" }，格式下拉选择JSON(application/json)

### 请求重定向
1. 编写json文件
```json
[
  {
    "description":"重定向到百度",
    "request":
    {
      "uri":"/redirect"
    },
    "redirectTo":"http://www.baidu.com"
  },
  {
    "description":"重定向到自己的网站",
    "request":
    {
      "uri":"/redirect/topath"
    },
    "redirectTo":"/redirect/new"
  },
  {
    "description":"这是被重定向的请求",
    "request":
    {
      "uri":"/redirect/new"
    },
    "response":
    {
      "text":"重定向成功...."
    }
  }
]
```
2. 启动服务。
3. 访问：http://localhost:7001/redirect
4. 访问：http://localhost:7001/redirect/topath

## 请求参数中文乱码问题
1. 设置请求头编码格式：content-type: "application/json;charset=UTF-8"
```json
{
    "description": "新建文章反垃圾",
    "request": {
      "uri": "/v1/article/check",
      "method": "post",
      "headers": {
        "content-type": "application/json;charset=UTF-8"
      },
      "json": {
        "infoId": "II0WZTY9YT6O3ZQ",
        "infoType": "article",
        "title": "重拳反腐无禁区、全面监督无死角 世界感知中国打击腐败决心",
        "content": "综述：重拳反腐无禁区 全面监督无死角——世界感知中国打击腐败之决心",
        "picList": [{
          "url": "http://recc.nosdn.127.net/irec-20180711-d4a7e8986acc0244a738999103f38ddc.jpg"
        }, {
          "url": "http://recc.nosdn.127.net/irec-20180529-54fb2021979d9224f32dedfd9e140248.jpg"
        }]
      }
    },
    "response": {
      "status": 200,
      "json": {
        "code": 0,
        "message": "success",
        "data": {
          "status": 2,
          "sec": {
            "status": 2,
            "article": {
              "status": 1,
              "labels": [
                {
                  "label": 500,
                  "level": 1,
                  "rating": 0.0,
                  "url": null,
                  "detail": {
                    "hint": [
                      "中国打击腐败"
                    ]
                  }
                }
              ]
            },
            "pic": {
              "status": 2,
              "labels": [
                {
                  "label": 100,
                  "level": 2,
                  "rating": 1.0,
                  "url": "http://recc.nosdn.127.net/irec-20180711-d4a7e8986acc0244a738999103f38ddc.jpg",
                  "detail": null
                }
              ]
            },
            "video": null
          },
          "qa": null
        }
      }
    }
  }
```
1. 在Postman的Headers里添加Content-Type为application/json;charset=UTF-8
2. body里面选择raw格式发送请求即可。
3. 如果不行将启动服务也添加utf-8。命令：java -Dfile.encoding=UTF-8 -jar moco-runner-0.12.0-standalone.jar http -p 7001 -c SecurityCheck.json