<!-- TOC -->

- [Flask学习笔记](#flask学习笔记)
    - [安装](#安装)
    - [Hello Flask](#hello-flask)
    - [app.run()的参数](#apprun的参数)
    - [路由](#路由)
        - [URL的变量规则](#url的变量规则)
        - [构造URL](#构造url)
        - [指定HTTP方法](#指定http方法)
        - [静态文件](#静态文件)
        - [集中映射](#集中映射)
    - [模板](#模板)
    - [处理请求](#处理请求)
        - [Request与Response](#request与response)
        - [Cookies](#cookies)
        - [Sessions](#sessions)
        - [文件上传](#文件上传)
        - [文件下载](#文件下载)
        - [重定向](#重定向)
    - [视图](#视图)
        - [视图装饰器](#视图装饰器)
        - [基于方法的视图](#基于方法的视图)
    - [蓝图](#蓝图)

<!-- /TOC -->

# Flask学习笔记
与大而全的tornado相比，Flask是一个Python编写的Web微框架。

## 安装
万能的pip：
```
pip install flask
```

## Hello Flask
flask的程序有两种启动方式，旧版本的flask启动使用如下方式：
```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello Flask!'


if __name__ == '__main__':
    app.run()
```
将这段代码保存到test.py，然后执行如下命令即可运行：
```
python test.py
```
运行成功后，提示：
```
* Serving Flask app "test" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
新版本的官方网站使用如下方式运行：
```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello Flask!'
```
然后先设置一个环境变量, 在Linux下：
```
export FLASK_APP=test2.py
```
在windows下：
```
set FLASK_APP=test2.py
```
然后执行：
```
flask run
```
运行成功的输出信息与第一种方式相似。  
对比这两种方式，还是第一种与其他的Python程序的运行方式一致，我更容易接受。

## app.run()的参数
新版本的Flask中，很多配置是通过环境变量来实现的，我个人对这种方式比较反感，不太容易接受这种方式。在以后的笔记中，我尽量规避环境变量的方式，找到可以通过代码控制的方式。如指定服务端的ip和端口，开启调试模式，都可以通过app.run()的参数进行指定。  
app.run()有如下参数：
* host：主机名
* port：端口
* debug：是否进入调试模式
* load_dotenv：从.env或.flaskenv中读取环境变量的配置
* options：传递给werkzeug.serving.run_simple函数的参数

一般情况下，使用前三个就足够了。

## 路由
相比tornado，flask的路由稍微复杂一些，但复杂也意味着我们可以控制更多的东西。前面例子中的@app.route('/')就是路由装饰器，使用它，可以指定url和HTTP方法。

### URL的变量规则
tornado使用正则表达式来匹配URL，Flask使用了一种规则：/path/\<converter:varname\>，其中，converter是转换器，varname是变量名，Flask允许的转换器有如下几种：
转换器 | 作用
:-- | :--
string | 接受除了斜杠之外的字符串（默认）
int | 接受整数
float | 接受浮点数
path | 可以接受带斜杠的字符串
uuid | 接受UUID字符串
官网给出的例子如下：
```python
@app.route('/user/<username>')
def show_user_profile(username):
    # show the user profile for that user
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    # show the post with the given id, the id is an integer
    return 'Post %d' % post_id

@app.route('/path/<path:subpath>')
def show_subpath(subpath):
    # show the subpath after /path/
    return 'Subpath %s' % subpath
```

### 构造URL
为了得到一个可以访问的URL，Flask提供了url_for()函数，使用它可以构造一个URL：
```python
from flask import Flask, url_for

app = Flask(__name__)

@app.route('/')
def index():
    return 'index'

@app.route('/login')
def login():
    return 'login'

@app.route('/user/<username>')
def profile(username):
    return '{}\'s profile'.format(username)

with app.test_request_context():
    print(url_for('index'))
    print(url_for('login'))
    print(url_for('login', next='/'))
    print(url_for('profile', username='John Doe'))
```
其输出结果为：
```
/
/login
/login?next=/
/user/John%20Doe
```

### 指定HTTP方法
可以通过装饰器的methods参数来指定这个url的HTTP方法，如果不指定，默认是GET。如：
```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        return do_the_login()
    else:
        return show_the_login_form()
```

### 静态文件
Flask默认的静态文件在代码目录的static目录下，将文件放到该目录下后，可以直接通过/static/文件名访问。  
如想通过url_for得到url，可以使用filename参数：
```python
url_for('static', filename='style.css')
```

### 集中映射
使用装饰器创建路由虽然简单，但过分分散，不太容易进行管理，Flask也提供了集中映射的功能，可以直观的看到所有的URL映射：
```python
app.add_url_rule('/foo', view_func=views.foo)
```

## 模板
Flask默认使用Jinja2作为模板引擎它的语法与tornado内置的语法基本相同，只是{% end %}需要用{% endfor %}、{% endif %}代替。它的模板文件需要放在templates目录下。后端渲染模板可以通过render_template：
```python
from flask import render_template

@app.route('/hello/')
@app.route('/hello/<name>')
def hello(name=None):
    return render_template('hello.html', name=name)
```

## 处理请求
### Request与Response
Flask有一个全局的Request对象，通过该对象可以获取请求的具体内容。比如通过request.method对象取得请求的HTTP方法等，如：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask, request

app = Flask(__name__)

@app.route('/')
def hello_world():
    return request.method

if __name__ == '__main__':
    app.run(host='0.0.0.0')
```
一般的，处理函数可以直接return即可，flask可以通过返回值的类型来自动向浏览器发送数据。但需要进行特殊处理的时候，需要通过make_response函数，构造一个Response来返回。

### Cookies
可以通过request对象的cookies属性取得浏览器的Cooikes，可以使用Cooikes[key]来取得，官方建议使用Cooikes.get(key)来获取，这样可以避免key不存在的错误。但实际上，当key不存在时，这两种方式都会抛出异常，都需要额外的处理。  
写入Cooikes则需要通过Response的set_cookie函数来完成，如：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask, request, make_response

app = Flask(__name__)

@app.route('/')
def GetCooikes():
    return request.cookies.get('key')

@app.route('/cooike')
def WriteCooikes():
    resp = make_response()
    resp.set_cookie('key', 'Hello')
    return resp

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

### Sessions
flask也内置了session对象，可以用来保存数据。要想使用session，首先要为app指定一个secret_key：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask, request, make_response, session

app = Flask(__name__)
app.secret_key = 'fdase43209fdl98'

@app.route('/')
def home():
    if session.has_key('name'):
        return 'Hello, %s' % session['name']
    else:
        return u'请登录'

@app.route('/login')
def login():
    session['name'] = 'test'
    return 'ok'

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```
上面的代码登录后首页就可以显示Hello, test，不过换一个浏览器则还是显示请登录。

### 文件上传
后端需要利用 request 的files属性即可获取浏览器上传的文件，这也是一个字典，包含了被上传的文件。如果想获取上传的文件名，可以使用filename属性，不过需要注意这个属性可以被客户端更改，所以并不可靠。更好的办法是利用werkzeug提供的secure_filename方法来获取安全的文件名。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
from flask import Flask, request
from werkzeug.utils import secure_filename

app = Flask(__name__)

@app.route('/upload', methods=['GET', 'POST'])
def upload():
    if request.method == 'POST':
        if request.files.has_key('attachment'):
            f = request.files['attachment']
            f.save(os.path.join('uploads/', secure_filename(f.filename)))
            return "ok"
        else:
            return "no file"
    else:
        return """
        <form action="/upload" enctype="multipart/form-data" method="POST">
            <input type="file" name="attachment"/>
            <button type="submit">提交</button>
        </form>
        """

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

### 文件下载
Flask提供了简单的文件下载函数：send_from_directory
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
from flask import Flask, send_from_directory

app = Flask(__name__)

@app.route('/download/<filename>')
def download(filename):
    return send_from_directory('uploads', filename, as_attachment=True)

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```
我没有进行充分的测试，在使用tornado时，遇到过中文的文件名在不同的浏览器中需要使用不同的编码的情况。如果要实现自己可以控制的文件下载代码，需要先对header做一些设置，这些也需要通过response来进行。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
from flask import Flask, make_response

app = Flask(__name__)

@app.route('/download/<filename>')
def download(filename):    
    with open(os.path.join('uploads', filename), 'rb') as fp:
        content = fp.read()

    resp = make_response(content)
    resp.headers["Content-Type"] = "application/octet-stream"
    resp.headers["Content-Disposition"] = "attachment;filename=%s" % filename

    return resp

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

### 重定向
在有些需要跳转页面的地方，可以使用redirect函数进行重定向：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask, request, make_response, session

app = Flask(__name__)
app.secret_key = 'fdase43209fdl98'

@app.route('/')
def home():
    if session.has_key('name'):
        return 'Hello, %s' % session['name']
    else:
        return redirect(url_for('login'))

@app.route('/login')
def login():
    session['name'] = 'test'
    return redirect(url_for('home'))

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```
当后端处理遇到错误时，可以通过错误处理重定向到错误页面：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import Flask, abort

app = Flask(__name__)

@app.route('/')
def home():
    abort(404)

@app.errorhandler(404)
def page_not_found(error):
    return 'error'

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

## 视图
Flask中每个请求的处理函数被称为视图。

### 视图装饰器
当多个视图需要进行相同的操作时，可以编写一个视图装饰器。我们可以使用视图装饰器很方便的验证用户是否登录。
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *
from functools import wraps

app = Flask(__name__)
app.secret_key = 'fdase43209fdl98'

def login_required(func):
    @wraps(func)
    def decorated_function(*args, **kwargs):
        if not 'user' in session:
            return redirect(url_for('login', next=request.path))
        return func(*args, **kwargs)

    return decorated_function

@app.errorhandler(401)
def gotoLogin(error):    
    return redirect(url_for('login'))

@app.route('/')
@login_required
def home():
    return 'Hello, %s' % session['user']

@app.route('/login')
def login():   
    session['user'] = 'test'
    return redirect(request.args['next'])

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

### 基于方法的视图
Flask路由装饰器可以指定HTTP方法，然后还需要在视图中判断当前是哪一个方法，这种方式比较繁琐，想想tornado只需要在类中定义get/post等方法就可以指定HTTP方法，多简单。其实，Flask也提供类似的类，名为MethodView:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *
from functools import wraps
from flask.views import MethodView

app = Flask(__name__)

# HomeView类
class HomeView(MethodView):
    def get(self):
        return 'get'

    def post(self):
        return 'post'

    def put(self):
        return 'put'

    def delete(self):
        return 'delete'

# 将HomeView类添加到路由
homeView = HomeView.as_view('homes')
app.add_url_rule('/', view_func=homeView)

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```
MethodView也支持视图装饰器：
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *
from functools import wraps
from flask.views import MethodView

app = Flask(__name__)
app.secret_key = 'fdase43209fdl98'

def login_required(func):
    @wraps(func)
    def decorated_function(*args, **kwargs):
        if not 'user' in session:
            return redirect(url_for('login', next=request.path))
        return func(*args, **kwargs)

    return decorated_function

# HomeView类
class HomeView(MethodView):
    decorators = [login_required]
    def get(self):
        return 'Hello, %s' % session['user']

@app.route('/login')
def login():   
    session['user'] = 'test'
    return redirect(request.args['next'])

# 将HomeView类添加到路由
homeView = HomeView.as_view('homes')
app.add_url_rule('/', view_func=homeView)
app.add_url_rule('/login', view_func=login)

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```

## 蓝图
为了使网站功能分类更清晰，开发人员更好的分工合作，Flask提供了很好的协作手段，叫做蓝图（Blueprint），每个同一类的功能可以放到一个蓝图中，然后把多个蓝图注册到app中，即可实现多人协作。  
每个蓝图的url映射都是独立的，在向app注册蓝图时，需要为每个蓝图指定一个基本url，蓝图中的url需要由基本的url和本蓝图中的url结合访问。  
我们把上面登录的例子拆分成三个py，一个是主程序，一个是首页的蓝图，另一个是登录的蓝图。  
homeDp.py:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *
from functools import wraps
from flask.views import MethodView

home_dp = Blueprint('home', __name__)

def login_required(func):
    @wraps(func)
    def decorated_function(*args, **kwargs):
        if not 'user' in session:
            # 这种情况下，因为本蓝图中没有login函数，不能再使用url_for
            return redirect('/login?next=%s' % request.path)
        return func(*args, **kwargs)

    return decorated_function

# HomeView类
class HomeView(MethodView):
    decorators = [login_required]
    def get(self):
        return 'Hello, %s' % session['user']

homeView = HomeView.as_view('homes')
home_dp.add_url_rule('/', view_func=homeView)
```
loginDp.py:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *

login_dp = Blueprint('login', __name__)

@login_dp.route('/')
def login():   
    session['user'] = 'test'
    return redirect(request.args['next'])
```
test.py:
```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

from flask import *
from homeDp import home_dp
from loginDp import login_dp

app = Flask(__name__)
app.secret_key = 'fdase43209fdl98'

app.register_blueprint(home_dp, url_prefix='/')
app.register_blueprint(login_dp, url_prefix='/login')

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=True)
```
蓝图之间是完全独立的，可以为每个蓝图创建自己的目录，并且拥有自己的static和templates目录。