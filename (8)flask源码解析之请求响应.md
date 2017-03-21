# flask源码解析之请求响应
---
## Request
---
flask的请求对象
```python
from flask import request
with app.request_context(environ):
    assert request.mothod=="POST"
```
它对应了flask.wrappers:Request类，它接受environ字典变量，并提供了很多常用属性和方法。（它不能被应用修改，只读）。它继承自werkzeug.wrappers:Request类，又加入了与flask有关的东西，view_args，blueprint，json处理。


其werkzeug..wrapper中的Request类，基类为BaseRequest。其初始化为绑定environ，没有处理。然后用property对对象进行封装。，里面有很多cache_property的装饰器包装的东西，它类似property，但只有第一次访问的时候才会调用底层的方法，以后只是使用其返回值，如何实现看utils模块。cache_property通过继承自property，并在其内部通过`__set__`和`__get__`的描述符来实现。访问时，调用get方法，会现在obj.`__dict__`来寻找，不存在就调用底层的函数，并把得到的值保存起来，再返回。Request就先讲到这里，我们来看下Response如何实现

## Response
---

**Response**包含三部分：状态栏（HTTP版本，状态吗和说明），头部（用于各种控制和协商），body（服务端返回的数据）。而我们在写程序时，也可以直接写入
```python
@app.routr('/')
def index():
    return 'Hello World!',200,{'X-Foo':'bar'}
```
上文提到最后在full_dispatch_request中提到finalize_request进行处理。这里调用make_response来生成response对象，和process_response对response做后续处理。其中make_response可以把不同的东西装化成response，如果是Response返回本身；如果是字符串返回字符串类型，作为body，自动设置头部；如果是元组则会去进行解析。最后变成Response对象。


Response对象也是在wrappers.py中定义，继承自werkzeug.wrappers的Response，只设置返回html。（1，不要直接操作Response对象，用make_response来生成；2，如果要使用自定义的响应对象，可以覆盖flask app中的response_class属性。


其中werkzeug.wrappers中的Response继承自BaseResponse。其中的属性为可读写的，其body可以用接口来直接读写，其头部使用的是werkzeug.datastructures中的Headers类。和字典基本类似，但是其保存的值有序，并且允许相同的key值存在。


基本的东西都讲完了，我们看下session是如何实现的。
