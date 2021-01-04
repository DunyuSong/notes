#  Flask
## 基本使用
### 请求映射@app.route('/path')
```python
from flask import Flask

# application对象
app = Flask(__name__)


# Flask中的route()装饰器用于将URL绑定到函数.
# application对象的app.add_url_rule('/', 'hello', hello_world)函数也可用于将URL与函数绑定
@app.route('/hello')
def hello_world():
    return "Hello Flask"

# if __name__ == '__main__': 确保服务器只会在该脚本被 Python 解释器直接执行的时候才会运行，而不是作为模块导入的时候。
if __name__ == "__main__":
    app.run()
```
### 请求重定向 redirect
```python
from flask import Flask, redirect, url_for

app = Flask(__name__)


@app.route('/admin')
def hello_admin():
    return 'Hello Admin'


@app.route('/guest/<guest>')
def hello_guest(guest):
    return 'Hello %s as Guest' % guest


# 将请求重定向到另一个资源
@app.route('/user/<name>')
def hello_user(name):
    if name == 'admin':
        # 该函数接受函数的名称作为第一个参数，以及一个或多个关键字参数，
        return redirect(url_for('hello_admin'))
    else:
        return redirect(url_for('hello_guest', guest=name))


if __name__ == '__main__':
    app.run(debug=True)
```
