第三章 处理响应结果
===================

修改默认的Content-Type
-----------------------

Play可以自动根据响应内容推断返回的数据类型。例如：

.. code-block:: scala
  
  val textResult = Ok("Hello World!")

会自动设置 ``Content-Type为text/plain`` 。

.. code-block:: scala
  
  val xmlResult = Ok(<message>Hello World!</message>)

会设置 ``Content-Type为application/xml`` 。

这些是通过 ``play.api.http.ContentType`` 来实现的。

不过我们也可以手动设置返回类型。

.. code-block:: scala

	val htmlResult = Ok(<h1>Hello World!</h1>).as("text/html")

或者：

.. code-block:: scala
	
	val htmlResult2 = Ok(<h1>Hello World!</h1>).as(HTML)


设置HTTP headers
----------------

我们也可以设置或者更新HTTP头部信息。

.. code-block:: scala
	
	val result = Ok("Hello World!").withHeaders(
  		CACHE_CONTROL -> "max-age=3600",
  		ETAG -> "xx")


设置或删除Cookie
----------------

设置Cookie

.. code-block:: scala
	
	val result = Ok("Hello world").withCookies(
  		Cookie("theme", "blue"))
	
删除Cookie：

.. code-block:: scala
	
	val result2 = result.discardingCookies(DiscardingCookie("theme"))

设置并删除Cookie：

.. code-block:: scala
	
	val result3 = result.withCookies(Cookie("theme", "blue")).discardingCookies(DiscardingCookie("skin"))


设置响应数据编码格式
--------------------

Play默认使用UTF-8编码，不过也可以人工指定。只需要声明一个隐式参数转换就可以。

.. code-block:: scala
	
	import play.api.mvc.Codec

	class Application extends Controller {

  		implicit val myCustomCharset = Codec.javaSupported("iso-8859-1")

  		def index = Action {
    		Ok(<h1>Hello World!</h1>).as(HTML)
  		}

	}
	
