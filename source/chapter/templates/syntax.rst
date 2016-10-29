模板语法
=======

概览
----

Play Scala模板是一个包含scala代码的文本文件，它可以生成各种文本格式的文件，例如HTML、XML、CSV等。这个模板框架让开发者很容易的进行前后端的开发。

每个模板都会根据规范被编译成标准的的Scala函数，如果模板文件为 ``views/Application/index.scala.html`` ，则编译成类 ``views.html.Application.index`` ， 这个类具有 ``apply()`` 方法。

例如，下面模板：

.. code-block:: scala

  @(customer: Customer, orders: List[Order])

  <h1>Welcome @customer.name!</h1>

  <ul>
  @for(order <- orders) {
    <li>@order.title</li>
  }
  </ul>

定义上述模板之后，从任意其他Scala代码中，我们可以调用下面的方法：

.. code-block:: scala
  
  val content = views.html.Application.index(c, o)

魔术字符@
--------

Scala模板有且只有一个魔法字符 ``@`` ，每当遇到这个字符的时候，就表明scala语句的开始，但是我们不需要像要其它模板语言一样，闭合scala代码段，Scala会自动判断代码的结束。

.. code-block:: scala

  Hello @customer.name!
         ^^^^^^^^^^^^^
         Dynamic code
         
如果想插入一条包含多个参数的scala代码，可以使用括号显示声明。

.. code-block:: scala
  Hello @(customer.firstName + customer.lastName)!
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                      Dynamic Code

还可以使用花括号插入多条scala声明语句：

.. code-block:: scala
  
   Hello @{val name = customer.firstName + customer.lastName; name}!
       ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                             Dynamic Code

由于 ``@`` 是特殊字符，所以有时候需要进行转义，使用 ``@@`` 进行转义

.. code-block:: scala
  
  My email is bob@@example.com

模板参数
-------

一个模板其实就像一个scala函数，所以它需要参数，这些参数必须在模板的开始处进行声明。

.. code-block:: scala
  
  @(customer: Customer, orders: List[Order])

还可以给参数设置默认值：

.. code-block:: scala
  
  @(title: String = "Home")

甚至可以传递参数组合：

.. code-block:: scala
  
  @(title: String)(body: Html)


循环
----

可以使用scala的 ``for`` 循环语句进行循环操作：

.. code-block:: scala

  <ul>
  @for(p <- products) {
    <li>@p.name ($@p.price)</li>
  }
  </ul>

注意，必须确保 ``{`` 必须与 ``for`` 位于同一行。

判断
----

模板中的判断语句与scala一样。

.. code-block:: scala
  
  @if(items.isEmpty) {
    <h1>Nothing to display</h1>
  } else {
    <h1>@items.size items!</h1>
  }

创建可复用代码块
--------------

相当于创建一个宏命令：

.. code-block:: scala
  
  @display(product: Product) = {
    @product.name ($@product.price)
  }

  <ul>
  @for(product <- products) {
    @display(product)
  }
  </ul>

也可以定义完全由scala组成的复用代码：

.. code-block:: scala
  
  @title(text: String) = @{
    text.split(' ').map(_.capitalize).mkString(" ")
  }

  <h1>@title("hello world")</h1>

惯例情况下，如果可复用代码块中名字前带有 ``implicit`` ，它就需要标注为 ``implicit`` 。

.. code-block:: scala
  
  @implicitFieldConstructor = @{ MyFieldConstructor() }

定义可复用的变量
---------------

使用 ``defining`` 定义可复用的变量。

.. code-block:: scala
  
  @defining(user.firstName + " " + user.lastName) { fullName =>
    <div>Hello @fullName</div>
  }


导入声明
--------

可以在模板的开头导入任何你想导入的包。

.. code-block:: scala
  
  @(customer: Customer, orders: List[Order])
  @import utils._

如果想使用绝对路径的话，在导入语句前面使用 **root**。

.. code-block:: scala
  
  @import _root_.company.product.core._
  
如果需要在所有模板中都导入同一个包，可以在 ``build.sbt`` 中进行声明。

.. code-block:: scala
  
  TwirlKeys.templateImports += "org.abc.backend._"

注释
-----

使用 ``@**@`` 进行注释:

.. code-block:: scala

  @*********************
    * This is a comment *
  *********************@

还可以在模板文件的开头注释，以保存到Scala的API文档中。

.. code-block:: scala
  
  @*************************************
  * Home page.                        *
  *                                   *
  * @param msg The message to display *
  *************************************@
  @(msg: String)

  <h1>@msg</h1>

转义
-----

默认情况下，动态内容根据模板类型解析成相应的格式文件，如果只想输出原始的内容片段，将它包含在模板内容类型中即可：

.. code-block:: scala

  <p>
    @Html(article.content)
  </p>
