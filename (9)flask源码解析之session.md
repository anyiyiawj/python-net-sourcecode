# flask源码解析之session
## session解析
---
session和cookie的关系就不再这里讲解了，我这里写下session的大概解析机理：

 1. 请求过来的时候，flask会根据cookie信息创建出session变量，保存在请求的上下文中
 2. 视图函数可以获取session中的信息，实现自己的逻辑处理
 3. flask会在发送response的时候，根据session的值，把它写回到cookie中
##从cookie得到session
---
我么先来分析下，如何从请求中得到session的。通过上文的剖析，我们知道session在RequestContext实例的变量中。我们在ctx.py中找到RequestContext，其初始化时session为None，在其push方法中，有这样的代码：
```python
self.session = self.app.open_session(self.request)
if self.session is None:
    self.session = self.app.make_null_session()
```
如果session在的话，调用open_session的方法，这样就可以在flask程序中调用session。如果session不在，调用make_null_session方法来返回一个空的session值。我们来看下open_session方法。返回session_interface.open_session(self, request)的结果。我们查看session_interface，这是一个SecureCookieSessionInterface的实例，也是操作session的接口。我们再来关注这个类，在session.py的文件中。这个类从名字就可以看出是用来程序和内部的一个接口类。


SecureCookieSessionInterface类有open_session方法，传入app和request。这个方法内部s = self.get_signing_serializer(app)来获取session的签名算法。 然后通过val = request.cookies.get(app.session_cookie_name)在request中取出cookie的值。然后用算法来解析cookie的值。然后在session类的对象传入cookie。


我们把关注点放在session 对象到底是怎么定义的？签名算法是怎么工作的？


首先session类为SecureCookieSession类。这个类就是一个基本的字典，外加一些特殊的属性，比如 permanent（flask 插件会用到这个变量）、modified（表明实例是否被更新过，如果更新过就要重新计算并设置 cookie，因为计算过程比较贵，所以如果对象没有被修改，就直接跳过）。


怎么知道实例的数据被更新过呢？ SecureCookieSession 是基于 werkzeug/datastructures:CallbackDict类 实现的，这个类可以指定一个函数作为 on_update 参数，每次有字典操作的时候（`__setitem__`、`__delitem__`、`clear`、`popitem`、`update`、`pop`、`setdefault`）会调用这个函数。


对于开发者来说，可以把 session简单地看成字典，所有的操作都是和字典一致的。
---

然后，我们看下签名算法，即 get_signing_serializer。这里有四个重要参数：

 1. secret_key:密钥。必须设置
 2. salt：安全加盐
 3. serializer：序列算法
 4. signer_kwargs:其他参数，包括摘要/hash算法（默认sha1）和签名算法（默认hmac）
URLSafeTimedSerializer 是 itsdangerous 库的类，主要用来进行数据验证，增加网络中数据的安全性。itsdangerours 提供了多种 Serializer，可以方便地进行类似 json 处理的数据序列化和反序列的操作。

## 返回时改变cookie值
---

然后我们看下flask如何进行的返回response时，自动把session写回到cookie的。其中我们见说的finalize_response后调用process_response来处理对象。其中有
```python
def process_response(self, response):
    ...
    if not self.session_interface.is_null_session(ctx.session):
        self.save_session(ctx.session, response)
    return response
```
这就是改变的地方，save_session，把当前上下文的session保存在response中，其过程和open_session对应，为调用SecureCookieSessionInterface中的save_session方法，根本为response.set_cookie方法。

---

session 在 cookie 中的值，是一个字符串，由句号分割成三个部分。第一部分是 base64 加密的数据，第二部分是时间戳，第三部分是校验信息。


这样flask基本上原理就明白了，Django的原理和这个运行原理很类似，只不过源码的包过于大。下面我开始关注如何实现多线程，多进程，多路复用，异步等功能让我们的服务器实现并发。

  
