第二十三章 日志
==============

Play框架提供了一个日志接口。这个日志接口包括如下几个部分：

- Logger  

- Log levels

- Appenders

Logger
-------

在应用中创建Logger的实例来发送日志信息。每个 ``Logger`` 实例都有一个名字，以便区分。

Logger以名字中的点号区分继承关系。例如，以 ``'com.foo'`` 命名的 ``Logger`` 是 ``'com.foo.bar.Baz'`` 的父 ``Logger`` 。所有的 ``logger``都继承自一个根 ``logger`` ，通过 ``logger`` 继承可以配置一套的 ``logger``。

Play应用提供了一个默认的名字为 ``application`` 的 ``Logger`` 。

日志级别
--------

日志级别用于区分不同类型的日志信息。

Play中日志级别如下：

- OFF  禁用日志信息

- ERROR 运行错误

- WARN 警告信息

- INFO 日志信息

- DEBUG 调试信息

- TRACE 详细信息

输出源
-------
``Appenders`` 用于定义日志的输出源，日志API支持输出日志信息到不同的地方，通过配置可以实现输出日志到命令行，数据库等等。

使用日志
--------

首先导入 ``Logger`` 类及其伴生对象

.. code-block:: scala

  import play.api.Logger


默认Logger
----------

``Logger`` 对象是默认的 ``Logger`` ，它的名字是 ``application`` 。

.. code-block:: scala
  // Log some debug info
  Logger.debug("Attempting risky calculation.")

  try {
   val result = riskyCalculation
  
   // Log result if successful
    Logger.debug(s"Result=$result")
  } catch {
    case t: Throwable => {
     // Log error with message and Throwable.
     Logger.error("Exception with riskyCalculation", t)
   }
  }
  
自定义Logger
------------

通过 ``Logger`` 的 ``apply`` 方法可以创建一个 ``Logger``

.. code-block:: scala
  
  val accessLogger: Logger = Logger("access")

不过更普遍的方法针对每个类使用该类的类名作为 ``Logger`` 的名字：

.. code-block:: scala
  
  val logger: Logger = Logger(this.getClass())


日志配置
--------

Play框架使用 ``SLF4J`` 进行日志记录，背后使用 ``Logback`` 作为日支引擎。
