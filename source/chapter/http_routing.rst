第二章 HTTP 路由
================

内置HTTP路由
-----------

路由用于将HTTP请求导向 ``Action`` 。

HTTP请求被MVC视为事件处理，事件包括两部分信息：

- 请求路径，包括查询字符串

- 请求方法

路由表定义在 ``conf/routes`` 文件中，它也会被编译，如果路由规则书写错误，Play将抛出异常。

依赖注入
-------

Play支持生成两种类型的路由：

- 依赖注入路由

- 静态路由

默认是依赖注入路由，如果需要使用静态路由，需要在 ``build.sbt`` 中添加如下配置：

.. code-block:: scala
  
  routesGenerator := StaticRoutesGenerator


路由文件格式
-----------

``conf/routes`` 中定义了应用所有的路由规则，每个路由包括了请求方法和URL规则，它们与 ``Action`` 的调用相关联。

例如，如下路由规则：

.. code-block:: scala
  
  GET   /clients/:id          controllers.Clients.show(id: Long)

每个路由规则都以请求方法开始，然后是URL规则，最后是 ``Action`` 调用的定义。

也可以在文件中添加注释，以 ``#`` 开头：

.. code-block:: scala
  
  # Display a client.
  GET   /clients/:id          controllers.Clients.show(id: Long)


也可以使用别的路由表文件，使用 ``->`` ：

.. code-block:: scala
  
  ->      /api                        api.MyRouter


HTTP方法
--------

Play支持的HTTP方法包括 ``GET`` , ``POST`` , ``PUT`` ,  ``DELETE`` ,  ``HEAD`` 。

URL规则
-------

URL规则定义了路由的请求路径，请求路径可以包含动态部分。

*******
静态路径
*******


例如，定义 ``GET /clients/all`` 规则：

.. code-block:: scala
  
  GET   /clients/all          controllers.Clients.list()

*******
动态路径
*******

如果你需要从路由中获取 ``client`` 的 ``id`` ，可以这样配置：

.. code-block:: scala
  
  GET   /clients/:id          controllers.Clients.show(id: Long)


一个路由规则可以有多个动态部分。

默认的路由匹配规则实际由正则表达式 ``[^/]+`` 表示。

如果需要匹配包含 ``/`` 的URL，可以使用 ``*id`` 的语法，它会采用 ``.*`` 的正则表达式：

.. code-block:: scala
  
  GET   /files/*name          controllers.Application.download(name)


例如，对于 ``GET /files/images/logo.png`` ，``name`` 将匹配 ``images/logo.png`` 。

Play还支持自定义URL规则，使用 ``$id<regex>`` 语法：

.. code-block:: scala
  
  GET   /items/$id<[0-9]+>    controllers.Items.show(id: Long)


调用Action生成器方法
-------------------

路由定义的最后一部分就是调用 ``Action``生成方法，这部分必须定义一个合法的方法，该方法返回一个 ``Action`` 类型的值。

如果方法没有定义任何参数：

.. code-block::

GET   /                     controllers.Application.homePage()

如果方法定义了参数，则参数值将从请求URI或者请求字符串中获取：

.. code-block:: scala
  
  # Extract the page parameter from the path.
  GET   /:page                controllers.Application.show(page)

  # Extract the page parameter from the query string.
  GET   /                     controllers.Application.show(page)


下面是对应的方法：

.. code-block:: scala
  
  def show(page: String) = Action {
    loadContentFromDatabase(page).map { htmlContent =>
      Ok(htmlContent).as("text/html")
    }.getOrElse(NotFound)
  }

********
参数类型
********

如果参数类型为 ``String`` ，可以不注明参数类型，如果需要将参数转换为特定的 ``Scala`` 类型，需要明确指定参数类型：

.. code-block:: scala
  
  GET   /clients/:id          controllers.Clients.show(id: Long)

``show`` 方法也需要指定参数类型：

.. code-block:: scala
  
  def show(id: Long) = Action {
    Client.findById(id).map { client =>
      Ok(views.html.Clients.display(client))
    }.getOrElse(NotFound)
  }


*********
指定参数值
*********

有时候需要指定参数的值：

.. code-block:: scala
  # Extract the page parameter from the path, or fix the value for /
  GET   /                     controllers.Application.show(page = "home")
  GET   /:page                controllers.Application.show(page)


*************
设置参数默认值
*************

有时候还需要设置参数默认值：

.. code-block:: scala

  # Pagination links, like /clients?page=3
  GET   /clients              controllers.Clients.list(page: Int ?= 1)


*******
可选参数
*******

还可以设置可选参数：

.. code-block:: scala
  
  # The version parameter is optional. E.g. /api/list-all?version=3.0
  GET   /api/list-all         controllers.Api.list(version: Option[String])



路由权重
-------

优先匹配首先定义的规则

反向路由
-------

也可以通过调用的方法反向生成URL，对于路由规则中的 ``controller`` ，play会在 ``routes`` 目录中生成一个反向控制器，返回 ``play.api.mvc.Call`` 。

``play.api.mvc.Call`` 定义了一个HTTP调用，它提供了请求方法和URI。

例如：

.. code-block:: scala

  package controllers

  import play.api._
  import play.api.mvc._

  class Application extends Controller {

    def hello(name: String) = Action {
      Ok("Hello " + name + "!")
    }

  }

映射到路由表：

.. code-block:: scala

  # Hello action
  GET   /hello/:name          controllers.Application.hello(name)

可以反向获取 ``hello`` 方法的URL：

.. code-block:: scala
  
  // Redirect to /hello/Bob
  def helloBob = Action {
    Redirect(routes.Application.hello("Bob"))
  }



默认路由
-------

Play提供了一些默认的路由：

.. code-block:: scala

  # Redirects to https://www.playframework.com/ with 303 See Other
  GET   /about      controllers.Default.redirect(to = "https://www.playframework.com/")

  # Responds with 404 Not Found
  GET   /orders     controllers.Default.notFound

  # Responds with 500 Internal Server Error
  GET   /clients    controllers.Default.error

  # Responds with 501 Not Implemented
  GET   /posts      controllers.Default.todo


