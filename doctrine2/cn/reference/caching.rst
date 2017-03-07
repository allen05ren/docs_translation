Caching // 缓存
====================

Doctrine provides cache drivers in the ``Common`` package for some
of the most popular caching implementations such as APC, Memcache
and Xcache. We also provide an ``ArrayCache`` driver which stores
the data in a PHP array. Obviously, when using ``ArrayCache``, the 
cache does not persist between requests, but this is useful for 
testing in a development environment.

Doctrine 在 ``Common``包中为一些最受欢迎的缓存实现，比如 APC、Memcache和 Xcache
提供了缓存驱动。我们也提供了 ``ArrayCache`` 驱动在 PHP 数组中存储数据。很显然，当使用
``ArrayCache`` 时，请求之间缓存没有持久，但是对于开发环境中的测试这是非常有用的。

Cache Drivers // 缓存驱动
------------------------------

The cache drivers follow a simple interface that is defined in
``Doctrine\Common\Cache\Cache``. All the cache drivers extend a
base class ``Doctrine\Common\Cache\CacheProvider`` which implements
this interface.

缓存驱动遵循一个在 ``Doctrine\Common\Cache\Cache`` 定义的简单接口。所有缓存驱动扩展了
一个实现了此接口的基类 ``Doctrine\Common\Cache\CacheProvider``。

The interface defines the following public methods for you to implement:

此接口定义了以下 public 方法用于你去实现：

-  fetch($id) - Fetches an entry from the cache
-  fetch($id) - 从缓存中取回一个条目
-  contains($id) - Test if an entry exists in the cache
-  contains($id) - 测试在缓存中是否存在一个条目
-  save($id, $data, $lifeTime = false) - Puts data into the cache for x seconds. 0 = infinite time
-  save($id, $data, $lifeTime = false) - 将数据放入缓存 x 秒。0=无限时间
-  delete($id) - Deletes a cache entry
-  delete($id) - 删除一个缓存条目

Each driver extends the ``CacheProvider`` class which defines a few
abstract protected methods that each of the drivers must
implement:

每个驱动扩展了定义了一些每个缓存驱动必须实现的抽象的 protected 方法的 ``CacheProvider`` 类。

-  \_doFetch($id)
-  \_doContains($id)
-  \_doSave($id, $data, $lifeTime = false)
-  \_doDelete($id)

The public methods ``fetch()``, ``contains()`` etc. use the
above protected methods which are implemented by the drivers. The
code is organized this way so that the protected methods in the
drivers do the raw interaction with the cache implementation and
the ``CacheProvider`` can build custom functionality on top of
these methods.

public 方法 ``fetch()``、``contains()`` 等使用以上的由驱动实现的 protected 方法。
代码以此方式被组织，所以在驱动中的这些 protected 方法做了与缓存实现的原始（raw）交互并且
``CacheProvider`` 可以基于这些方法构建自定义的功能。

This documentation does not cover every single cache driver included
with Doctrine. For an up-to-date-list, see the
`cache directory on GitHub <https://github.com/doctrine/cache/tree/master/lib/Doctrine/Common/Cache>`_.

本文档没有覆盖 Doctrine 包含的每个缓存驱动，包括。一个更新的列表，请查看
`GitHub 上的缓存目录 <https://github.com/doctrine/cache/tree/master/lib/Doctrine/Common/Cache>`_。

APC
~~~

In order to use the APC cache driver you must have it compiled and
enabled in your php.ini. You can read about APC
`in the PHP Documentation <http://us2.php.net/apc>`_. It will give
you a little background information about what it is and how you
can use it as well as how to install it.

为了使用 APC 缓存驱动你必须已经编译了它并在你的 php.ini 中启用。你可以
`在 PHP 中的文档 <http://us2.php.net/apc>`_ 了解 APC。它将给你一些背景信息，关于它是什么、
你如何使用它以及如何安装它。

Below is a simple example of how you could use the APC cache driver
by itself.

以下是一个简单的例子，你可以如何使用 APC 缓存驱动自身。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\ApcCache();
    $cacheDriver->save('cache_id', 'my_data');

Memcache
~~~~~~~~

In order to use the Memcache cache driver you must have it compiled
and enabled in your php.ini. You can read about Memcache
`on the PHP website <http://php.net/memcache>`_. It will
give you a little background information about what it is and how
you can use it as well as how to install it.

