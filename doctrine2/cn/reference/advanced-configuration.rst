Advanced Configuration // 高级配置
=======================================

The configuration of the EntityManager requires a
``Doctrine\ORM\Configuration`` instance as well as some database
connection parameters. This example shows all the potential
steps of configuration.

EntityManager 的配置需要一个 ``Doctrine\ORM\Configuration`` 实例以及一些数据库的参数。
此例展示所有潜在的配置步骤。

.. code-block:: php

    <?php
    use Doctrine\ORM\EntityManager,
        Doctrine\ORM\Configuration;
    
    // ...
    
    if ($applicationMode == "development") {
        $cache = new \Doctrine\Common\Cache\ArrayCache;
    } else {
        $cache = new \Doctrine\Common\Cache\ApcCache;
    }
    
    $config = new Configuration;
    $config->setMetadataCacheImpl($cache);
    $driverImpl = $config->newDefaultAnnotationDriver('/path/to/lib/MyProject/Entities');
    $config->setMetadataDriverImpl($driverImpl);
    $config->setQueryCacheImpl($cache);
    $config->setProxyDir('/path/to/myproject/lib/MyProject/Proxies');
    $config->setProxyNamespace('MyProject\Proxies');
    
    if ($applicationMode == "development") {
        $config->setAutoGenerateProxyClasses(true);
    } else {
        $config->setAutoGenerateProxyClasses(false);
    }
    
    $connectionOptions = array(
        'driver' => 'pdo_sqlite',
        'path' => 'database.sqlite'
    );
    
    $em = EntityManager::create($connectionOptions, $config);

.. note::

    Do not use Doctrine without a metadata and query cache!
    Doctrine is optimized for working with caches. The main
    parts in Doctrine that are optimized for caching are the metadata
    mapping information with the metadata cache and the DQL to SQL
    conversions with the query cache. These 2 caches require only an
    absolute minimum of memory yet they heavily improve the runtime
    performance of Doctrine. The recommended cache driver to use with
    Doctrine is `APC <http://www.php.net/apc>`_. APC provides you with
    an opcode-cache (which is highly recommended anyway) and a very
    fast in-memory cache storage that you can use for the metadata and
    query caches as seen in the previous code snippet.

    不要不使用元数据和查询缓存的情况下使用 Doctrine！Doctrine 被优化用于与缓存一起
    工作。Doctrine 中主要为缓存而优化的部分是使用元数据缓存的元数据映射信息和使用查询
    缓存的 DQL 到 SQL 的转换。这2个缓存仅需要绝对最小的内存然而它们大大地增强了 Doctrine
    的运行时性能。被推荐使用于 Doctrine 的缓存驱动是 `APC <http://www.php.net/apc>`_。
    APC 提供你使用 opcode-cache（强烈推荐的）和一个非常快的你可以用于元数据和查询缓存的
    内存缓存存储，正如在前面的代码片断看到的。

Configuration Options // 配置选项
--------------------------------------

The following sections describe all the configuration options
available on a ``Doctrine\ORM\Configuration`` instance.

以下部分描述所有在 ``Doctrine\ORM\Configuration`` 实例中可用的配置选项。

Proxy Directory (***REQUIRED***) // 代理目录（***必须的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~-------------------------------

.. code-block:: php

    <?php
    $config->setProxyDir($dir);
    $config->getProxyDir();

Gets or sets the directory where Doctrine generates any proxy
classes. For a detailed explanation on proxy classes and how they
are used in Doctrine, refer to the "Proxy Objects" section further
down.

获取或设置 Doctrine 生成任何代理类的目录。对于详细的解释使用代理类和它们如何被用在
Doctrine 中，参考下面的“代理对象”部分。

Proxy Namespace (***REQUIRED***) // 代理命名空间（***必须的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~----------------------------------

.. code-block:: php

    <?php
    $config->setProxyNamespace($namespace);
    $config->getProxyNamespace();

Gets or sets the namespace to use for generated proxy classes. For
a detailed explanation on proxy classes and how they are used in
Doctrine, refer to the "Proxy Objects" section further down.

获得或设置用于生成的代理类的命名空间。对于详细的解释使用代理类和它们如何被用在
Doctrine 中，参考下面的“代理对象”部分。

