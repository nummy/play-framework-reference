第十六章 缓存
============

Play 默认使用 ``EHCache`` 实现缓存API。你也可以自定义插件实现。

添加依赖
--------

在 ``build.sbt`` 中添加依赖：

.. code-block:: scala
  
  libraryDependencies ++= Seq(
    cache
  )

访问缓存API
-----------

缓存API由 ``CacheApi`` 对象提供，可以以依赖的形式插入到对象中。

.. code-block:: scala
  
  import play.api.cache._
  import play.api.mvc._
  import javax.inject.Inject

  class Application @Inject() (cache: CacheApi) extends Controller {

  }

保存数据
+++++++

.. code-block:: scala
  
  cache.set("item.key", connectedUser)

还可以设置有效时间：
.. code-block:: scala
  
  import scala.concurrent.duration._

  cache.set("item.key", connectedUser, 5.minutes)

获取数据
+++++++

.. code-block::scala
  
  val maybeUser: Option[User] = cache.get[User]("item.key")

当没有找到数据的时候，还可以提供一个可调用对象作为附加参数。

.. code-block:: scala
  
  val user: User = cache.getOrElse[User]("item.key") {
    User.findById(connectedUser)
  }
  
删除数据
++++++++

.. code-block:: scala
  
  cache.remove("item.key");


访问不同的缓存
-------------

默认情况下缓存保存在 ``play`` 中，如果需要保存到不同的缓存，需要在 ``application.conf`` 中进行配置。

.. code-block:: scala

  play.cache.bindCaches = ["db-cache", "user-cache", "session-cache"]

接下来就可以使用 ``NamedCache`` 来访问这些缓存了：

.. code-block:: scala
  
  import play.api.cache._
  import play.api.mvc._
  import javax.inject.Inject

  class Application @Inject()(
      @NamedCache("session-cache") sessionCache: CacheApi
  ) extends Controller {

  }


缓存HTTP响应
-----------

Play的HTTP响应可以被缓存然后再使用，使用 ``Cached`` 类创建缓存：

.. code-block:: scala
  
  import play.api.cache.Cached
  import javax.inject.Inject

  class Application @Inject() (cached: Cached) extends Controller {

  }

使用固定的键来缓存响应结果：

.. code-block:: scala
  
  def index = cached("homePage") {
    Action {
      Ok("Hello world")
    }
  }

如果结果是变化的，可以用不同的 ``key`` 来缓存：

.. code-block:: scala
  def userProfile = Authenticated {
   user =>
      cached(req => "profile." + user) {
        Action {
         Ok(views.html.profile(User.find(user)))
       }
     }
  }
```

缓存控制
--------

控制缓存的结果也非常简单，下面的例子值只缓存响应码为200
的结果。

.. code-block:: scala

  def get(index: Int) = cached.status(_ => "/resource/"+ index, 200) {
    Action {
      if (index > 0) {
        Ok(Json.obj("id" -> index))
     } else {
       NotFound
      }
    }
  }

或者只缓存404几分钟:

.. code-block:: scala

  def get(index: Int) = {
    val caching = cached
      .status(_ => "/resource/"+ index, 200)
     .includeStatus(404, 600)

   caching {
      Action {
        if (index % 2 == 1) {
         Ok(Json.obj("id" -> index))
       } else {
         NotFound
       }
      }
   }
  }


