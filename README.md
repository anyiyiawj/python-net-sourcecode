# Python 网络编程入门

---

## 从socket讲起
了解TCP/IP协议以后，我们知道了五层模型，这里的**socket（套接字）便是对传输层的抽象**。这样我们编程就不用看考虑其下面的细节，可直接利用socket对象来使用TCP/IP协议。我们也可在上面进行包装来提供http或者ftp等应用层服务。
python中socket模块提供了这个功能，我们先来编辑一个服务器端（python3）：
```python
from socket import *
from time import ctime

HOST=''
PORT=21567
BUFSIZ=1024
ADDR=(HOST,PORT)

tcpSerSock=socket(AF_INET,SOCK_STREAM)#创建socket实例
tcpSerSock.bind(ADDR)#连接本地的地址
tcpSerSock.listen(5)#开始监听
while True:
    print('writing for connection...')
    tcpCliSock,addr=tcpSerSock.accept()#接收到客户端的连接请求
    print('...connected from:',addr)
    
    while True:
        data=tcpCliSock.recv(BUFSIZ)#受到客户端的发送数据
        if not data:
            break
        tcpCliSock.send('[%s],%s')%(ctime().decode('utf-8'),data)#发送数据，发送的数据必须是utf-8格式
    tcpCliSock.close()
tcpSerSock.close()
```
然后在写一下客户端：
```python
from socket import *

HOST='127.0.0.1'
PORT=21567
BUFSIZ=1024
ADDR=(HOST,PORT)

tcpCliSock=socket(AF_INET,SOCK_STREAM)#创建客户端socket
tcpCliSock.connect(ADDR)#连接服务器

while True:
    data=input('>')
    if not data:
        break
    tcpCliSock.send(data)#发送数据
    data=tcpCliSock.recv(BUFSIZ)#接收数据
    if not data:
        break
    print(data.decode('utf-8'))
tcpCliSock.close()
```
这样就简单写好了基于TCP的一服务器和一客户端，任何的网络模块都是对这个模块的封装，具体的内容再去参看《python核心手册》，比如UDP怎么实现。
下面我们看一下对socket简单封装的模块socketserver
## socketserver模块
这个模块是从类的角度来对上述的内容来进行封装，其中所写的类分为三种：
1，**网络模块类**：BaseServer，TCPServer，UDPServer。其中TCPServer，UDPServer继承自BaseServer。（其实UDPServer继承自TCPServer，但是我更倾向于直接继承自BaseServer，这样更容易理解，它之所以继承自TCPServer，是为了写的更加简洁。）
2，**网络模块的混合类**：ForkingMixIn，ThreadingMixIn(混合效果，你可以看到有ThreadingTCPServer，便懂它的作用)
3，**处理器类**：主要是用来处理请求，BaseRequestHandler，StreamRequestHandler，DatagramRequestHandler类，其中后两个继承自BaseRequestHandler（一回来分析怎么来处理请求）
先写如何来应用这个模块来创建服务端：
```python
from socketserver import (TCPServer as TCP,StreamRequestHandler as SRH)
from time import ctime

HOST=''
PORT=21567
ADDR=(HOST,PORT)

class MyRequestHandler(SRH):#定义处理器类
    def handler(self):#定义handle方法
        print('...connect from ',self.client_address)
        self.wfile.write('[%s]%s'%(ctime().encode('utf-8'),self.rfile.readline()))#用wfile和rfile来作为文件描述符来操作，socket就是文件描述符
        
tcpServ=TCP(ADDR,MyRequestHandler)#实例化网络服务器，传入地址，端口和处理器
print('waiting for connention...')
tcpServ.serve_forever()#调用内部服务方法
```
客户端程序同前（我们主要写的程序都是服务器端程序，以后不再提客户端程序）
然后我们分析下，如何启动的程序，**首先先看TCPServer类**，其**初始化**同上创建和绑定了socket对象，使用了父类BaseServer的初始化方法来绑定地址和处理器。然后调用了server_bind和server_activate方法
我们分别看下这两个方法，server_bind的方法给socket绑定地址和端口，server_activate实现了监听功能(你发现这里的东西和你自己用socket写的几乎一模一样)。
然后上面程序中，我们调用了server_forever的方法，我们来看下内部的**server_forever**方法。
我们发现TCPServer中并没有server_forever的方法，我们来看一下其父类的server_forever方法，其内部调用了_handle_request_noblock的方法和 service_actions方法,（注意，只看最基本的东西，不要太在乎细节，先广度优先，以后又需要再去深度研究），我们先看_handle_request_noblock，service_actions暂且不表：
_handle_request_noblock方法，首先用get_request方法（自己去找你会发现是accept方法，得到客户端socket为request和其地址client_address），然后调用了process_request方法，process_request内部又用了finish_request方法，然后调用shutdown_request的方法。（shutdown_request是用了close方法），而finish_request中**直接实例化自己绑定的处理器类来处理**，传入request和client_address和自己。这样我们的目光转向了处理器。（先停一下，我们看看我们现在已经做了什么，首先，我们创建了socket的实例，然后bind，listen，accept；到最后要close，所以处理器完成的就是发送和接受的工作。）
来看**StreamRequestHandler**，先用其父类的初始化函数，绑定了request，client_address,server。然后调用setup，handle，finish。我们看下StreamRequestHandler中，setup用于启动读写的文件描述符，finish来关闭这两个描述符。
欸，handle方法呢，你去翻下最上面的程序，是不是有一个handle方法呢，对，handle方法一般是留给写程序的你来调用，通过对读写描述符的读写来实现发送和接受的。
这样，整个socketserver模块就搞清楚，这也是用python编程的基础。
我们知道socket是对TCP/IP的包装，下一次我们要用http模块在socketserver基础上实现http协议的包装。