Metadata Driver (***REQUIRED***) // 元数据驱动（***必须的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    $config->setMetadataDriverImpl($driver);
    $config->getMetadataDriverImpl();

Gets or sets the metadata driver implementation that is used by
Doctrine to acquire the object-relational metadata for your
classes.

获取或设置 Doctrine 用于为你获得对象关联元数据的元数据驱动实现。

There are currently 4 available implementations:

当前有4个可用的实现：

-  ``Doctrine\ORM\Mapping\Driver\AnnotationDriver``
-  ``Doctrine\ORM\Mapping\Driver\XmlDriver``
-  ``Doctrine\ORM\Mapping\Driver\YamlDriver``
-  ``Doctrine\ORM\Mapping\Driver\DriverChain``

Throughout the most part of this manual the AnnotationDriver is
used in the examples. For information on the usage of the XmlDriver
or YamlDriver please refer to the dedicated chapters
``XML Mapping`` and ``YAML Mapping``.

贯穿本手册的大部分示例中使用的是 AnnotationDriver。对于 XmlDriver 或 YamlDriver
的用法的信息，请参考专门的 ``XML 映射`` 和 ``YAML 映射`` 章节。

The annotation driver can be configured with a factory method on
the ``Doctrine\ORM\Configuration``:

注释驱动可以使用在 ``Doctrine\ORM\Configuration`` 上的一个工厂方法来配置：

.. code-block:: php

    <?php
    $driverImpl = $config->newDefaultAnnotationDriver('/path/to/lib/MyProject/Entities');
    $config->setMetadataDriverImpl($driverImpl);

The path information to the entities is required for the annotation
driver, because otherwise mass-operations on all entities through
the console could not work correctly. All of metadata drivers
accept either a single directory as a string or an array of
directories. With this feature a single driver can support multiple
directories of Entities.

对于注释驱动到实体的路径信息是必须的，因为不然通过控制台对所有实体的大量操作（mass-operations）
不能正常地工作。所有的元数据驱动接受单个目录作为字符串或目录的数组。使用此功能
单个驱动可以支持实体的多个目录。

Metadata Cache (***RECOMMENDED***) // 元数据缓存（***推荐的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    $config->setMetadataCacheImpl($cache);
    $config->getMetadataCacheImpl();

Gets or sets the cache implementation to use for caching metadata
information, that is, all the information you supply via
annotations, xml or yaml, so that they do not need to be parsed and
loaded from scratch on every single request which is a waste of
resources. The cache implementation must implement the
``Doctrine\Common\Cache\Cache`` interface.

获得或设置缓存实现以用于缓存元数据信息，那是通过你提供的注释、xml 或 yaml 的所有信息，
所以它们不需要在每次单个请求都解析和从头开始加载，那是资源的浪费。缓存实现必须实现
``Doctrine\Common\Cache\Cache`` 接口。

Usage of a metadata cache is highly recommended.

强烈建议使用元数据缓存。

The recommended implementations for production are:

为生产推荐的实现是：

-  ``Doctrine\Common\Cache\ApcCache``
-  ``Doctrine\Common\Cache\MemcacheCache``
-  ``Doctrine\Common\Cache\XcacheCache``
-  ``Doctrine\Common\Cache\RedisCache``

For development you should use the
``Doctrine\Common\Cache\ArrayCache`` which only caches data on a
per-request basis.

对于开发你应该使用 ``Doctrine\Common\Cache\ArrayCache``，它仅在每个请求的基础上缓存数据。

Query Cache (***RECOMMENDED***) // 查询缓存（***推荐的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    $config->setQueryCacheImpl($cache);
    $config->getQueryCacheImpl();

Gets or sets the cache implementation to use for caching DQL
queries, that is, the result of a DQL parsing process that includes
the final SQL as well as meta information about how to process the
SQL result set of a query. Note that the query cache does not
affect query results. You do not get stale data. This is a pure
optimization cache without any negative side-effects (except some
minimal memory usage in your cache).

获取或设置缓存实现以用于缓存 DQL 查询，那是 DQL 解析过程的结果。它包含最终 SQL
以及有关如何处理查询的 SQL 结果集的元（meta）信息。注意，查询缓存不会影响查询结果。
你不能获得陈旧的数据。这是一个纯最优化缓存没有任何负面的影响（除了在你缓存中的一些
最小的内存使用）

