第四章 Session和Flash
====================

如果需要在多个HTTP请求中传递数据，可以使用 ``Session`` 或者 ``Flash`` 。它们的区别在于：

- ``Session`` 中的数据会在整个会话过程中一直保存

- ``Flash`` 中数据只会保存到下一次请求

由于Play中 ``session``或 ``flash`` 的数据都只是保存在接下来的请求中，而不是服务器中，所以对保存的数据大小有限制，最大为4kb。默认的``cookie``名为
``PLAY_SESSION``，可以编辑配置的 ``session.cookieName`` 进行修改。

在session中保存数据
------------------

.. code-block:: scala
  
  Ok("Welcome!").withSession(
    "connected" -> "user@gmail.com")
    
上面的代码会替换整个 ``session`` 信息。如果只是想添加额外的信息，可以使用如下语法：

.. code-block:: scala
  
  Ok("Hello World!").withSession(
    request.session + ("saidHello" -> "yes"))

还可以从 ``session`` 中删除某个字段信息：

.. code-block:: scala
  
  Ok("Theme reset!").withSession(
    request.session - "theme")


获取session数据
--------------

从HTTP请求中获取 ``session`` 信息：

.. code-block:: scala
  
  def index = Action { request =>
    request.session.get("connected").map { user =>
      Ok("Hello " + user)
    }.getOrElse {
      Unauthorized("Oops, you are not connected")
    }


删除整个会话
-----------

.. code-block:: scala

  Ok("Bye").withNewSession