为了使用 Memcache 缓存驱动你必须已经编译了它并在你的 php.ini 中启用。你可以
`在 PHP 网站上 <http://php.net/memcache>`_ 了解 Memcache。它将给你一些背景信息，关于它是什么、
你如何使用它以及如何安装它。

Below is a simple example of how you could use the Memcache cache
driver by itself.

以下是一个简单的例子，你可以如何使用 Memcache 缓存驱动自身。

.. code-block:: php

    <?php
    $memcache = new Memcache();
    $memcache->connect('memcache_host', 11211);
    
    $cacheDriver = new \Doctrine\Common\Cache\MemcacheCache();
    $cacheDriver->setMemcache($memcache);
    $cacheDriver->save('cache_id', 'my_data');

Memcached
~~~~~~~~~

Memcached is a more recent and complete alternative extension to
Memcache.

Memcached 是一个更新的且完整的可替换 Memcache 的扩展。

In order to use the Memcached cache driver you must have it compiled
and enabled in your php.ini. You can read about Memcached
`on the PHP website <http://php.net/memcached>`_. It will
give you a little background information about what it is and how
you can use it as well as how to install it.

为了使用 Memcached 缓存驱动你必须已经编译了它并在你的 php.ini 中启用。你可以
`在 PHP 网站上 <http://php.net/memcached>`_ 了解 Memcached。它将给你一些背景信息，关于它是什么、
你如何使用它以及如何安装它。

Below is a simple example of how you could use the Memcached cache
driver by itself.

以下是一个简单的例子，你可以如何使用 Memcached 缓存驱动自身。

.. code-block:: php

    <?php
    $memcached = new Memcached();
    $memcached->addServer('memcache_host', 11211);
    
    $cacheDriver = new \Doctrine\Common\Cache\MemcachedCache();
    $cacheDriver->setMemcached($memcached);
    $cacheDriver->save('cache_id', 'my_data');

Xcache
~~~~~~

In order to use the Xcache cache driver you must have it compiled
and enabled in your php.ini. You can read about Xcache
`here <http://xcache.lighttpd.net/>`_. It will give you a little
background information about what it is and how you can use it as
well as how to install it.

为了使用 Xcache 缓存驱动你必须已经编译了它并在你的 php.ini 中启用。你可以
`在这里 <http://xcache.lighttpd.net/>`_ 了解 Xcache。它将给你一些背景信息，关于它是什么、
你如何使用它以及如何安装它。

Below is a simple example of how you could use the Xcache cache
driver by itself.

以下是一个简单的例子，你可以如何使用 Xcache 缓存驱动自身。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\XcacheCache();
    $cacheDriver->save('cache_id', 'my_data');

Redis
~~~~~

In order to use the Redis cache driver you must have it compiled
and enabled in your php.ini. You can read about what Redis is
`from here <http://redis.io/>`_. Also check
`A PHP extension for Redis <https://github.com/nicolasff/phpredis/>`_ for how you can use
and install the Redis PHP extension.

为了使用 Redis 缓存驱动你必须已经编译了它并在你的 php.ini 中启用。你可以
`从这里 <http://redis.io/>`_ 阅读关于什么是 Redis。也可以查看 `Redis 的 PHP 扩展 <https://github.com/nicolasff/phpredis/>`_
，如何使用和安装 Redis 的 PHP 扩展。

Below is a simple example of how you could use the Redis cache
driver by itself.

以下是一个简单的例子，你可以如何使用 Redis 缓存驱动自身。

.. code-block:: php

    <?php
    $redis = new Redis();
    $redis->connect('redis_host', 6379);

    $cacheDriver = new \Doctrine\Common\Cache\RedisCache();
    $cacheDriver->setRedis($redis);
    $cacheDriver->save('cache_id', 'my_data');

Using Cache Drivers // 使用缓存驱动
---------------------------------------

In this section we'll describe how you can fully utilize the API of
the cache drivers to save data to a cache, check if some cached data 
exists, fetch the cached data and delete the cached data. We'll use the
``ArrayCache`` implementation as our example here.