Usage of a query cache is highly recommended.

强烈地推荐使用查询缓存。

The recommended implementations for production are:

为生产推荐的实现是：

-  ``Doctrine\Common\Cache\ApcCache``
-  ``Doctrine\Common\Cache\MemcacheCache``
-  ``Doctrine\Common\Cache\XcacheCache``
-  ``Doctrine\Common\Cache\RedisCache``

For development you should use the
``Doctrine\Common\Cache\ArrayCache`` which only caches data on a
per-request basis.

对于开发你应该使用 ``Doctrine\Common\Cache\ArrayCache``，它仅在每个请求的基础上缓存数据。

SQL Logger (***Optional***) // SQL 日志器（***可选的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    <?php
    $config->setSQLLogger($logger);
    $config->getSQLLogger();

Gets or sets the logger to use for logging all SQL statements
executed by Doctrine. The logger class must implement the
``Doctrine\DBAL\Logging\SQLLogger`` interface. A simple default
implementation that logs to the standard output using ``echo`` and
``var_dump`` can be found at
``Doctrine\DBAL\Logging\EchoSQLLogger``.

获取或设置日志器（logger）以用于 logging 所有由 Doctrine 执行的 SQL 语句。
日志器类必须实现 ``Doctrine\DBAL\Logging\SQLLogger`` 接口。一个简单的默认实现
可以在 ``Doctrine\DBAL\Logging\EchoSQLLogger`` 找到，它使用 ``echo`` 和
``var_dump`` 来 logs 到标准输出。

Auto-generating Proxy Classes (***OPTIONAL***) // 自动生成代理类（***可选的***）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Proxy classes can either be generated manually through the Doctrine
Console or automatically at runtime by Doctrine. The configuration
option that controls this behavior is:

代理类可以通过 Doctrine 控制台手动生成或由 Doctrine 在运行时自动地生成。控制
此行为的配置选项是：

.. code-block:: php

    <?php
    $config->setAutoGenerateProxyClasses($mode);

Possible values for ``$mode`` are:

``$mode`` 可能的值是：

-  ``Doctrine\Common\Proxy\AbstractProxyFactory::AUTOGENERATE_NEVER``

Never autogenerate a proxy. You will need to generate the proxies
manually, for this use the Doctrine Console like so:

永远不要自动生成代理。你将需要手动地生成代理，为此使用 Doctrine 控制台类似这样：

.. code-block:: php

    $ ./doctrine orm:generate-proxies

When you do this in a development environment,
be aware that you may get class/file not found errors if certain proxies
are not yet generated. You may also get failing lazy-loads if new
methods were added to the entity class that are not yet in the proxy class.
In such a case, simply use the Doctrine Console to (re)generate the
proxy classes.

当你在生成环境中这样做了，请注意，如果某些代理没有被生成你可能得到类/文件找不到的错误。如果新的方法
已被添加到尚未在代理类中的实体类，你也可能得到懒加载失败。在这种情况下，简单地使用 Doctrine 控制台
以重新生成代理类。

-  ``Doctrine\Common\Proxy\AbstractProxyFactory::AUTOGENERATE_ALWAYS``

Always generates a new proxy in every request and writes it to disk.

总是在每一个请求中生成一个新代理并写入到磁盘。

-  ``Doctrine\Common\Proxy\AbstractProxyFactory::AUTOGENERATE_FILE_NOT_EXISTS``

Generate the proxy class when the proxy file does not exist.
This strategy causes a file exists call whenever any proxy is
used the first time in a request.

当代理文件不存在时生成代理类。每当任何代理在请求中第一次被使用时，此策略将导致一次文件存在调用。

-  ``Doctrine\Common\Proxy\AbstractProxyFactory::AUTOGENERATE_EVAL``

Generate the proxy classes and evaluate them on the fly via eval(),
avoiding writing the proxies to disk.
This strategy is only sane for development.

生成代理类并通过 eval() 在运行中评估它们，以避免写入代理到磁盘。此策略仅对于开发是明智的。

In a production environment, it is highly recommended to use
AUTOGENERATE_NEVER to allow for optimal performances. The other
options are interesting in development environment.

在生成环境中，强烈推荐使用 AUTOGENERATE_NEVER 以允许优化性能。其他的选项在开发环境中是
有趣的。

