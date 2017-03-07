Installation and Configuration // 安装与配置
=================================================

Doctrine can be installed with `Composer <http://www.getcomposer.org>`_.  For
older versions we still have `PEAR packages
<http://pear.doctrine-project.org>`_.

Doctrine 可以用 `Composer <http://www.getcomposer.org>`_ 安装。对于老版本，我们仍然有
`PEAR 包 <http://pear.doctrine-project.org>`_。

Define the following requirement in your ``composer.json`` file:

在你的 ``composer.json`` 中定义以下需要：

::

    {
        "require": {
            "doctrine/orm": "*"
        }
    }

Then call ``composer install`` from your command line. If you don't know
how Composer works, check out their `Getting Started
<http://getcomposer.org/doc/00-intro.md>`_ to set up.

然后从你的命令调用 ``composer install``。如果你不知道 Composer 如何运作，查看它的
`入门 <http://getcomposer.org/doc/00-intro.md>`_ 以配置。

Class loading // 类加载
----------------------------

Autoloading is taken care of by Composer. You just have to include the composer autoload file in your project:

自动加载由 Composer 负责。你只是需要包含 Composer 的自动加载文件在你的项目中：

.. code-block:: php

    <?php
    // bootstrap.php
    // Include Composer Autoload (relative to project root).
    require_once "vendor/autoload.php";

Obtaining an EntityManager // 获得 EntityManager
------------------------------------------------------

Once you have prepared the class loading, you acquire an
*EntityManager* instance. The EntityManager class is the primary
access point to ORM functionality provided by Doctrine.

一旦你已经准备好了类加载，你获得一个*EntityManager*实例。该 EntityManager 类
是由 Doctrine 提供的 ORM 功能的主要访问点。

.. code-block:: php

    <?php
    // bootstrap.php
    require_once "vendor/autoload.php";

    use Doctrine\ORM\Tools\Setup;
    use Doctrine\ORM\EntityManager;

    $paths = array("/path/to/entity-files");
    $isDevMode = false;

    // the connection configuration
    $dbParams = array(
        'driver'   => 'pdo_mysql',
        'user'     => 'root',
        'password' => '',
        'dbname'   => 'foo',
    );

    $config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
    $entityManager = EntityManager::create($dbParams, $config);

Or if you prefer XML:

或者如果你喜欢 XML：

.. code-block:: php

    <?php
    $paths = array("/path/to/xml-mappings");
    $config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
    $entityManager = EntityManager::create($dbParams, $config);

Or if you prefer YAML:

或者如果你喜欢 YAML：

.. code-block:: php

    <?php
    $paths = array("/path/to/yml-mappings");
    $config = Setup::createYAMLMetadataConfiguration($paths, $isDevMode);
    $entityManager = EntityManager::create($dbParams, $config);

.. note::
    If you want to use yml mapping you should add yaml dependency to your `composer.json`:

    如果你要使用 yml 映射你需要添加 yaml 依赖到你的 `composer.json`：
    
    ::
    
        "symfony/yaml": "*"

Inside the ``Setup`` methods several assumptions are made:

``Setup`` 方法内部的几个假设：

-  If `$isDevMode` is true caching is done in memory with the ``ArrayCache``. Proxy objects are recreated on every request.
-  如果 `$isDevMode` 为 true 使用 ``ArrayCache`` 在内存中完成缓存。代理对象在每次请求中被重新创建。
-  If `$isDevMode` is false, check for Caches in the order APC, Xcache, Memcache (127.0.0.1:11211), Redis (127.0.0.1:6379) unless `$cache` is passed as fourth argument.
-  如果 `$isDevMode` 为 false，按顺序 APC, Xcache, Memcache (127.0.0.1:11211), Redis (127.0.0.1:6379) 检查缓存，除非传递了第四个参数 $cache。
-  If `$isDevMode` is false, set then proxy classes have to be explicitly created through the command line.
-  如果 `$isDevMode` 为 false，那么代理类必须被明确地创建，通过命令行设置。
-  If third argument `$proxyDir` is not set, use the systems temporary directory.
-  如果第三个参数 `$proxyDir` 没有设置，使用系统的临时目录。

If you want to configure Doctrine in more detail, take a look at the :doc:`Advanced
Configuration <reference/advanced-configuration>` section.

如果你想要更加详细地配置 Doctrine，看一看 :doc:`高级配置 <reference/advanced-configuration>` 部分。

.. note::

    You can learn more about the database connection configuration in the
    `Doctrine DBAL connection configuration reference <http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html>`_.

    你可以在 `Doctrine DBAL 连接配置参考 <http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/configuration.html>`_
    中学习到更多关于数据库连接的配置。

Setting up the Commandline Tool // 设置命令行工具
-----------------------------------------------------

Doctrine ships with a number of command line tools that are very helpful
during development. You can call this command from the Composer binary
directory:

Doctrine 附带了一些命令行工具，在开发期间非常地有用。你可以从 Composer 二进制目录调用这些命令：

.. code-block:: sh

    $ php vendor/bin/doctrine

You need to register your applications EntityManager to the console tool
to make use of the tasks by creating a ``cli-config.php`` file with the
following content:

你需要注册你应用程序的 EntityManager 到控制台工具，通过创建一个带有如下内容的 ``cli-config.php`` 文件
来使用它们：

On Doctrine 2.4 and above:

在 Doctrine 2.4及以上版本上：

.. code-block:: php

    <?php
    use Doctrine\ORM\Tools\Console\ConsoleRunner;

    // replace with file to your own project bootstrap
    require_once 'bootstrap.php';

    // replace with mechanism to retrieve EntityManager in your app
    $entityManager = GetEntityManager();

    return ConsoleRunner::createHelperSet($entityManager);

On Doctrine 2.3 and below:

在 Doctrine 2.3 及以下版本上：

.. code-block:: php

    <?php
    // cli-config.php
    require_once 'my_bootstrap.php';

    // Any way to access the EntityManager from  your application
    $em = GetMyEntityManager();

    $helperSet = new \Symfony\Component\Console\Helper\HelperSet(array(
        'db' => new \Doctrine\DBAL\Tools\Console\Helper\ConnectionHelper($em->getConnection()),
        'em' => new \Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper($em)
    ));
