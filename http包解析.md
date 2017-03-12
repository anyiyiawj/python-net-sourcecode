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
我们从http.server入手，其中内置了一个test函数，来创建测试服务器。其中传入处理器类为BaseHTTPRequestHandler，服务器类为HTTPServer，协议为HTTP/1.0，port=8000。我们看下函数内容为创建服务器类实例httpd，然后让其server_forever。（类似上篇文章中的样子吧），我们来看下HTTPServer。


HTTPServer继承自TCPServer类，其初始化同上TCPServer类。（开启绑定和开始监听）。然后其server_forever方法，没有改写，故按父类TCPServer，实现了接受和关闭，处理的过程交给BaseHTTPRequestHandler。


BaseHTTPRequestHandler继承自StreamRequestHandler。上篇分析可得关键在于handle方法，这里的handle方法，调用handle_one_request方法。这个方法，首先读取了请求复制给raw_requestline，然后调用parse_request方法来解析请求（解析出来的command为方法，path为路径，request_version为http版本）。看看调用什么方法，最后写入刷新。


先写到这里，因为网络上的资源很少，先写到这里，以后更新。







