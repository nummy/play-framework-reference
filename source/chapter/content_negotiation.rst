第七章 内容协商
==============

内容协商
-------

HTTP通过 ``Accept`` 头部俩指明请求体的格式：

.. code-block:: scala
  
  val list = Action { implicit request =>
    val items = Item.findAll
    render {
      case Accepts.Html() => Ok(views.html.list(items))
      case Accepts.Json() => Ok(Json.toJson(items))
    }
  }

``Accepts.Html()`` 与 ``Accepts.Json()`` 都是提取器。


请求提取器
---------

Play 的 ``Accepts`` 支持以下 ``MIME`` 类型：

- Xml

- Html

- Json

- JavaScript

也可以自定义 ``MIME`` 类型，只需使用 ``play.api.mvc.Accepting``类即可。

.. code-block:: scala
  {
    val AcceptsMp3 = Accepting("audio/mp3")
    render {
      case AcceptsMp3() => ???
    }
  }
