# http模块
---
这篇文章对http模块进行分析来，看下如何对socketserver来进行来包装。


首先http模块是以包的形式来实现的，如下：
```
|http
    |-__init__.py
    |-client.py
    |-cookiejar.py
    |-cookies.py
    |-server.py
```
我们从http.server入手，其中


http.server内置了一个test函数，来创建测试服务器。其中传入处理器类为BaseHTTPRequestHandler，服务器类为HTTPServer，协议为HTTP/1.0，port=8000。我们看下函数内容为创建服务器类实例httpd，然后让其server_forever。（类似上篇文章中的样子吧），我们来看下HTTPServer。


HTTPServer继承自TCPServer类，其初始化同上TCPServer类。（开启绑定和开始监听）。然后其server_forever方法，没有改写，故按父类TCPServer，实现了接受和关闭，处理的过程交给BaseHTTPRequestHandler。



**BaseHTTPRequestHandler**继承自StreamRequestHandler。上篇分析可得关键在于handle方法，这里的handle方法，调用**handle_one_request**方法。这个方法，首先读取了请求复制给raw_requestline，然后调用**parse_request**方法来解析请求（解析出来的command为方法，path为路径，request_version为http版本）。然后**通过"do_"+command来构造出要访问的函数，然后调用这个函数并将处理业务，返回response**


但是由于BaseHTTPRequestHandler没有do_GET或者do_POST的方法，我们看下其子类SimpleHTTPRequestHandler中的do_GET方法，其中先send_head,然后copyfile，最后关闭close文件，就是实现了发送数据。（其中send_head返回response的内容，copyfile是把内容发送）


再来整体看一下这个模块，有这几个模块有如下的类：
```
服务器：
HTTPServer#继承自TCPServer
处理器类：
BaseHTTPRequestHandler#继承自StreamRequestHandler
    |
    V
SimpleHTTPRequestHandler#基础http服务
    |
    V
CGIHTTPRequestHandler#提供cgi接口
```
这样就是实现了http协议。下面我们来看下如何实现wsgi接口。