Before v2.4, ``setAutoGenerateProxyClasses`` would accept a boolean
value. This is still possible, ``FALSE`` being equivalent to
AUTOGENERATE_NEVER and ``TRUE`` to AUTOGENERATE_ALWAYS.

在v2.4以前，``setAutoGenerateProxyClasses`` 将接受一个布尔值。这仍然是可能的，``FALSE``
等价于 AUTOGENERATE_NEVER 且 ``TRUE`` 等价于 AUTOGENERATE_ALWAYS。

Development vs Production Configuration // 开发 vs 生产配置
---------------------------------------------------------------

You should code your Doctrine2 bootstrapping with two different
runtime models in mind. There are some serious benefits of using
APC or Memcache in production. In development however this will
frequently give you fatal errors, when you change your entities and
the cache still keeps the outdated metadata. That is why we
recommend the ``ArrayCache`` for development.

记住，你应该编写你的 Doctrine2 引导使用两种不同的运行时模式。在生成中使用 APC 或 Memcache
有一些重要的优势。然而，在开发中这将经常给你致命的错误，当你更改你的实体且缓存仍然保留过时的
元数据时。这就是为何我们推荐 ``ArrayCache`` 用于开发。

Furthermore you should have the Auto-generating Proxy Classes
option to true in development and to false in production. If this
option is set to ``TRUE`` it can seriously hurt your script
performance if several proxy classes are re-generated during script
execution. Filesystem calls of that magnitude can even slower than
all the database queries Doctrine issues. Additionally writing a
proxy sets an exclusive file lock which can cause serious
performance bottlenecks in systems with regular concurrent
requests.

此外，你应该在开发中让自动生成代理类选项为 true 且在生成中为 false。如果此选项设置为 ``TRUE``
它可以严重损耗你的脚本性能，如果某些代理类在脚本执行期间重新生成的话。巨大的文件系统调用甚至可以比
Doctrine 发出的所有数据库查询还要慢。另外，编写代理设置为独占的文件锁，在具有常规并发请求的系统中
可能导致严重的性能瓶颈。

Connection Options // 连接配置
-----------------------------------

The ``$connectionOptions`` passed as the first argument to
``EntityManager::create()`` has to be either an array or an
instance of ``Doctrine\DBAL\Connection``. If an array is passed it
is directly passed along to the DBAL Factory
``Doctrine\DBAL\DriverManager::getConnection()``. The DBAL
configuration is explained in the
`DBAL section <./../../../../../projects/doctrine-dbal/en/latest/reference/configuration.html>`_.

``$connectionOptions`` 作为第一个参数传递到 ``EntityManager::create()``，它必须是一个数组或
一个 ``Doctrine\DBAL\Connection`` 的实例。如果传递的是一个数组，它直接地顺着传递到 DBAL 工厂
``Doctrine\DBAL\DriverManager::getConnection()``。DBAL 的配置在 `DBAL 部分 <./../../../../../projects/doctrine-dbal/en/latest/reference/configuration.html>`_ 中解释。

Proxy Objects // 代理对象
------------------------------

A proxy object is an object that is put in place or used instead of
the "real" object. A proxy object can add behavior to the object
being proxied without that object being aware of it. In Doctrine 2,
proxy objects are used to realize several features but mainly for
transparent lazy-loading.

代理对象是一个安装或使用的替代“真实”对象的对象。代理对象可以添加行为到正在被代理的对象而
无需察觉到它。在 Doctrine 2中，代理对象被用于实现几个功能，但是主要地用于透明的懒加载。

Proxy objects with their lazy-loading facilities help to keep the
subset of objects that are already in memory connected to the rest
of the objects. This is an essential property as without it there
would always be fragile partial objects at the outer edges of your
object graph.

代理对象使用其懒加载设施帮助保持已经在内存中的对象的子集连接到剩余的对象。这是一个必要的属性
因为没有它在对象图的外边缘将总是有脆弱的部分对象。

Doctrine 2 implements a variant of the proxy pattern where it
generates classes that extend your entity classes and adds
lazy-loading capabilities to them. Doctrine can then give you an
instance of such a proxy class whenever you request an object of
the class being proxied. This happens in two situations:

