模板常见用法
===========

Play框架中，模板其实就是函数，它可以被暴露在任意位置。下面是模板的常见用法。

Layout
------

首先创建模板 ``views/main.scala.html`` ，它将作为其它模板的基础模板。

.. code-block:: scala
  @(title: String)(content: Html)
  <!DOCTYPE html>
  <html>
    <head>
      <title>@title</title>
    </head>
   <body>
     <section class="content">@content</section>
   </body>
  </html>

从上可知，该模板接收两个参数：  ``title`` 和 ``HTML`` 内容块。定义好基础模板之后，就可以从其它模板中引用这个模板了。创建模板 ``views/Application/index.scala.html`` ：

.. code-block:: scala

  @main(title = "Home") {

    <h1>Home page</h1>

  }

也许我们还需要定义一个侧边栏。

.. code-block:: scala

  @(title: String)(sidebar: Html)(content: Html)
  <!DOCTYPE html>
  <html>
    <head>
      <title>@title</title>
    </head>
    <body>
      <section class="sidebar">@sidebar</section>
      <section class="content">@content</section>
    </body>
  </html>
  
这种情况下，继承模板这么写：

.. code-block:: scala
  @main("Home") {
    <h1>Sidebar</h1>
  
  } {
    <h1>Home page</h1>

  }

也可以分开来写：

.. code-block:: scala
  
  @sidebar = {
    <h1>Sidebar</h1>
  }

  @main("Home")(sidebar) {
    <h1>Home page</h1>

  }


