# Flask

https://dormousehole.readthedocs.io/en/latest/quickstart.html



## 路由

使用route()装饰器来把函数绑定到URL



### 变量规则

- 通过<variable_name>就可以在URL中添加[变量]( https://dormousehole.readthedocs.io/en/latest/quickstart.html#id7). 标记的部分会作为参数传递给函数, 
- 转换器类型: string, int, float, path(类型string,可以包含/), uuid



### 唯一的URL/重定向行为



```PY
@app.route('/projects/')
app.route('/about')
```

- 尾部有斜杠, 访问没有斜杠结尾的URL时Flask会自动重定向,并在尾部加上一个斜巷(/projects/)

- 没有斜杠, 如果访问时尾部加上斜杠, 会得到一个404错误

  

## URL构建

url_for() 用于构建**指定函数**的url. 它把 **函数名称**作为第一个参数, 它可以接受任意个关键字参数, 

- 每个关键字参数对应URL中的变量. 位置变量将添加到URL中作为查询参数.

  

## HTTP 方法

使用route()装饰器的methods参数来处理不同的HTTP方法, 缺省情况下, 为GET

```PY
@app.route('/login', methods = ['GET', 'POST'])
def login():
    if request.method == 'POST':
        return 'login post request'
    else:
        return 'login get request'
```



##   静态文件

使用特定的static短点就可以生成响应的URL 

url_for('static', filename='style.css'). 这个静态文件位置应该是static/style.css

   

## 渲染模板

Flask自定配置Jinja2模板引擎, 使用render_template()方法可以渲染模板. 主食要提供模板名称和需要作为参数传递给模板的变量

Python 代码：

```PY
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

模板代码： 同目录templates/xxx.html

```ht
<!doctype html>
<title>Hello from Flask</title>
{% if name %}
  <h1>Hello {{ name }}!</h1>
{% else %}
  <h1>Hello, World!</h1>
{% endif %}
```



模板对象内部可以使用`url_for()`和`get_flashed_messages()`函数一样访问`config`,`request`,``session`和`g`

> g是某个可以根据需要存储信息的东西。

### 模板继承

模板可以继承，可以是每个页面的特定元素（如页头，导航和页尾）保持一致。

自动转义默认开启。如变量`name`中包含HTML，那么会被自动转义。 可以使用`Markup`类将变量标记为安全的，或在模板中使用`|safe`过滤器

http://172.26.15.151:5000/hello/%3Cp%3Eandy/%3Cdiv%3Etest



## 操作请求

Flask中全局对象request来提供请求信息。



rquest对象是全局对象，如何保证线程安全呢？

### 本地环境

request不是通常意义下的全局对象。实际上是特定环境下本地对象的代理。当一个请求进来，服务器生成新的线程，Flask把当前线程作为活动环境， 并把当前应用和WSGI环境绑定到这个环境（线程）。request此时被代理到该活动环境。



在单元测试时，会遇到由于没有请求对象而导致依赖于请求的代码会突然崩溃的情况。 对策是自己创建一个请求对象并绑定到环境。 最简单的单元测试解决方案是使用[`test_request_context()`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Flask.test_request_context)环境管理器。通过`with`语句可以绑定一个测试请求， 以便于交互。

```py
with app.test_request_context('/hello', method = 'POST'):
    # now you can do something with the request until the
    # end of the with block, such as basic assertions:
    assert request.path == '/hello'
    assert request.method == 'POST'


# 另一种方法是吧整个WSGI环境换地给request_context()方法:

with app.request_context(environ):
    assert request.method == 'POST'
```



### [请求对象](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Request)

常见操作

1. 通过使用`method`属性可以操作当前请求方法， 通过使用`form`属性处理表单数据（在`POST`或者`PUT`请求中传输的数据）
2. 当`form`属性中不存在这个键时会引发一个`KeyError`。如果不像捕捉一个标准错误一样捕捉keyError, 那么会显示一个HTTP 400 Bad Request错误页面。
3. 要操作URL（如？key=value）中提交的参数可以使用args属性`searchword = request.args.get('key', '')`
4. 用户可能会改变URL导致出现一个400请求出错页， 这样降低了用户友好度。 因此， 我们推荐使用`get`或通过捕捉keyError来访问URL参数



## 文件上传

https://dormousehole.readthedocs.io/en/latest/quickstart.html#id17



## Cookies

可以使用响应对象的`set_cookie`方法来设置`cookies`。请求对象的`cookies`属性是一个包含了客户端书暗处的所有`cookies`的字典。



### 读取cookie



```py
from flask import request

@app.route('/')
def index():
	username = request.cookies.get('usernmae', '')
	# use cooies.get(key) insted of cookies[key] to get a
	# keyError if the cookie is missing
```



### 写cookie

```py
from flask import make_response

@app.route('/')
def index():
	resp = make_response(render_template())
	resp.set_cookie('username', 'xxxx')
	return resp
```

[注意]： cookies设置在响应对象上。 通常只是从视图函数返回字符串， Flask会把它们转换为响应对象。 如果想显示的转换， 可以使用[`make_response()`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.make_response)函数， 然后再修改它。



[关于响应](https://dormousehole.readthedocs.io/en/latest/quickstart.html#about-responses)

## 重定向和错误

使用`redirect()`函数可以重定向。使用`abort()`可以更早的退出请求，并返回错误代码。



```py
from flask import abort, redirect,url_for

@app.route('/')
def index():
	return redirect(url_for('login'))

@app.route('/login')
def login():
	abort(401);
	return 'hello'
	
# 缺省情况下，每种代码都会对应一个现实黑白的出错页面， 使用errorHandler装饰器可以定制出错页面。
@app.errorHandler(401)
def page_not_fount(error):
	return render_template('page_not_found.html', 404)
```

[errorHandler](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Flask.errorhandler)  [应用错误处理](https://dormousehole.readthedocs.io/en/latest/errorhandling.html)

[render_template](https://dormousehole.readthedocs.io/en/latest/api.html#flask.render_template)后面的404， 表示页面对页面出错代码是404，即使页面不存在。缺省情况下200表示一切正常。

## 关于响应

视图函数的返回值会自动转换为一个响应对象。转换规则：



1. 如果视图返回的是一个响应对象， 那么直接返回
2. 如果返回的是一个字符串，那么根据这个字符串和缺省值参数生成一个用于返回的响应对象
3. 如果返回的是一个字典，调用jsonify创建一个响应对象
4. 如果返回一个元组， 那么组件中的项目可以通过额外的信息。元组中必须至少包含一个项目，且项目应当由(response, status), (response, headers)或者（response, status, headers)组成。 status的值会重载状态码， headers是一个额外头部值组成的列表或字典
5. 如果以上都不是， 那么flask会假定返回值是一个有效的WSGI应用并把它转换为一个响应对象。

如果响应在视图内部掌控响应对象的结果， 那么可以使用make_response（）函数



假设如下视图:

```py
@app.errorHandler(404)
def not_found(error):
	return render_template('error.html'), 404
```



可以使用**make_response()**包裹返回表达式， 获得响应对象，并对改对象进行修改， 然后再返回：

```py
@app.errorHandler(404)
def not_found(error):
	resp = make_response(render_template('error'), 404)
	resp.headers['x-something'] = 'a value'
	
	return resp
```



## JSON格式API

如果从视图返回一个dict， 那么它被转换为一个JSON响应。



```py
@app.route('/me')
def me_api():
	user = get_curr_user();
	return {
		"username": user.username,
		"theme": user.theme,
		"image"： url_for("user_image", filename=user.name)
	}
```



还可以使用`jsonify()`函数。该函数会序列化任何支持的JSON数据类型。

```py
return jsonify([user.to_json() for user in users])
```



## 会话

session允许不同请求之间存储信息。 该对象相当于用秘钥加密的cookie



**使用会话之前必须设置一个秘钥**

```py
from flask import Flask,session, request, redirect, url_for

app = Flask(__name__)

app.secret_key =  b'_5#y2L"F4Q8z\n\xec]/'

@app.route("/")
def index():
    if 'username' in session:
        return f'Logged in as {session["username"]}'
    return 'You are not logged in'

@app.route('/login', methods = ['POST', 'GET'])
def login():
    if request.method == 'POST':
        session['username'] = request.form['username']
        return redirect(url_for('index'))
    return '''
        <form method="post">
            <p><input type="text" name="username" /></p>
            <p><input type="submit" value="login" /></p>
        </form>
    '''



@app.route('/logout')
def logout():
    #remove the username from the session if it's there
    session.pop('username', None)
    return redirect(url_for('index'))
```



基于cookie的会话说明， Flask会取出会话对象中的值，把值序列化后存储到cookie中。 



## 消息闪现

flask通过闪现系统来提供一个易用的反馈方式（应用，用户接口）。闪现系统的基本工作原理是在请求结束时记录一个消息， 提供且只提供下一个请求使用。 通常通过一个**布局模板**来展现闪现的消息。



[`flash`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.flash)用于闪现一个消息。在模板中，使用[`get_flashed_messages()`](https://dormousehole.readthedocs.io/en/latest/api.html#flask.get_flashed_messages)来操作消息。完整的例子： [闪现消息](https://dormousehole.readthedocs.io/en/latest/patterns/flashing.html)



## 日志

记录服务日志

```py
app.logger.debug('a value for debuging')
app.logger.warning('a warning occurred (%d apples)', 42)
app.logger.error('an error occurred')
```

[logger](https://dormousehole.readthedocs.io/en/latest/api.html#flask.Flask.logger)是一个标准的[Logger](https://docs.python.org/3/library/logging.html#logging.Logger) logger类，更多信息详见官方的[logging](https://docs.python.org/3/library/logging.html#module-logging)文档。

## 集成 WSGI 中间件

如果想要在应用中添加一个 WSGI 中间件，那么可以用应用的 `wsgi_app` 属性 来包装。例如，假设需要在 Nginx 后面使用 [`ProxyFix`](https://werkzeug.palletsprojects.com/en/2.0.x/middleware/proxy_fix/#werkzeug.middleware.proxy_fix.ProxyFix) 中间件，那么可以这样做:

```
from werkzeug.middleware.proxy_fix import ProxyFix
app.wsgi_app = ProxyFix(app.wsgi_app)
```

用 `app.wsgi_app` 来包装，而不用 `app` 包装，意味着 `app` 仍旧指向您 的 Flask 应用，而不是指向中间件。这样可以继续直接使用和配置 `app` 。

## 使用 Flask 扩展

扩展是帮助完成公共任务的包。例如 Flask-SQLAlchemy 为在 Flask 中轻松使用 SQLAlchemy 提供支持。

更多关于 Flask 扩展的内容请参阅 [扩展](https://dormousehole.readthedocs.io/en/latest/extensions.html) 。

## 部署到网络服务器

已经准备好部署您的新 Flask 应用了？请移步 [部署方式](https://dormousehole.readthedocs.io/en/latest/deploying/index.html) 。

