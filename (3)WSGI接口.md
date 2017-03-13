# WSGI接口（wsgiref模块解析）
---
先说下什么是WSGI，WSGI是一个接口规范，**连接在应用和服务器之间**，以便我们很好的开发应用。然后下面是一张图把基本上你听到的东西连接起来：


![image](https://github.com/anyiyiawj/python-net-sourcecode/blob/master/wsgi.png)


其中看到的Flask和Django都是我们要写的应用程序。Tornado功能很多可以做应用程序，也可以作为应用服务器，又是一个python的网络库，功能很强大，这些以后再一一介绍，先谈下WSGI接口是什么。


它提供给用户两个变量，**environ和start_response**，其中environ包装了环境变量和请求的东西，而start_response是为了处理响应的头部。并要求返回一个可迭代的对象作为相应。我们先看下怎么利用WSGI接口编写一个应用程序。
```python
def simple_wsgi_app(environ,start_response):
    status='200 OK'
    headers=[('context-type','text/plain')]
    start_response(status,headers)#响应头
    return ['Hello world!']#响应主题

from wsgiref.simple_server import make_server
httpd=make_server('',8000,simple_wsgi_app)
httpd.server_forever()
```
其中我们用到了python内部提供WSGI接口的包wsgiref，其中这个函数便是应用程序，下面的程序是服务器程序。先看下应用程序，是不是传入了environ和start_response函数，然后对其进行处理，最终返回。


然后我们看下wsgiref怎么样工作的，并且怎么就可以server_forever。我们来看wsgiref的源码，这个包包含的文件如下：
```
|wsgiref
    |-__init__.py
    |-header.py #头部处理
    |-handlers.py #核心负责wsgi程序处理
    |-simple_server.py #简单的wsgiHTTP服务器
    |-util.py #帮助函数
    |-validate.py #wsgi格式严查和校验
```
我们先从simple_server.py中的**make_server**入手，然后再去看看server_forever。首先其创建一个WSGIServer的实例，然后给这个服务器设置应用程序app（对于我们的例子就是simple_wsgi_app），我们先看看这个类WSGIServer。


WSGIServer继承自HTTPServer（和上篇文章的内容相同），用HTTPServer的方法完成初始化，绑定处理器WSGIRequestHandler（一会再来看这个处理器）。然后在这个类中找到set_app的方法，发现把应用程序绑定给application的属性。然后我们来找下server_forever的方法，由于其没有这个方法，我们来从其父类找。根据前两文的分析，应用BaseServer的这个方法，最终把问题交给处理器。但是在WSGIServer中**重写了server_bind的方法**。其中server_bind的时候调用了setup_environ方法来建立环境变量，放到env的字典中。好，这样我们开始放心的看下处理器WSGIRequestHandler。


其中，WSGIRequestHandler继承自BaseHTTPRequest类，老规矩先看handle方法。handle方法中也同父类中，接受请求解析请求。然后不同点在创建一个ServerHandler的实例，（其中调用了get_environ函数来对请求的环境变量进一步封装到env中返回），然后把自己作为request_handler传入，最后调用run方法。所以我们要把精力集中到ServerHandler类中。


ServerHandler继承自SimpleHandler（handlers.py文件），我们看下SimpleHandler的初始化，将环境变量赋值为base_env字段。然后我们看下run方法（传入app函数，得在SimpleHandler的父类BaseHandler中寻找）


run方法中，首先使用setup_environ函数（在setup_environ函数中，先获取现在的变量，再用add_cgi_vars合并，赋值给self.environ中），然后调用application（self.environ,self.start_response)来调用我们的应用程序，这里就是我们应用程序和服务器程序的WSGI接口。


我们联系我们的应用程序看下，start_respose方法（先通过start_response处理，在调用write函数来写入）。然后application返回的结果赋值给了result字段，然后调用finish_reposne函数来把可迭代的响应写入程序，并关闭响应。
其主要过程在下面：
```
WSGIServer     |  |WSGIRequestHandler|    |ServerHandler
1，封装socket， |  |1，建立environ     |    |1，补充environ
建立连接        |->|2，把请求交给       |->  |2，创建start_response
2，解析http请求 |  |wsgiserverhandler |    |3，调用wsgi_app
3，请求交给     |                  
handler        |
```
最后列一下主要出现的类：
```
1，BaseHandler
    |
    V
  SimpleHandler
    |         \
    V          V
ServerHandler  BaseCGIHandler(没有用到）
                  |
                  V
              IISCGIHandler(没有用到）
2，WSGIServer继承自HTTPServer
3，WSGIRequestHandler继承自BaseHTTPRequestHandler
```

这就是wsgiref模块的大概使用和内部机理，相信你对WSGI的理解更加深入，下一篇开始入手flask框架的内容







