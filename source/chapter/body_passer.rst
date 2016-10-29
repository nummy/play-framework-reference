第五章 body parser
=================

一个HTTP请求其实就是一个请求头跟着一个请求体。

请求头使用 ``RequestHeader`` 处理，请求体使用 ``BodyParser`` 处理。

由于Play是一个异步框架，所以传统的 ``InputStream`` 并不能用来读取请求体数据。Play使用异步流库 ``Akka Streams`` 来读取数据。

内置解析器
---------

大多数web应用都不用使用自定义解析器，一般我们不需要指明使用哪个解析器，Play会根据请求体重的 ``Content-Type`` 类型来推断要使用的解析器。