Doctrine 2 实现了一个代理模式的变体，它生成扩展你的实体类的类并向它们添加懒加载功能。
之后，每当你请求被代理类的对象时，Doctrine 可以给你一个这样的代理类实例。这发生在两种
情况中：

Reference Proxies // 引用代理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The method ``EntityManager#getReference($entityName, $identifier)``
lets you obtain a reference to an entity for which the identifier
is known, without loading that entity from the database. This is
useful, for example, as a performance enhancement, when you want to
establish an association to an entity for which you have the
identifier. You could simply do this:

``EntityManager#getReference($entityName, $identifier)`` 方法让你获得一个
对于那个已知的标识符的实体的引用，而不从数据库中加载那个实体。这是很有用的，例如，当你希望
建立一个到那个你拥有的标识符的实体的关联时，作为性能的增强。你可以简单地这样做：

.. code-block:: php

    <?php
    // $em instanceof EntityManager, $cart instanceof MyProject\Model\Cart
    // $itemId comes from somewhere, probably a request parameter
    $item = $em->getReference('MyProject\Model\Item', $itemId);
    $cart->addItem($item);

Here, we added an Item to a Cart without loading the Item from the
database. If you invoke any method on the Item instance, it would
fully initialize its state transparently from the database. Here
$item is actually an instance of the proxy class that was generated
for the Item class but your code does not need to care. In fact it
**should not care**. Proxy objects should be transparent to your
code.

这里，我们添加了一个 Item 到 Cart 而没有从数据库加载该 Item。如果你在
Item 实例上调用任何方法，它将从数据库中透明地完整地实例它的状态。这里 $item 事实上是一个
代理类的实例，它被生成用于该 Item 类，当你的代码不需要关心它。在实际中它**不应该关心**。
代理对象应该对你的代码透明。

Association proxies // 关联代理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The second most important situation where Doctrine uses proxy
objects is when querying for objects. Whenever you query for an
object that has a single-valued association to another object that
is configured LAZY, without joining that association in the same
query, Doctrine puts proxy objects in place where normally the
associated object would be. Just like other proxies it will
transparently initialize itself on first access.

第二个最重要的情况，Doctrine 使用代理对象是当查询对象时。每当你查询一个对象时，该对象
拥有被配置为懒的另一个对象的单值关联，在同样的查询中没有联结其他关联，Doctrine 会将代理对象
放在通常关联对象所在的位置。就像另一个代理，在第一次访问时它将透明地初始化自身。

.. note::

    Joining an association in a DQL or native query
    essentially means eager loading of that association in that query.
    This will override the 'fetch' option specified in the mapping for
    that association, but only for that query.

    在 DQL 或 原生查询中，联结一个关联本质上意味着在该查询中急切的加载该关联。这将覆盖在
    该关联的映射中定义的 'fetch' 选项，但仅对于该查询。

Generating Proxy classes // 生成代理类
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In a production environment, it is highly recommended to use
``AUTOGENERATE_NEVER`` to allow for optimal performances.
However you will be required to generate the proxies manually
using the Doctrine Console:

在一个生产环境中，强烈推荐使用 ``AUTOGENERATE_NEVER`` 以允许优化性能。但是，你将必须手动
使用 Doctrine 控制台生成代理：

.. code-block:: php

    $ ./doctrine orm:generate-proxies

The other options are interesting in development environment:

另一些在开发环境中的有趣选项：

- ``AUTOGENERATE_ALWAYS`` will require you to create and configure
  a proxy directory. Proxies will be generated and written to file
  on each request, so any modification to your code will be acknowledged.
- ``AUTOGENERATE_ALWAYS`` 将需要你创建和配置一个代理目录。代理将在每次请求上生成并写入文件。
  所以任何对你代码的修改将被感知。
- ``AUTOGENERATE_FILE_NOT_EXISTS`` will not overwrite an existing
  proxy file. If your code changes, you will need to regenerate the
  proxies manually.
- ``AUTOGENERATE_FILE_NOT_EXISTS`` 将不会覆盖现有代理文件。如果你的代码变更了，你将需要手动
  生成代理。
- ``AUTOGENERATE_EVAL`` will regenerate each proxy on each request,
  but without writing them to disk.
- ``AUTOGENERATE_EVAL``  将在每次请求上生成每个代理，但是不写入它们到磁盘。

