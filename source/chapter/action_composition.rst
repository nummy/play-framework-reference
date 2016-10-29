第六章 action组合
================

自定义action
-----------

创建 ``action`` 的方法定义在特质 ``ActionBuilder`` 中，我们创建的 ``action`` 实际上是 ``ActionBuilder`` 特质的实例。

如果要实现自定义 ``action`` ，只需要继承特质  ``ActionBuilder`` ，并实现 ``invokeBlock`` 方法，下面实现一个自定义 ``action`` ，它能记录每个访问请求。

.. code-block:: cala
  
  import play.api.mvc._

  object LoggingAction extends ActionBuilder[Request] {
    def invokeBlock[A](request: Request[A], block: (Request[A]) => Future[Result]) = {
      Logger.info("Calling action")
      block(request)
    }
  }
  
现在可以使用刚才定义的 ``LogginAction`` 了：

.. code-block:: scala
  
  def index = LoggingAction {
    Ok("Hello World")
  }


组合action
----------

在很多应用中，我们可能会定义很多类型的 ``Action`` 创建器，有的需要认证，有的可以记录日志。

那么可不可以重用这些代码呢？

