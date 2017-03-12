# Python 网络编程入门

---

## 从socket讲起
了解TCP/IP协议以后，我们知道了五层模型，这里的socket（套接字）便是对传输层的抽象。这样我们编程就不用看考虑其下面的细节，可直接利用socket对象来使用TCP/IP协议。我们也可在上面包装来提供http或者ftp等应用层服务。
python中socket模块提供了这个功能，我们先来编辑一个服务器端：
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
这样就简单写好了一服务器和一客户端，任何的网络模块都是对这个模块的封装，具体的内容再去参看《python核心手册》。
下面我们看一下对socket简单封装的模块socketserver
## socketserver模块


