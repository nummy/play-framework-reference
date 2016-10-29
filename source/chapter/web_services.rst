第十七章 web服务
===============

有时候我们需要调用其它站点的HTTP服务，Play通过WS库来支持这些异步调用。

调用WS API包括两个部分，发起请求，处理响应。

发送请求
--------

使用WS之前，需要在build.sbt中添加依赖：

.. code-block:: scala
  
  libraryDependencies ++= Seq(
    ws
  )

接下来就可以在组件中通过声明WSClient的注入来调用WS服务。

.. code-block:: scala

  import javax.inject.Inject
  import scala.concurrent.Future
  import scala.concurrent.duration._

  import play.api.mvc._
  import play.api.libs.ws._
  import play.api.http.HttpEntity

  import akka.actor.ActorSystem
  import akka.stream.ActorMaterializer
  import akka.stream.scaladsl._
  import akka.util.ByteString

  import scala.concurrent.ExecutionContext

  class Application @Inject() (ws: WSClient) extends Controller {

  }

使用 ``ws.url()`` 发送请求：

.. code-block:: scala
  
   val request: WSRequest = ws.url(url)

返回一个 ``WSRequest`` 对象，可以给它指定各种HTTP选项，比如头部信息，可以通过链式调用来构建复杂的请求:

.. code-block:: scala

  val complexRequest: WSRequest =
  request.withHeaders("Accept" -> "application/json")
    .withRequestTimeout(10000.millis)
    .withQueryString("search" -> "play")

最后调用你想使用的HTTP方法结束请求：

.. code-block:: scala
  
  val futureResponse: Future[WSResponse] = complexRequest.get()
 
请求结果以 ``Future[WSResponse]`` 形式返回， ``WSResponse`` 包含了服务端返回的数据。
