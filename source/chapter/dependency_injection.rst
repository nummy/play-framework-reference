第二十章 依赖注入
===============

Play框架支持运行时依赖注入和编译时依赖注入。

运行时依赖注入是指依赖路径在程序运行时才会创建，如果没有找到相应依赖，运行时并不会报错。

目的
-----

依赖注入实现以下目的：

- 实现松耦合绑定对象

- 避免了全局静态状态

运行时依赖注入
-------------

如果你有一个组件，比如 ``controller`` ，依赖于其他组件，这种情况可以是使用 ``@Inject`` 注解来声明。

``@Inject`` 注解可以用在字段或者构造器中，推荐使用在构造器中。

.. code-block:: scala
  
  import javax.inject._
  import play.api.libs.ws._

  class MyComponent @Inject() (ws: WSClient) {
    // ...
  }


注意 ``@Inject`` 注解必须位于类名之后，构造器参数之前，并且必须有 ``()`` 。

除此之外，``Guice``还支持其他几种依赖注入方式，但是构造器注入是最简洁的方式。

依赖注入控制器
--------------

有两种方式使用依赖控制器。

- 注入路由生成器

- 静态路由生成器

默认情况下，Play会将路由对象的 ``controller`` 声明为依赖，如果需要显示声明这个依赖，可以在  ``build.sbt`` 中进行配置：

.. code-block:: scala

  routesGenerator := InjectedRoutesGenerator


通过在 ``build.sbt`` 中进行如下配置，声明静态路由生成器：

.. code-block:: scala
  
  routesGenerator := StaticRoutesGenerator

推荐使用注入路由生成器。

组件声明周期
------------

依赖注入管理被注入组件的生命周期，当需要这些组件的时候，才会创建，然后注入到其它组件中。

组件生命周期工作方式如下：

- 当需要组件时进行创建，如果被使用多次，默认将会创建多个实例，如果你需要的是一个单例，需要将组件标记为 ``singleton`` 。

- 组件实例都是懒创建的，只有在需要的时候才会创建。

- 组件实例都会自动清除，它组件不再被引用时，将自动清除。

Singletons
-----------

有时候我们只需要创建一个对象实例，比如数据库连接，这时可以使用 ``@Singleton`` 注解实现：

.. code-block:: scala

  import javax.inject._

  @Singleton
  class CurrentSharePrice {
    @volatile private var price = 0

    def set(p: Int) = price = p
    def get = price
  }


清除
----

有些组件在Play关闭之后需要做一些清理工作，这可以通过 ``ApplicationLifecycle`` 组件实现：

.. code-block:: scala

  import scala.concurrent.Future
  import javax.inject._
  import play.api.inject.ApplicationLifecycle

  @Singleton
  class MessageQueueConnection @Inject() (lifecycle: ApplicationLifecycle) {
   val connection = connectToMessageQueue()
   lifecycle.addStopHook { () =>
     Future.successful(connection.stop())
   }

   //...
  }

注意，必须确保注册了stop钩子的类为单例类，否则容易发生内存泄露。

处理循环依赖
------------

循环依赖发生在如下情况：

.. code-block:: scala
  
  import javax.inject.Inject

  class Foo @Inject() (bar: Bar)
  class Bar @Inject() (baz: Baz)
  class Baz @Inject() (foo: Foo)

可以使用 ``Provider`` 解决这个问题：

.. code-block:: scala
  
  import javax.inject.{ Inject, Provider }

  class Foo @Inject() (bar: Bar)
  class Bar @Inject() (baz: Baz)
  class Baz @Inject() (foo: Provider[Foo])
