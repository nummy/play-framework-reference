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
  
设置超时时间
++++++++++++

如果需要指定请求的超时时间，可以使用 ``withRequestTimeout``来设置， 如果要一直等待下去，可以使用 ``Duration.Inf`` 作为参数。

.. code-block:: scala
  
  ws.url(url).withRequestTimeout(5000.millis).get()
  
提交表单数据
+++++++++++++
将表单数据以 ``Map[String, Seq[String]]]`` 的形式作为参数传递给 ``post`` 方法：

.. code-block:: scala
  
  ws.url(url).post(Map("key" -> Seq("value")))
  
提交multipart/form数据
++++++++++++++++++++++

如果是提交 ``multipart-form-encoded`` 数据，则需要将 ``Source[play.api.mvc.MultipartFormData.Part[Source[ByteString, Any]], Any]``类型的数据作为参数传递给 ``post`` 方法：

.. code-block:: scala
  
  ws.url(url).post(Source.single(DataPart("key", "value")))

如果是上传文件，则需要将 ``play.api.mvc.MultipartFormData.FilePart[Source[ByteString, Any]]`` 类型的数据传递给 ``post`` 方法:

.. code-block:: scala

  ws.url(url).post(Source(FilePart("hello", "hello.txt", Option("text/plain"), FileIO.fromFile(tmpFile)) :: DataPart("key", "value") :: List()))


提交JSON数据
++++++++++++

使用JSON库即可：

.. code-block:: 

  import play.api.libs.json._
  val data = Json.obj(
    "key1" -> "value1",
    "key2" -> "value2"
  )
  val futureResponse: Future[WSResponse] = ws.url(url).post(data)

提交XML数据
+++++++++++

提交XML数据最简单的方式就是使用XML字面量，XML字面量虽然方便，但是并不快，为了方便起见，可以使用XML视图模板或者JAXB库。

.. code-block:: scala
  
  val data = <person>
    <name>Steve</name>
    <age>23</age>
  </person>
  val futureResponse: Future[WSResponse] = ws.url(url).post(data)

流数据
+++++++

WS还支持流数据，用于上传大型文件，如果数据库支持Reactive Streams，则可以使用流数据：

.. code-block:: scala
  
  val wsResponse: Future[WSResponse] = ws.url(url)
    .withBody(StreamedBody(largeImageFromDB)).execute("PUT")
  
上面l ``largeImageFormDB`` 的数据类型为 ``Source[ByteString, _]``。

过滤请求
++++++++

还可以给WSRequest添加一个请求过滤器，请求过滤器通过继承特质 `` play.api.libs.ws.WSRequestFilter`` 来实现，然后使用 ``request.withRequestFilter(filter)`` 将它添加请求中.

WS提供了一个过滤器的实现，位于 ``play.api.libs.ws.ahc.AhcCurlRequestLogger`` ，它用于将请求的信息以SLF4J日志形式进行记录。

.. code-block:: scala

  ws.url(s"http://localhost:$testServerPort")
  .withRequestFilter(AhcCurlRequestLogger())
  .withBody(Map("param1" -> Seq("value1")))
  .put(Map("key" -> Seq("value")))  
  
将输入以下日志::
  
  curl \
  --verbose \
  --request PUT \
  --header 'Content-Type: application/x-www-form-urlencoded; charset=utf-8' \
  --data 'key=value' \
  'http://localhost:19001/


处理响应结果
------------

响应结果包裹在 ``Future`` 中。

当 ``Future`` 中的计算任务完成是，需要指定一个隐式参数--执行上下文， ``Future``的回调将在该上线文的线程池中执行。我们可以通过依赖注入指定执行上下文

.. code-block:: scala

  class PersonService @Inject()(implicit context: ExecutionContext) {
    // ...
  }

如果不适用依赖注入，也可以使用默认的执行上下文：

.. code-block:: scala
  
  implicit val context = play.api.libs.concurrent.Execution.Implicits.defaultContext

下面的例子使用都使用如下定义的样式类进行序列化和反序列化：

.. code-block:: scala

  case class Person(name: String, age: Int)

将响应解析为JSON
++++++++++++++++

调用 ``response.json`` 将响应数据解析为 ``JSON`` 对象。

.. code-block:: scala
  
  val futureResult: Future[String] = ws.url(url).get().map {
   response =>
      (response.json \ "person" \ "name").as[String]
  }
  
JSON库可以将隐式参数 ``Reads[T]`` 映射成相应的类：

.. code-block:: scala
  
  import play.api.libs.json._

  implicit val personReads = Json.reads[Person]

  val futureResult: Future[JsResult[Person]] = ws.url(url).get().map {
    response => (response.json \ "person").validate[Person]
  }
  
将响应解析为XML
+++++++++++++++

使用 ``response.xml`` 将数据解析为XML：

.. code-block:: scala

  val futureResult: Future[scala.xml.NodeSeq] = ws.url(url).get().map {
  response =>
    response.xml \ "message"
  }

