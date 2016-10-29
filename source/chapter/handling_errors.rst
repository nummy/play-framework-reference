第八章 异常处理
==============

HTTP程序可以返回两种错误:

- 客户端错误

- 服务器端错误

Play在大多数情况下都可以自动检测到客户端的错误，例如请求头错误，不支持的数据类型等。

Play也能处理服务器端错误，只要你在 ``Action`` 中抛出异常，Play就会返回一个异常的页面给客户端。

Play处理错误的接口是 ``HttpErrorHanlder`` ，它定义了两个方法：

- onClientError

- onServerError

自定义错误处理器
---------------

自定义错误处理可以这样实现：在项目根目录中创建 ``ErrorHandler`` 类，这个类需要继承 ``HttpErrorHandler`` 。例如：

.. code-block:: scala

  import play.api.http.HttpErrorHandler
  import play.api.mvc._
  import play.api.mvc.Results._
  import scala.concurrent._
  import javax.inject.Singleton;

  @Singleton
  class ErrorHandler extends HttpErrorHandler {

    def onClientError(request: RequestHeader, statusCode: Int, message: String) = {
      Future.successful(
        Status(statusCode)("A client error occurred: " + message)
      )
    }

    def onServerError(request: RequestHeader, exception: Throwable) = {
      Future.successful(
        InternalServerError("A server error occurred: " + exception.getMessage)
      )
    }
  }

也可以将这个类放在其他包中，然后在 ``application.conf`` 中指明位置：

.. code-block:: scala

  play.http.errorHandler = "com.example.ErrorHandler"


继承默认的Error Handler
----------------------

Play提供的 ··DefaultHttpErrorHandler·· 在开发环境中可以将错误信息渲染后返回给客户端。

我们可以继承这个类继续使用这个功能：

.. code-block:: scala

  import javax.inject._

  import play.api.http.DefaultHttpErrorHandler
  import play.api._
  import play.api.mvc._
  import play.api.mvc.Results._
  import play.api.routing.Router
  import scala.concurrent._

  @Singleton
  class ErrorHandler @Inject() (
      env: Environment,
      config: Configuration,
      sourceMapper: OptionalSourceMapper,
      router: Provider[Router]
    ) extends DefaultHttpErrorHandler(env, config, sourceMapper, router) {
  
    override def onProdServerError(request: RequestHeader, exception: UsefulException) = {
      Future.successful(
        InternalServerError("A server error occurred: " + exception.getMessage)
      )
    }

    override def onForbidden(request: RequestHeader, message: String) = {
      Future.successful(
        Forbidden("You're not allowed to access this resource.")
      )
    }
  }
