Pimple
======

Pimple is a small Dependency Injection Container for PHP 5.3 that consists
of just one file and one class (about 80 lines of code).

Pimple 是针对 PHP 5.3 的一个小巧的依赖注入容器，仅仅由一个文件和一个类（大约 80 行代码）组成。

`Download it`_, require it in your code, and you're good to go:

`下载它 <https://github.com/fabpot/Pimple>`_，在你的代码中 require 它，你就准备好了：

.. code-block:: php

    require_once '/path/to/Pimple.php';

Creating a container is a matter of instating the ``Pimple`` class:

创建一个容器就是一个实例化 ``Pimple`` 类的问题：

.. code-block:: php

    $container = new Pimple();

As many other dependency injection containers, Pimple is able to manage two
different kind of data: *services* and *parameters*.

作为许多其他依赖注入的容器，Pimple 能够管理两种不同类型的数据： *services* 和 *parameters*。

Defining Parameters // 定义 Parameters
----------------------------------------------

Defining a parameter is as simple as using the Pimple instance as an array:

定义一个 parameter 如同把 Pimple 实例当作一个数组一样简单：

.. code-block:: php

    // define some parameters // 定义一些 parameters
    $container['cookie_name'] = 'SESSION_ID';
    $container['session_storage_class'] = 'SessionStorage';

Defining Services // 定义 Services
------------------------------------------

A service is an object that does something as part of a larger system.
Examples of services: Database connection, templating engine, mailer. Almost
any object could be a service.

一个 service 是一个执行某些事的对象，作为一个较大系统的一部分。services 的例子：数据库连接、模版引擎或邮件程序。
几乎任何 **全局（global）** 对象都可以是一个 service。

Services are defined by anonymous functions that return an instance of an
object:

Services 通过匿名函数返回一个对象的实例来定义：

.. code-block:: php

    // define some services // 定义一些 services
    $container['session_storage'] = function ($c) {
        return new $c['session_storage_class']($c['cookie_name']);
    };

    $container['session'] = function ($c) {
        return new Session($c['session_storage']);
    };

Notice that the anonymous function has access to the current container
instance, allowing references to other services or parameters.

注意，匿名函数可以访问当前容器实例，并且允许引用其他 services 或 parameters。

As objects are only created when you get them, the order of the definitions
does not matter, and there is no performance penalty.

由于对象仅当你获取它们时才被创建，定义的顺序无关紧要，并且没有性能损失。

Using the defined services is also very easy:

使用已定义的 services 同样也时非常容易：

.. code-block:: php

    // get the session object // 获取 session 对象
    $session = $container['session'];

    // the above call is roughly equivalent to the following code:
    // 上面的调用大致等价于以下代码：
    // $storage = new SessionStorage('SESSION_ID');
    // $session = new Session($storage);

Defining Shared Services // 定义共享的 Services
-----------------------------------------------------

By default, each time you get a service, Pimple returns a new instance of it.
If you want the same instance to be returned for all calls, wrap your
anonymous function with the ``share()`` method:

默认，每次你获取一个 service，Pimple 返回一个新的此 service 实例 。如果你希望所有的调用都返回同一个实例，请使用
``share()`` 方法封装你的匿名函数：

.. code-block:: php

    $container['session'] = $container->share(function ($c) {
        return new Session($c['session_storage']);
    });

Protecting Parameters // 保护 Parameters
-----------------------------------------------

Because Pimple sees anonymous functions as service definitions, you need to
wrap anonymous functions with the ``protect()`` method to store them as
parameter:

因为 Pimple 将匿名函数视为 service 定义，你需要使用 ``protect()`` 方法封装匿名函数以作为
parameters 来存储它们：

.. code-block:: php

    $container['random'] = $container->protect(function () { return rand(); });

Modifying services after creation // 在创建后修改 Services
---------------------------------------------------------------

In some cases you may want to modify a service definition after it has been
defined. You can use the ``extend()`` method to define additional code to
be run on your service just after it is created:

有时候你可能希望在定义了 service 后修改它。你可以使用 ``extend()`` 方法来定义在你的 service
创建之后运行的附加代码：

.. code-block:: php

    $container['mail'] = function ($c) {
        return new \Zend_Mail();
    };

    $container['mail'] = $container->extend('mail', function($mail, $c) {
        $mail->setFrom($c['mail.default_from']);
        return $mail;
    });

The first argument is the name of the object, the second is a function that
gets access to the object instance and the container. The return value is
a service definition, so you need to re-assign it on the container.

第一个参数是 service 的名称，第二个参数是可获得访问此 service 对象实例和容器的函数。返回值是一个
service 定义，所以你需要在容器上重新指定（re-assign）它。

If the service you plan to extend is already shared, it's recommended that you
re-wrap your extended service with the ``shared`` method, otherwise your extension
code will be called every time you access the service:

如果计划扩展的服务已经共享了，推荐你使用 ``shared`` 方法重新封装你的已扩展的 service，否则，
你的扩展代码将在你每次访问此 service 时都被调用：

.. code-block:: php

    $container['twig'] = $container->share(function ($c) {
        return new Twig_Environment($c['twig.loader'], $c['twig.options']);
    });

    $container['twig'] = $container->share($container->extend('twig', function ($twig, $c) {
        $twig->addExtension(new MyTwigExtension());
        return $twig;
    }));

Fetching the service creation function // 取回 service 创建函数
--------------------------------------------------------------------

When you access an object, Pimple automatically calls the anonymous function
that you defined, which creates the service object for you. If you want to get
raw access to this function, you can use the ``raw()`` method:

当你访问一个对象时，Pimple 自动地调用你定义的匿名函数，以便为你创建 service 对象。如果你希望获得
此函数的原始（raw）访问，你可以使用 ``raw()`` 方法：

.. code-block:: php

    $container['session'] = $container->share(function ($c) {
        return new Session($c['session_storage']);
    });

    $sessionFunction = $container->raw('session');

Packaging a Container for reusability // 打包可复用性容器
-------------------------------------------------------------

If you use the same libraries over and over, you might want to create reusable
containers. Creating a reusable container is as simple as creating a class
that extends ``Pimple``, and configuring it in the constructor:

如果你反复使用同样的一些库，你可能想创建可复用的容器。创建一个可复用的容器如同创建一个 ``Pimple``
的扩展类一样简单，在构造器中配置它：

.. code-block:: php

    class SomeContainer extends Pimple
    {
        public function __construct()
        {
            $this['parameter'] = 'foo';
            $this['object'] = function () { return stdClass(); };
        }
    }

Using this container from your own is as easy as it can get:

使用你自己的容器同样容易，可以这样得到它：

    $container = new Pimple();

    // define your project parameters and services
    // 定义你的项目 parameters 和 services
    // ...

    // embed the SomeContainer container // 嵌入 SomeContainer 容器
    $container['embedded'] = $container->share(function () { return new SomeContainer(); });

    // configure it // 配置它
    $container['embedded']['parameter'] = 'bar';

    // use it // 使用它
    $container['embedded']['object']->...;

.. _Download it: https://github.com/fabpot/Pimple