在本部分我们将描述你如何可以完整地利用缓存的 API 来保存数据到缓存、查看一些缓存的数据是否存在、
取回缓存的数据和删除缓存的数据。这里我们将使用 ``ArrayCache`` 实现作为我们的例子。

.. code-block:: php

    <?php
    $cacheDriver = new \Doctrine\Common\Cache\ArrayCache();

Saving // 保存
~~~~~~~~~~~~~~~~~~~

Saving some data to the cache driver is as simple as using the
``save()`` method.

使用 ``save()`` 方法保存一些数据到缓存驱动就是这么简单。

.. code-block:: php

    <?php
    $cacheDriver->save('cache_id', 'my_data');

The ``save()`` method accepts three arguments which are described
below:

``save()`` 方法接受三个参数，描述如下：

-  ``$id`` - The cache id
-  ``$id`` - 缓存 id。
-  ``$data`` - The cache entry/data.
-  ``$data`` - 缓存条目/数据。
-  ``$lifeTime`` - The lifetime. If != false, sets a specific
   lifetime for this cache entry (null => infinite lifeTime).
-  ``$lifeTime`` - 生命周期。如果 != false，为此缓存条目设置一个特定的生命周期
   （null => 无限生命周期）。

You can save any type of data whether it be a string, array,
object, etc.

你可以保存任何类型的数据，不管它是一个字符串、数组、对象等。

.. code-block:: php

    <?php
    $array = array(
        'key1' => 'value1',
        'key2' => 'value2'
    );
    $cacheDriver->save('my_array', $array);

Checking // 检查
~~~~~~~~~~~~~~~~~~~~~

Checking whether cached data exists is very simple: just use the
``contains()`` method. It accepts a single argument which is the ID
of the cache entry.

检查是否缓存的数据存在是非常简单的：仅使用 ``contains()`` 方法。它接受一个参数，即缓存条目的 id。

.. code-block:: php

    <?php
    if ($cacheDriver->contains('cache_id')) {
        echo 'cache exists';
    } else {
        echo 'cache does not exist';
    }

Fetching // 取回
~~~~~~~~~~~~~~~~~~~~~

Now if you want to retrieve some cache entry you can use the
``fetch()`` method. It also accepts a single argument just like
``contains()`` which is again the ID of the cache entry.

现在如果你希望取回一些缓存条目，你可以使用 ``fetch()`` 方法。它仅接受一个参数，类似于
``contains()`` 仍是缓存条目的 id。

.. code-block:: php

    <?php
    $array = $cacheDriver->fetch('my_array');

Deleting // 删除
~~~~~~~~~~~~~~~~~~~~~

As you might guess, deleting is just as easy as saving, checking
and fetching. You can delete by an individual ID, or you can 
delete all entries.

正如你可能猜到的，删除就像保存、检查和取回一样简单。你可以通过单独的 ID 删除或你可以
删除所有条目。

By Cache ID // 通过缓存id
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: php

    <?php
    $cacheDriver->delete('my_array');

All // 所有
^^^

If you simply want to delete all cache entries you can do so with
the ``deleteAll()`` method.

如果你只是希望删除所有缓存条目，你可以使用 ``deleteAll()`` 方法。

.. code-block:: php

    <?php
    $deleted = $cacheDriver->deleteAll();

Namespaces // 命名空间
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you heavily use caching in your application and use it in
multiple parts of your application, or use it in different
applications on the same server you may have issues with cache
naming collisions. This can be worked around by using namespaces.
You can set the namespace a cache driver should use by using the
``setNamespace()`` method.

如果你在你的应用程序中大量使用缓存并在你的应用程序的多个部分使用它，或在同一服务器上的不同
应用程序中使用它，你可能有缓存命名冲突的问题。这可以通过使用命名空间来解决。你可以通过使用
``setNamespace()`` 方法设置缓存驱动应该使用的命名空间。

.. code-block:: php

    <?php
    $cacheDriver->setNamespace('my_namespace_');

Integrating with the ORM // 与 ORM 集成
--------------------------------------------

The Doctrine ORM package is tightly integrated with the cache
drivers to allow you to improve the performance of various aspects of
Doctrine by simply making some additional configurations and
method calls.

Doctrine ORM 包与缓存驱动紧密地集成，允许你通过简单地进行一些额外配置和方法调用来增强
Doctrine 各个方面的性能。

