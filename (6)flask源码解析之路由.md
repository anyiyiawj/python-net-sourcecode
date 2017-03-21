# flask源码解析之路由
---
## 路由过程
---
**路由过程是根据请求的url找到函数的过程。**我们今天来分析下flask的路由过程是什么样的。


做过flask开发的，应该知道flask建立路由规则有两种方法：

 1. @app.route()的装饰器
 2. app.add_url_rule(rule,endpoint=None，view_func=None)的函数


在叙述内涵之前，我们先对端点值这个东东弄清楚，下例:
```python
def give_greeting(name):
    return 'Hello, {0}!'.format(name)
app.add_url_rule('/greeting/<name>', 'give_greeting', give_greeting)#两种方法等价，传入参数见上
```
你会问端点值为什么和视图函数相同，和为什么要有端点值，只要视图函数不就行了。


首先，端点值默认等于视图函数，也可以设置为不等于，如
```python
@app.route('/greeting/<name>', endpoint='say_hello')#这时的端点值为say_hello
def give_greeting(name):
    return 'Hello, {0}!'.format(name)
```
那端点值有什么作用呢？也就是为什么需要这个。因为flask有一个url_for的函数，传入端点值就可以得到想要的url，这样我们就可以改变设置的url，而不用改变每一个引用它的地方。


其实本质上的路由过程是**先把url映射到端点上，通过端点映射到视图函数上。**为什么要有中间这个过程，是因为蓝本的存在，可以把应用分割成一个个以命名空间区分的小部分。如果这样的话，不同蓝本中的路由相同则不能应用。所以把指定的蓝本名称作为端点值的一部分就可来实现。
## 内部如何增加路由

---

我们再看下内部的体现，查看Flask类下的route方法和add_url_rule方法，发现在内部的route方法只是在包装了add_url_rule方法的装饰器，所以这两种方法是完全等价的，我们看下add_url_rule方法。


在这个函数中，先不看endpoint和methods的处理，其主要调用了内部的url_rule_class方法来创造了werkzeug.routing中的Rule类，然后再url_map中更新路由地图（为werkzeug.routing中的Map类），然后再view_functions字典中存入视图函数。


我们看下werkzeug.routing中的类，

 1. Rule类传入路由和端点值，构成路由规则
 2. Map类，传入一个Rule的列表，其实例中的bind方法绑定域名，返回MapAdapter对象。
 3. MapAdapter，其match方法（传入匹配的路由和方法），用实现compile的正则表达式去匹配给出的真是路径信息，把所有匹配组建转化成对应的值，返回端点值和字典，也可以报错和重定向。
 
 ## 如何从url到视图函数
 
 ---
 
**werkzeug的过程就是通过url找到处理该url的endpoint过程。**flask内部是endpiont到view_point的过程。


返回上文提到的dispatch_request的函数入手，首先他从请求上下文栈顶拿到request，找到其url_rule，根据rule中的endpoint在view_functions中找到视图函数，传入自己的请求的参数。

其中关于_request_ctx_stack保存的是RequestContext对象，在ctx.py中。有初始化和match_request方法。其中初始化的时候调用create_url_adapter方法，把app的url_mapbind到环境变量。而后调用match_request其实调用了url_adapter.match方法，进行了实际的匹配工作。我们使用url_rule.endpoint就是匹配到的端点值。


这样我们就知道了添加路由和根据路由到端点值，再到函数的过程。下面我们看下上下文的东西。
