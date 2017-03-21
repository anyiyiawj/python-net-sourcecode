# flask源码解析之概述

---
上文可知，我们的应用程序的入口在flask.app中Flask类中的**__call__**的地方。看到这里面就用了我们的wsgi接口（environ和start_response)，然后转调用wsgi_app函数。


在wsgi_app函数中，首先，request_context(environ)来构造一个请求上下文，压入栈中，在最后又出栈(这里以后再来分析，我们将来会花一篇来分析，现在当作黑箱）。然后通过full_dispatch_request来返回Response对象，我们来在这个函数。在full_dispatch_request中我们可以看到很多钩子方法，重点在dispatch_request的方法，然后用finalize_request来处理成会Response。(以后讲这个Response的如何生成）。


你可能对于为什么返回Response对象后为什么不直接返回Response，而是response(environ, start_response)感到奇怪。其实这里的Response并不等于我们在 wsgi接口中返回的可迭代的主题。它是一个包含头部，状态和主题的一个类，我们在werkzeug.wrappers中Response类有`__call__`的方法。解析出来这三个组成，并调用start_response方法，返回可迭代路径。
```python
  def __call__(self, environ, start_response):
        app_iter, status, headers = self.get_wsgi_response(environ)
        start_response(status, headers)
        return app_iter
```
下文开始把这些东西慢慢讲细。