Query Cache // 查询缓存
~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is highly recommended that in a production environment you cache
the transformation of a DQL query to its SQL counterpart. It
doesn't make sense to do this parsing multiple times as it doesn't
change unless you alter the DQL query.

强烈地推荐你在生产环境中缓存 DQL 查询到对应的 SQL 的转换。做这样的解析多次是没有意义的，
因为它没有变更，除非你修改了此 DQL 查询。

This can be done by configuring the query cache implementation to
use on your ORM configuration.

这可以通过在你的 ORM 配置上配置使用查询缓存实现来完成.

.. code-block:: php

    <?php
    $config = new \Doctrine\ORM\Configuration();
    $config->setQueryCacheImpl(new \Doctrine\Common\Cache\ApcCache());

Result Cache // 结果缓存
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The result cache can be used to cache the results of your queries
so that we don't have to query the database or hydrate the data
again after the first time. You just need to configure the result
cache implementation.

结果缓存可以被用于缓存你的查询结果，所以我们不必在第一次后再一次查询数据库或水合数据。
你仅需要配置结果缓存的实现。

.. code-block:: php

    <?php
    $config->setResultCacheImpl(new \Doctrine\Common\Cache\ApcCache());

Now when you're executing DQL queries you can configure them to use
the result cache.

现在当你正在执行 DQL 查询你可以配置它们以使集结果缓存。

.. code-block:: php

    <?php
    $query = $em->createQuery('select u from \Entities\User u');
    $query->useResultCache(true);

You can also configure an individual query to use a different
result cache driver.

你也可以配置一个单独的查询使用一个不同的结果缓存驱动。

.. code-block:: php

    <?php
    $query->setResultCacheDriver(new \Doctrine\Common\Cache\ApcCache());

.. note::

    Setting the result cache driver on the query will
    automatically enable the result cache for the query. If you want to
    disable it pass false to ``useResultCache()``.

    在查询上设置结果缓存将自动地为该查询启用结果缓存。如果你希望禁用它，传递 false 到
    ``useResultCache()``。

    ::

        <?php
        $query->useResultCache(false);


If you want to set the time the cache has to live you can use the
``setResultCacheLifetime()`` method.

如果你希望设置缓存时间，你可以使用 ``setResultCacheLifetime()`` 方法。

.. code-block:: php

    <?php
    $query->setResultCacheLifetime(3600);

The ID used to store the result set cache is a hash which is
automatically generated for you if you don't set a custom ID
yourself with the ``setResultCacheId()`` method.

如果你没有使用 ``setResultCacheId()`` 方法自己设置一个自定义的 ID，将
自动地为你生成一个哈希（hash）用于存储结果集缓存的 ID。

.. code-block:: php

    <?php
    $query->setResultCacheId('my_custom_id');

You can also set the lifetime and cache ID by passing the values as
the second and third argument to ``useResultCache()``.

你也可以通过传递值作为第二个和第三个参数到 ``useResultCache()`` 设置生命周期和缓存 ID。

.. code-block:: php

    <?php
    $query->useResultCache(true, 3600, 'my_custom_id');

Metadata Cache // 元数据缓存
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your class metadata can be parsed from a few different sources like
YAML, XML, Annotations, etc. Instead of parsing this information on
each request we should cache it using one of the cache drivers.

你的类元数据可以从不同的源解析，如 YAML、XML、注释等等。作为在每次请求上解析这些信息的替代，
我们应该使用缓存驱动之一缓存它。

Just like the query and result cache we need to configure it
first.

同查询和结果缓存一样，首先我们需要配置它。

.. code-block:: php

    <?php
    $config->setMetadataCacheImpl(new \Doctrine\Common\Cache\ApcCache());

Now the metadata information will only be parsed once and stored in
the cache driver.

现在元数据信息将仅解析一次并存储在缓存驱动中。

Clearing the Cache // 清除缓存
-----------------------------------

We've already shown you how you can use the API of the
cache drivers to manually delete cache entries. For your
convenience we offer command line tasks to help you with
clearing the query, result and metadata cache.

我们已经展示给你如何使用缓存驱动 API 以手动删除缓存条目。为了你的方便我们提供了
命令行任务以帮助你清除查询、结果和元数据缓存。

From the Doctrine command line you can run the following commands:

