# Open API
为了便于CI直接触发接口用例的执行，平台对外提供了若干开放接口。
## 接入说明
### 调用路径
https://xxx.com/v1/api/open

### 请求方式
所有的Open API调用方式均为POST请求，请求数据格式为json，对应的Content-Type为application/json

### 请求token及签名密钥
Open API调用所需的token及签名密钥可以通过登录之后在“个人信息”中查看。（待考虑？）

### 通用请求头
所有的Open API都需要在请求头中携带token和sign参数，用于用户身份及权限校验，其中sign参数的生成规则为
> sign=md5(请求的json字符串|密钥)

例如用户的密钥是：99236455a6c70b37 请求json为：
```json
{
    "timestamp":1517567433325,
    "id":125
}
```
则签名值为 {"timestamp":1517567433325,"id":125}|99236455a6c70b37 取32位MD5
> sign=894c16f7a1270460f631a639227a3e60

### 通用请求体
所有的请求体中必须包含timestamp字段（13位时间戳），如：
```json
{
    "timestamp":1517567433325,
    "id":125
}
```
### 请求示例
```

if __name__ == '__main__':
    url = 'https://xxx.com/open/api/history/detail'
    token = ''#token
    key = ''#key
    timestamp = int(round(time.time() * 1000))
    data = {
        "taskId": "bc77b94f-f12b-11e7-9ca0-79b471358865",
        "timestamp": timestamp
    }
    src = json.dumps(data) + "|" + key
    m2 = hashlib.md5()
    m2.update(src)
    sign = m2.hexdigest()
    headers = {
        "token": token,
        "sign": sign,
    }
    print json.dumps(data)
    print sign
    r = requests.post(url, json=data, headers=headers)
    print r.json()["msg"]
```