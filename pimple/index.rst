Pimple
======
.. caution::


    注意，翻译此文档时 Pimple 的版本为 3.0.2。之后版本的内容或许会有变更，最新的英文文档请移步
    `Pimple 的官方 github 页面 <https://github.com/silexphp/Pimple>`_。

.. caution::

    This is the documentation for Pimple 3.x. If you are using Pimple 1.x, read
    the `Pimple 1.x documentation`_. Reading the Pimple 1.x code is also a good
    way to learn more about how to create a simple Dependency Injection
    Container (recent versions of Pimple are more focused on performance).

    本文档针对的是 Pimple 3.x。如果你使用 Pimple 1.x，请查看 :doc:`Pimple 1.x 文档 <pimple-1.x>`。阅读 Pimple 1.x
    的代码也是一种学习如何创建一个简单的依赖注入容器的好方式（最新版本的 Pimple 更多聚焦在性能上面）。

Pimple is a small Dependency Injection Container for PHP.

Pimple 是一个小巧的 PHP 依赖注入容器。

Installation // 安装
---------------------------

Before using Pimple in your project, add it to your ``composer.json`` file:

在你的项目中使用 Pimple 之前，请将它添加到你的 ``composer.json`` 文件：

.. code-block:: bash

    $ ./composer.phar require pimple/pimple "~3.0"

Alternatively, Pimple is also available as a PHP C extension:

或者，Pimple 也有可用的 PHP C 扩展：

.. code-block:: bash

    $ git clone https://github.com/silexphp/Pimple
    $ cd Pimple/ext/pimple
    $ phpize
    $ ./configure
    $ make
    $ make install

Usage // 使用
------------------

Creating a container is a matter of creating a ``Container`` instance:

创建一个容器就是创建一个 ``Container`` 实例的问题：

.. code-block:: php

    use Pimple\Container;

    $container = new Container();

As many other dependency injection containers, Pimple manages two different
kind of data: **services** and **parameters**.

作为许多其他依赖注入的容器，Pimple 管理着两种不同类型的数据： **services** 和 **parameters**。

Defining Services // 定义 Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A service is an object that does something as part of a larger system. Examples
of services: a database connection, a templating engine, or a mailer. Almost
any **global** object can be a service.

一个 service 是一个执行某些事的对象，作为一个较大系统的一部分。services 的例子：数据库连接、模版引擎或邮件程序。
几乎任何 **全局（global）** 对象都可以是一个 service。

Services are defined by **anonymous functions** that return an instance of an
object:

Services 通过 **匿名函数** 返回一个对象的实例来定义：

.. code-block:: php

    // define some services // 定义一些 services
    $container['session_storage'] = function ($c) {
        return new SessionStorage('SESSION_ID');
    };

    $container['session'] = function ($c) {
        return new Session($c['session_storage']);
    };

Notice that the anonymous function has access to the current container
instance, allowing references to other services or parameters.

注意，匿名函数可以访问当前容器实例，并且允许引用其他 services 或 parameters。

As objects are only created when you get them, the order of the definitions
does not matter.

由于对象仅当你获取它们时才被创建，定义的顺序无关紧要。

Using the defined services is also very easy:

使用已定义的 services 同样也时非常容易：

.. code-block:: php

    // get the session object // 获取 session 对象
    $session = $container['session'];

    // the above call is roughly equivalent to the following code:
    // 上面的调用大致等价于以下代码：
    // $storage = new SessionStorage('SESSION_ID');
    // $session = new Session($storage);

Defining Factory Services // 定义工厂 Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, each time you get a service, Pimple returns the **same instance**
of it. If you want a different instance to be returned for all calls, wrap your
anonymous function with the ``factory()`` method

默认，每次你获取一个 service，Pimple 返回此 service 的 **同一个实例** 。如果你希望所有的调用都返回不同的实例，请使用
``factory()`` 方法封装你的匿名函数：

.. code-block:: php

    $container['session'] = $container->factory(function ($c) {
        return new Session($c['session_storage']);
    });

Now, each call to ``$container['session']`` returns a new instance of the
session.

现在，每次调用 ``$container['session']`` 返回一个新的 session 的实例。

Defining Parameters // 定义 Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Defining a parameter allows to ease the configuration of your container from
the outside and to store global values:

定义一个 parameter 允许从外部简化你的容器的配置并存储全局值：

.. code-block:: php

    // define some parameters // 定义一些 parameters
    $container['cookie_name'] = 'SESSION_ID';
    $container['session_storage_class'] = 'SessionStorage';

If you change the ``session_storage`` service definition like below:

如果你改变 ``session_storage`` service 的定义像如下这样：

.. code-block:: php

    $container['session_storage'] = function ($c) {
        return new $c['session_storage_class']($c['cookie_name']);
    };

You can now easily change the cookie name by overriding the
``session_storage_class`` parameter instead of redefining the service
definition.

你现在可以简单地通过重新定义该 service 定义覆盖 ``session_storage_class`` parameter 来改变 cookie 的名称。

Protecting Parameters // 保护 Parameters
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Because Pimple sees anonymous functions as service definitions, you need to
wrap anonymous functions with the ``protect()`` method to store them as
parameters:

因为 Pimple 将匿名函数视为 service 定义，你需要使用 ``protect()`` 方法封装匿名函数以作为
parameters 来存储它们：

.. code-block:: php

    $container['random_func'] = $container->protect(function () {
        return rand();
    });

Modifying Services after Definition // 在定义后修改 Services
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some cases you may want to modify a service definition after it has been
defined. You can use the ``extend()`` method to define additional code to be
run on your service just after it is created:

有时候你可能希望在定义了 service 后修改它。你可以使用 ``extend()`` 方法来定义在你的 service
创建之后运行的附加代码：

.. code-block:: php

    $container['session_storage'] = function ($c) {
        return new $c['session_storage_class']($c['cookie_name']);
    };

    $container->extend('session_storage', function ($storage, $c) {
        $storage->...();

        return $storage;
    });

The first argument is the name of the service to extend, the second a function
that gets access to the object instance and the container.

第一个参数是将要扩展的 service 的名称，第二个参数是可获得访问此 service 对象实例和容器的函数。

Extending a Container // 扩展容器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you use the same libraries over and over, you might want to reuse some
services from one project to the next one; package your services into a
**provider** by implementing ``Pimple\ServiceProviderInterface``:

如果你反复使用同样的一些库，你可能想从一个项目到下一个项目复用某些 services；你可以通过实现
``Pimple\ServiceProviderInterface`` 接口打包你的 services 到一个 **provider** 中：

.. code-block:: php

    use Pimple\Container;

    class FooProvider implements Pimple\ServiceProviderInterface
    {
        public function register(Container $pimple)
        {
            // register some services and parameters
            // on $pimple
            // 在 $pimple 上注册一些 services 和 parameters
        }
    }

Then, register the provider on a Container:

然后，在容器上注释此 provider：

.. code-block:: php

    $pimple->register(new FooProvider());

Fetching the Service Creation Function // 取回 Service 创建函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you access an object, Pimple automatically calls the anonymous function
that you defined, which creates the service object for you. If you want to get
raw access to this function, you can use the ``raw()`` method:

当你访问一个对象时，Pimple 自动地调用你定义的匿名函数，以便为你创建 service 对象。如果你希望获得
此函数的原始（raw）访问，你可以使用 ``raw()`` 方法：

.. code-block:: php

    $container['session'] = function ($c) {
        return new Session($c['session_storage']);
    };

    $sessionFunction = $container->raw('session');

.. _Pimple 1.x documentation: https://github.com/silexphp/Pimple/tree/1.1