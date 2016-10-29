第九章 异步HTTP编程
==================

处理异步结果
-----------

Play原生支持异步HTTP编程，它以异步无阻塞的方式处理客户端请求。

Play中 ``controller`` 默认是异步，在Play运行过程中， ``action`` 的代码必须执行速度比较快，否则会阻塞，那么如何处理需要计算很长时间的任务呢？

答案是使用 ``future`` 作为返回结果。

``Future[Result]`` 最终将赋值为 ``Result`` 类型，通过将 ``Future[Result]`` 替换为 ``Result`` ，我们就可以快速生成响应而不用等待。

客户端会一直处于阻塞等待响应，但是服务端不会。

为了返回 ``Future[Result]`` ，需要先创建一个 ``Future`` ，以便在以后给 ``Result`` 赋值。

.. code-block:: scala
  
  import play.api.libs.concurrent.Execution.Implicits.defaultContext

  val futurePIValue: Future[Double] = computePIAsynchronously()
  val futureResult: Future[Result] = futurePIValue.map { pi =>
    Ok("PI value computed: " + pi)
  }


返回异步结果
-----------

之前我们一直使用 ``apply`` 来生成 ``action`` ，为了返回异步结果，需要使用 ``Action.async`` 方法：

.. code-block:: scala
  
  import play.api.libs.concurrent.Execution.Implicits.defaultContext

  def index = Action.async {
    val futureInt = scala.concurrent.Future { intensiveComputation() }
    futureInt.map(i => Ok("Got result: " + i))
  }


``Actions`` 默认也是异步的。

处理超时
-------

为了避免客户端一直处于等待中，可以使用 ``promise timeout`` 来处理：

.. code-block:: scala

  import play.api.libs.concurrent.Execution.Implicits.defaultContext
  import scala.concurrent.duration._

  def index = Action.async {
    val futureInt = scala.concurrent.Future { intensiveComputation() }
    val timeoutFuture = play.api.libs.concurrent.Promise.timeout("Oops", 1.second)
    Future.firstCompletedOf(Seq(futureInt, timeoutFuture)).map {
      case i: Int => Ok("Got result: " + i)
      case t: String => InternalServerError(t)
    }
  }


流式响应
-------

在现实中，我们可能需要发送大量的数据，这时可以采用流式响应：

.. code-block:: scala
  def index = Action {

    val file = new java.io.File("/tmp/fileToServe.pdf")
    val path: java.nio.file.Path = file.toPath
    val source: Source[ByteString, _] = FileIO.fromPath(path)
    
   Result(
     header = ResponseHeader(200, Map.empty),
     body = HttpEntity.Streamed(source, some(file.length), Some("application/pdf"))
   )
  }


分发文件
-------

在Play中分发文件也很方便：

.. code-block:: scala

  def index = Action {
   Ok.sendFile(new java.io.File("/tmp/fileToServe.pdf"))
  }

Play会自动计算 ``Conten-Type`` 和 ``Content-Disposition``，不过也可以自定义：

.. code-block:: scala
  
  def index = Action {
    Ok.sendFile(
      content = new java.io.File("/tmp/fileToServe.pdf"),
      fileName = _ => "termsOfService.pdf"
   )
  }

如果你想将文件直接显示在浏览器中，可以这样设置：

.. code-block:: scala

  def index = Action {
    Ok.sendFile(
      content = new java.io.File("/tmp/fileToServe.pdf"),
      inline = true
    )
  }


块响应
-----

如果后台数据是动态生成的，这时没法计算数据大小，只能分块发送。

.. code-block:: scala

  def index = Action {
    val CHUNK_SIZE = 100
    val data = getDataStream
    val dataContent: Source[ByteString, _] = StreamConverters.fromInputStream(data, CHUNK_SIZE)
  
    Ok.chunked(dataContent)
  }

当然，我们也可以使用任意 ``Source`` 的块数据：

.. code-block:: scala

  def index = Action {
    val source = Source.apply(List("kiki", "foo", "bar"))
    Ok.chunked(source)
  }
