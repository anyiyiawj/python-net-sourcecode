# flask源码解析之上下文

---


## 简介
---
上下文：一段程序的外部变量值集合


flask把上下文作为**类似全局变量**的东西来使用，之所以成为类似因为在多线程或者多协程的情况下，每一个访问都是独特的，但是对于这个访问本身来说是全局的。


python的threading模块中的local变量，就可以某个变量访问的时候，只访问自己的数据。flask的实现方法于之类似，它有两种上下文application context和request context。定义在global.py文件中。


global文件中，有application context和request context，其中application context演化出来current_app和g，而request context演化出来request和session。用到了LocalStack和LocalProxy这两个东西，有它们我们可以动态的获取两个上下文的内容，在并发程序中每个视图函数都会看到自己的上下文。

## 实现Local
---

其中LocalStack和LocalProxy为werkzeug中的local.py中定义的。其中还有Local，我们先来看下这个，它类似threading.local，看到其内部数据保存在**__storage__这个字典中**中，后续访问都是对该属性的操作，其内部都调用**__ident_func__**来找到线程的id，进而找到对应内部的字典。其中**__release_local__方法**清空当前线程的数据，然后定义了get，set，del等方法，然后用**__call__**方法来创建一个LocalProxy对象。


LocalStack是基于Local实现的栈结构，提供了隔离的栈访问，提供了push,pop,top方法。其**__call__**方法返回当前线程或者协程栈顶元素的代理对象。我们看到的request_context就是LocalStack的实例。它将当前的请求都放会栈里，什么时候使用时读取。


LocalProxy是一个Local对象的代理，负责把自己的操作转发给内部的Local对象。其中_get_current_object方法，来去对应的对象。


## 回到程序内部
---

我们回到global来看我们是怎么实现的
```python

def _lookup_req_object(name):
    top = _request_ctx_stack.top
    if top is None:
        raise RuntimeError(_request_ctx_err_msg)
    return getattr(top, name)#返回栈顶的name属性

_request_ctx_stack = LocalStack()#请求上下文栈
request = LocalProxy(partial(_lookup_req_object, 'request'))
session = LocalProxy(partial(_lookup_req_object, 'session'))
```
怎么把上下文放入栈中，其中在Flask的wsgi_app中
```python
ctx = self.request_context(environ)#构造了一个请求上下文，包装为request上下文
ctx.push()#压入栈
```
其中request_context创建RequestContext的实例（传入字典），然后在ctx中为每个request context保存了请求信息，初始化的最后调用了match_request来实现路由的逻辑匹配，其中push把应用和请求上下文的信息保存到栈上，保存session；pop反之。


好，我们已经知道到该的过程，那么在RequestContext的初始化中调用Request（self，environ）是怎么把environ包装成Request的呢？然后又是如何完成最终的响应，请看下文
