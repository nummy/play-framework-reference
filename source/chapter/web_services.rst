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

带认证的请求
++++++++++++

如果需要HTTP认证，可以在请求中指明认证方式以及相关参数，WS支持的认证模式包括以下几种：

- BASIC

- DIGEST

- KERBEROS

- NTLM

- SPNEGO

.. code-block:: scala
  
  ws.url(url).withAuth(user, password, WSAuthScheme.BASIC).get()

带重定向的请求
+++++++++++++

如果请求结果导致了320或者301重定向，可以自动重定向，而不是再发送一次请求：

.. code-block:: scala
  
  ws.url(url).withFollowRedirects(true).get()
  
带头部信息的请求
++++++++++++++++

HTTP头部信息可以通过一系列的键值对元组来指明：

.. code-block:: scala
  
  ws.url(url).withHeaders("headerKey" -> "headerValue").get()
 
例如，通过设置 ``Content-Type`` 来指定请求中发送的数据类型：

.. code-block:: scala
  
  ws.url(url).withHeaders("Content-Type" -> "application/xml").post(xmlString)
  
带虚拟主机的请求
+++++++++++++++

可以在请求中指明虚拟主机：

.. code-block:: scala

  ws.url(url).withVirtualHost("192.168.1.1").get()
