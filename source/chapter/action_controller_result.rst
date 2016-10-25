Actions、Controllers与Results
==========================

------------
什么是Action
------------

Play应用中大多数的请求都是通过 ``Action`` 来处理。 ``Action`` 本质上是一个函数， ``(play.api.mvc.Request => play.api.mvc.Result)`` ，它接收请求，处理之后再响应客户端。

.. code-block:: scala

  def echo = Action { request =>
    Ok("Got request [" + request + "]")
  }


----------
创建Action
----------

最简单的方式：

.. code-block:: scala

  Action {
    Ok("Hello world")
  }

接收请求数据：

.. code-block:: scala
  
  Action {implicit request =>
    Ok("Got request [" + request + "]")
  }

指定 ``Bodyparse`` 参数：

.. code-block:: scala
  
  Action(parse.json) { implicit request =>
    Ok("Got request [" + request + "]")
  }


-------------------------
Controllers是Action生成器
-------------------------

``Controllers`` 用于生成 ``Action`` 。

.. code-block:: scala

  package controllers

  import play.api.mvc._

  class Application extends Controller {

    def index = Action {
      Ok("It works!")
    }

  }


---------
简单结果
---------

响应结果通过 ``play.api.mvc.Result`` 来定义。

.. code-block:: scala

  import play.api.http.HttpEntity

  def index = Action {
    Result(
      header = ResponseHeader(200, Map.empty),
      body = HttpEntity.Strict(ByteString("Hello world!"), Some("text/plain"))
    )
  }

不过play提供了快捷方法 ``Ok()`` :

.. code-block:: scala
  
  def index = Action {
    Ok("Hello world!")
  }

其它简便方法：

.. code-block:: scala

  val ok = Ok("Hello world!")
  val notFound = NotFound
  val pageNotFound = NotFound(<h1>Page not found</h1>)
  val badRequest = BadRequest(views.html.form(formWithErrors))
  val oops = InternalServerError("Oops")
  val anyStatus = Status(488)("Strange response type")


-------
重定向
-------

.. code-block:: scala
  
  def index = Action {
    Redirect("/user/home")
  }


----------
TODO 页面
----------

暂未实现的页面

.. code-block:: scala
  
  def index(name:String) = TODO
