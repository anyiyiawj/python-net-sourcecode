# flask源码解读之初探
---
先写一个最简单的flask程序。
```python
from flask import Flask
app = Flask(__name__)  #实例化Flask类

@app.route('/')
def hello_world():
    return 'Hello, World!'

if __name__ == '__main__':
    app.run()   #开启服务
```
我们打开flask的源码（0.12版），其中文件很多，我们先看最重要的app文件。这里面有个Flask类（就是上面用到的那个），去看run方法（传入地址和端口），其中最重要的地方时run_simple方法，它来自哪里呢？它来自werkzeug的serving.py。werkzeug是flask底层关于服务器的工具包。我们先调到werkzeug中看一看。


werkzeug包中也有很多文件，我们看到哪里讲哪里，不在一一列举。我们来看下serving.py这个包（提供基础的web服务），首先找到run_simple这个函数。(传入地址，端口和应用)，首先看是否在调试模式，然后把静态文件转为了response，关键在最下面的地方if和else的地方（if是查询改变代码时时候重启），都导向了inner方法。inner方法内，先获取了可用的socket对象，然后，调用make_server来生成Basewsgiserver实例，然后让这个实例serve_forever开启服务。我们来重点放在Basewsgiserver上


Basewsgiserver继承自HTTPServer（讲过的），其初始化采用WSGIRequestHandler处理器（注意，这个不是从wgsiref中来的，是另外定义的，请不要混淆，这里的werkzeug和wsgiref是完全独立的模块，都和http包有关联），绑定app和采用父类的方法初始化，其中的serve_forever没有什么特别，让我们把视角放在**WSGIRequestHandler**处理器类上。


记得处理器最重要的是**handle**方法，我们看下WSGIRequestHandler处理器的handle，它在它的内部调用了父类的handle方法。记得吗，其父类要调用handle_one_request方法，而这个类中改写了这个方法，首先接受到最初的请求，然后解析请求（父类的方法），最后run_wsgi方法。这个run_wsgi方法，先传入环境，在最底部调用内部execute函数（传入应用）。在execute函数中第一行就是**application_iter = app(environ, start_response)**，这不就是我们要找的wsgi接口吗？然后把结果用自己的write写入就可以。

这样我们就可以理解其启动的整个过程，在flask中Flask的实例作为app传入
，最终调用app(environ, start_response)方法，那么app的**__callable__**函数就很关键。也是我们分析应用程序的突破口。（对的，我们今天看到的是如何实现wsgi接口把服务器和应用程序连接起来）


我们下次就从**__callable__**入手，分析怎么把复杂的应用程序通过这个接口传递的。