Autoloading Proxies // 自动加载代理
---------------------------------------

When you deserialize proxy objects from the session or any other storage
it is necessary to have an autoloading mechanism in place for these classes.
For implementation reasons Proxy class names are not PSR-0 compliant. This
means that you have to register a special autoloader for these classes:

当你从会话或任何其他存储反序列化（deserialize）代理对象时，有必要为这些类适当的拥有一种自动加载机制。
出于实现的原因，代理类名称不遵循 PSR-0。这意味着你必须为这些类注册一个特定的自动加载器：

.. code-block:: php

    <?php
    use Doctrine\Common\Proxy\Autoloader;

    $proxyDir = "/path/to/proxies";
    $proxyNamespace = "MyProxies";

    Autoloader::register($proxyDir, $proxyNamespace);

If you want to execute additional logic to intercept the proxy file not found
state you can pass a closure as the third argument. It will be called with
the arguments proxydir, namespace and className when the proxy file could not
be found.

如钩你希望执行额外的逻辑以拦截代理文件找不到的状态，你可以传递一个闭包（closure）作为第三个参数。
当代理文件找不到时它将带代理目录、命名空间、类名调用。

Multiple Metadata Sources // 多元数据源
--------------------------------------------

When using different components using Doctrine 2 you may end up
with them using two different metadata drivers, for example XML and
YAML. You can use the DriverChain Metadata implementations to
aggregate these drivers based on namespaces:

当使用 Doctrine 2 应用不同组件时，你可能最终会使用两个不同的元数据驱动程序，例如 XML 和 YAML。
你可以使用驱动链（DriverChain）元数据实现以基于命名空间来聚合这些驱动：

.. code-block:: php

    <?php
    use Doctrine\ORM\Mapping\Driver\DriverChain;

    $chain = new DriverChain();
    $chain->addDriver($xmlDriver, 'Doctrine\Tests\Models\Company');
    $chain->addDriver($yamlDriver, 'Doctrine\Tests\ORM\Mapping');

Based on the namespace of the entity the loading of entities is
delegated to the appropriate driver. The chain semantics come from
the fact that the driver loops through all namespaces and matches
the entity class name against the namespace using a
``strpos() === 0`` call. This means you need to order the drivers
correctly if sub-namespaces use different metadata driver
implementations.

基于实体的命名空间，实体的加载被委派给恰当的驱动。链的语义实际来源于驱动循环遍历所有
命名空间并针对该命名空间使用一个 ``strpos() === 0`` 调用匹配实体类名。这意味着，
如果子命名空间使用不同的元数据驱动实现，你需要正确地排序驱动。

Default Repository (***OPTIONAL***) // 默认仓库（***可选的***）
------------------------------------------------------------------

Specifies the FQCN of a subclass of the EntityRepository.
That will be available for all entities without a custom repository class.

指定 FQCN（完全限定类名）的 EntityRepository 的子类。

.. code-block:: php

    <?php
    $config->setDefaultRepositoryClassName($fqcn);
    $config->getDefaultRepositoryClassName();

The default value is ``Doctrine\ORM\EntityRepository``.
Any repository class must be a subclass of EntityRepository otherwise you got an ORMException

默认值是 ``Doctrine\ORM\EntityRepository``。任何仓库类必须是 EntityRepository
的子类，否则你将得到一个 ORMException 异常。

Setting up the Console // 设置控制台
-----------------------------------------

Doctrine uses the Symfony Console component for generating the command
line interface. You can take a look at the ``vendor/bin/doctrine.php``
script and the ``Doctrine\ORM\Tools\Console\ConsoleRunner`` command
for inspiration how to setup the cli.

Doctrine 使用 Symfony 控制台组件来生成命令行的接口。你可以看一看 ``vendor/bin/doctrine.php``
脚本和 ``Doctrine\ORM\Tools\Console\ConsoleRunner`` 命令来启发如何配置该命令行接口。

In general the required code looks like this:

通常，必要的代码来起来像这样：

.. code-block:: php

    <?php
    $cli = new Application('Doctrine Command Line Interface', \Doctrine\ORM\Version::VERSION);
    $cli->setCatchExceptions(true);
    $cli->setHelperSet($helperSet);
    Doctrine\ORM\Tools\Console\ConsoleRunner::addCommands($cli);
    $cli->run();

