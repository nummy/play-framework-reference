第十八章 Akka
============

Akka用于构建高并发可扩展应用，它的容错性很高。

actor系统
----------

Akka工作在 ``actor`` 系统之上， ``actor`` 系统用于管理资源。

Play应用定义了一个特殊的 ``actor`` 系统，这个系统的生命周期与 ``play`` 应用的生命周期保持一致。

定义actors
++++++++++

使用Akka之前，需要先创建一个 ``actor`` ：

.. code-block:: scala
  
  import akka.actor._

  object HelloActor {
   def props = Props[HelloActor]
  
   case class SayHello(name: String)
  }

  class HelloActor extends Actor {
    import HelloActor._
  
    def receive = {
      case SayHello(name: String) =>
        sender() ! "Hello, " + name
    }
  }

上面的 ``actor`` 遵守了Akka规范：

- 发送或者接受的消息以及协议， 都定义在伴生对象中

- 定义了props方法


创建和使用actors
----------------

创建actor，需要使用 ``ActorSystem`` ，通过依赖注入实现：

.. code-block:: scala

  import play.api.mvc._
  import akka.actor._
  import javax.inject._
  
  import actors.HelloActor

  @Singleton
  class Application @Inject() (system: ActorSystem) extends Controller {

    val helloActor = system.actorOf(HelloActor.props, "hello-actor")

   //...
  }

``actorOf`` 方法用来创建 ``actor`` ，注意这里 ``Controller`` 声明为单例类，这是因为 ``actor`` 与类相关联，不能创建两个名字相同的 ``actor``。

发送消息
--------

``actor`` 最基本的用法就是发送消息，当发送消息给 ``actor`` 的时候，并没有返回。

但是HTTP是需要返回的，这时可以返回 ``Future`` 结果类型。

.. code-block:: scala

  import play.api.libs.concurrent.Execution.Implicits.defaultContext
  import scala.concurrent.duration._
  import akka.pattern.ask
  implicit val timeout: Timeout = 5.seconds

  def sayHello(name: String) = Action.async {
    (helloActor ? SayHello(name)).mapTo[String].map { message =>
      Ok(message)
    }
  }

需要注意的是:

- 使用 ``？`` 操作符表示等待返回

- 返回结果是 ``Future[Result]`` 类型

- 需要声明隐式参数 ``timeout``

依赖注入 ``actors`` 
------------------