从 Doctrine 命令行你可以运行下列命令：

To clear the query cache use the ``orm:clear-cache:query`` task.

为清除查询缓存使用 ``orm:clear-cache:query`` 任务。

.. code-block:: php

    $ ./doctrine orm:clear-cache:query

To clear the metadata cache use the ``orm:clear-cache:metadata`` task.

为清除元数据缓存使用 ``orm:clear-cache:metadata`` 任务。

.. code-block:: php

    $ ./doctrine orm:clear-cache:metadata

To clear the result cache use the ``orm:clear-cache:result`` task.

为清除结果缓存使用 ``orm:clear-cache:result`` 任务。

.. code-block:: php

    $ ./doctrine orm:clear-cache:result

All these tasks accept a ``--flush`` option to flush the entire
contents of the cache instead of invalidating the entries.

所有这些任务接受一个 ``--flush`` 选项以 flush 缓存的全部内容，替代使条目无效。

Cache Chaining // 缓存链
-----------------------------

A common pattern is to use a static cache to store data that is
requested many times in a single PHP request. Even though this data
may be stored in a fast memory cache, often that cache is over a
network link leading to sizable network traffic.

通常的模式是使用静态缓存存储在单个 PHP 请求中需要多次请求的数据。即使这些数据可能被
存储在一个快速的内存缓存，通常这类缓存是基于网络连接的，这导致了相当客观的网络流量。

The ChainCache class allows multiple caches to be registered at once.
For example, a per-request ArrayCache can be used first, followed by
a (relatively) slower MemcacheCache if the ArrayCache misses.
ChainCache automatically handles pushing data up to faster caches in
the chain and clearing data in the entire stack when it is deleted.

ChainCache 类允许一次注册多缓存。例如，每个请求 ArrayCache 可以被首先使用，如果
ArrayCache 丢失（misses），随后是一个（相对）较慢的 MemcacheCache。在该链中，
ChainCache 自动地处理推送数据到更快的缓存并且当它被删除时在整个栈中清除数据。

A ChainCache takes a simple array of CacheProviders in the order that
they should be used.

ChainCache 按它们应该被使用的顺序接受一个简单的 CacheProviders 数组。

.. code-block:: php

    $arrayCache = new \Doctrine\Common\Cache\ArrayCache();
    $memcache = new Memcache();
    $memcache->connect('memcache_host', 11211);
    $chainCache = new \Doctrine\Common\Cache\ChainCache([
        $arrayCache,
        $memcache,
    ]);

ChainCache itself extends the CacheProvider interface, so it is
possible to create chains of chains. While this may seem like an easy
way to build a simple high-availability cache, ChainCache does not
implement any exception handling so using it as a high-availability
mechanism is not recommended.

ChainCache 自身扩展了 CacheProvider 接口，所以创建链的时是可能的。虽然这可能
看上去像一种简单的方式以构建一个简单的高可用缓存，ChainCache 不能实现任何异常的处理，
所以使用它作为高可用机制是不推荐的。

Cache Slams // 缓存碰撞
----------------------------

Something to be careful of when using the cache drivers is
"cache slams". Imagine you have a heavily trafficked website with some
code that checks for the existence of a cache record and if it does
not exist it generates the information and saves it to the cache.
Now, if 100 requests were issued all at the same time and each one
sees the cache does not exist and they all try to insert the same
cache entry it could lock up APC, Xcache, etc. and cause problems.
Ways exist to work around this, like pre-populating your cache and
not letting your users' requests populate the cache.

当使用缓存驱动程序时要小心的事情是 “缓存碰撞（cache slams）”。设想你有一个大流量的网站
使用一些代码检查缓存记录的存在并且如果它不存在便生成信息并存储到缓存。现在，如果在同一时间
100请求被发出并且每个都看到该缓存不存在并它们尝试插入同样的缓存条目，它可能锁定 APC、Xcache等
并且导致问题。存在绕过此问题的方式，像预填充（pre-populating）你的缓存和不让你的用户的请求填充缓存。

You can read more about cache slams
`in this blog post <http://notmysock.org/blog/php/user-cache-timebomb.html>`_.

你可以阅读更多有关缓存碰撞（cache slams）`在此博客文章中 <http://notmysock.org/blog/php/user-cache-timebomb.html>`_。
