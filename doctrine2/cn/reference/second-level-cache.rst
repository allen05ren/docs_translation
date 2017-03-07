The Second Level Cache // 二级缓存
=========================================

.. note::

    The second level cache functionality is marked as experimental for now. It
    is a very complex feature and we cannot guarantee yet that it works stable
    in all cases.

    二级缓存功能现在被标记为实验性的。它是一个非常复杂的功能并且我们也不能保证它稳定工作在所有情况下。

The Second Level Cache is designed to reduce the amount of necessary database access.
It sits between your application and the database to avoid the number of database hits as much as possible.

二级缓存旨在减少必要的数据库访问量。它位于你的应用程序和数据库之间以尽可能避免数据库命中的数量。

When turned on, entities will be first searched in cache and if they are not found,
a database query will be fired and then the entity result will be stored in a cache provider.

当启用时，实体首先在缓存中搜索且如果它们没有找到，一个数据库查询将被触发，然后该实体结果将被存储在一个缓存提供者中。

There are some flavors of caching available, but is better to cache read-only data.

有一些口味的缓存可用，但要好于缓存只读数据

Be aware that caches are not aware of changes made to the persistent store by another application.
They can, however, be configured to regularly expire cached data.

注意，缓存不知道另一个应用程序对持久存储所做的更改。但是，它们可以被配置以定期地过期缓存的数据。

Caching Regions // 缓存区域
----------------------------------

Second level cache does not store instances of an entity, instead it caches only entity identifier and values.
Each entity class, collection association and query has its region, where values of each instance are stored.

二级缓存不能存储实体的实例，反而它仅缓存实体标识符和值。每个实体类、集合关联和查询有其区域，其中存储每个实例的值。

Caching Regions are specific region into the cache provider that might store entities, collection or queries.
Each cache region resides in a specific cache namespace and has its own lifetime configuration.

缓存区域是可能存储实体、集合或查询的缓存提供者中的特定区域。每个缓存区域驻留在一个特定的缓存命名空间中并且拥有其自己的生命周期时间配置。

Notice that when caching collection and queries only identifiers are stored.
The entity values will be stored in its own region

注意，当缓存集合和查询时仅存储标识符。实体的值将被存储在其自己的区域中。

Something like below for an entity region :

实体区域有点像下面：

.. code-block:: php

    <?php
    [
      'region_name:entity_1_hash' => ['id'=> 1, 'name' => 'FooBar', 'associationName'=>null],
      'region_name:entity_2_hash' => ['id'=> 2, 'name' => 'Foo', 'associationName'=>['id'=>11]],
      'region_name:entity_3_hash' => ['id'=> 3, 'name' => 'Bar', 'associationName'=>['id'=>22]]
    ];


If the entity holds a collection that also needs to be cached.
An collection region could look something like :

如果实体保存的一个集合也需要被缓存。一个集合区域可能看起来有点像：

.. code-block:: php

    <?php
    [
      'region_name:entity_1_coll_assoc_name_hash' => ['ownerId'=> 1, 'list' => [1, 2, 3]],
      'region_name:entity_2_coll_assoc_name_hash' => ['ownerId'=> 2, 'list' => [2, 3]],
      'region_name:entity_3_coll_assoc_name_hash' => ['ownerId'=> 3, 'list' => [2, 4]]
    ];

A query region might be something like :

一个查询区域可能有点像：

.. code-block:: php

    <?php
    [
      'region_name:query_1_hash' => ['list' => [1, 2, 3]],
      'region_name:query_2_hash' => ['list' => [2, 3]],
      'region_name:query_3_hash' => ['list' => [2, 4]]
    ];


.. note::

    The following data structures represents now the cache will looks like, this is not actual cached data.

    以下数据结构代表目前缓存将看起来像，这不是真实的缓存数据。

.. _reference-second-level-cache-regions:

Cache Regions // 缓存区域
--------------------------------

``Doctrine\ORM\Cache\Region\DefaultRegion`` It's the default implementation.
 A simplest cache region compatible with all doctrine-cache drivers but does not support locking.

``Doctrine\ORM\Cache\Region\DefaultRegion`` 它是默认的实现。
最简单的缓存区域与所有 Doctrine 缓存驱动兼容，但是不支持锁。

``Doctrine\ORM\Cache\Region`` and ``Doctrine\ORM\Cache\ConcurrentRegion``
Defines contracts that should be implemented by a cache provider.

``Doctrine\ORM\Cache\Region`` 和 ``Doctrine\ORM\Cache\ConcurrentRegion``
定义应该有缓存提供者实现的契约。

It allows you to provide your own cache implementation that might take advantage of specific cache driver.

它允许你提供你自己的缓存实现，该缓存实现可能利用特定缓存驱动的。

If you want to support locking for ``READ_WRITE`` strategies you should implement ``ConcurrentRegion``; ``CacheRegion`` otherwise.

如果你希望为 ``READ_WRITE`` 策略提供锁的支持，你应该实现 ``ConcurrentRegion``，否则，实现 ``CacheRegion``。

Cache region // 缓存区域
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Defines a contract for accessing a particular region.

为访问特别的区域定义一个契约。

``Doctrine\ORM\Cache\Region``

Defines a contract for accessing a particular cache region.

为访问特别的缓存区域定义一个契约。

`See API Doc <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.Region.html/>`_.

`查看 API 文档 <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.Region.html/>`_。

Concurrent cache region // 并发缓存区域
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A ``Doctrine\ORM\Cache\ConcurrentRegion`` is designed to store concurrently managed data region.
By default, Doctrine provides a very simple implementation based on file locks ``Doctrine\ORM\Cache\Region\FileLockRegion``.

``Doctrine\ORM\Cache\ConcurrentRegion`` 被设计以存储并发地 managed 数据区域。
默认地，Doctrine 提供一个非常简单的基于文件锁 ``Doctrine\ORM\Cache\Region\FileLockRegion`` 的实现。

If you want to use an ``READ_WRITE`` cache, you should consider providing your own cache region.

如果你希望使用一个 ``READ_WRITE`` 缓存，你应该考虑提供你自己的缓存区域。

``Doctrine\ORM\Cache\ConcurrentRegion``

Defines contract for concurrently managed data region.

为并发地 managed 数据区域定义的契约。

`See API Doc <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.ConcurrentRegion.html/>`_.

`查看 API 文档 <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.ConcurrentRegion.html/>`_。

Timestamp region // 时间戳区域
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``Doctrine\ORM\Cache\TimestampRegion``

Tracks the timestamps of the most recent updates to particular entity.

跟踪最近更新至特定实体的时间戳。

`See API Doc <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.TimestampRegion.html/>`_.

`查看 API 文档 <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.TimestampRegion.html/>`_.

.. _reference-second-level-cache-mode:

Caching mode // 缓存模式
-------------------------------

* ``READ_ONLY`` (DEFAULT)

  * Can do reads, inserts and deletes, cannot perform updates or employ any locks.
  * 可以读取、插入、删除，不能执行更新或利用任何锁。
  * Useful for data that is read frequently but never updated.
  * 对于经常读取的数据有用，但是从不更新。
  * Best performer.
  * 最好的执行者。
  * It is Simple.
  * 简单。

* ``NONSTRICT_READ_WRITE``

  * Read Write Cache doesn’t employ any locks but can do reads, inserts, updates and deletes.
  * 读写缓存不能利用任何锁，但是可以读取、插入、更新和删除。
  * Good if the application needs to update data rarely.
  * 如果应用程序很好需要更新，适用该模式。
    

* ``READ_WRITE``

  * Read Write cache employs locks before update/delete.
  * 读写缓存更新/删除之前利用锁。
  * Use if data needs to be updated.
  * 如果数据需要被更新，使用该模式。
  * Slowest strategy.
  * 最慢的策略。
  * To use it a the cache region implementation must support locking.
  * 为了使用它，缓存区域实现必须支持锁。


Built-in cached persisters // 内置缓存的持久器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cached persisters are responsible to access cache regions.

缓存的持久器负责访问缓存区域。

    +-----------------------+-------------------------------------------------------------------------------------------+
    | Cache Usage           | Persister                                                                                 |
    +=======================+===========================================================================================+
    | READ_ONLY             | Doctrine\\ORM\\Cache\\Persister\\Entity\\ReadOnlyCachedEntityPersister                    |
    +-----------------------+-------------------------------------------------------------------------------------------+
    | READ_WRITE            | Doctrine\\ORM\\Cache\\Persister\\Entity\\ReadWriteCachedEntityPersister                   |
    +-----------------------+-------------------------------------------------------------------------------------------+
    | NONSTRICT_READ_WRITE  | Doctrine\\ORM\\Cache\\Persister\\Entity\\NonStrictReadWriteCachedEntityPersister          |
    +-----------------------+-------------------------------------------------------------------------------------------+
    | READ_ONLY             | Doctrine\\ORM\\Cache\\Persister\\Collection\\ReadOnlyCachedCollectionPersister            |
    +-----------------------+-------------------------------------------------------------------------------------------+
    | READ_WRITE            | Doctrine\\ORM\\Cache\\Persister\\Collection\\ReadWriteCachedCollectionPersister           |
    +-----------------------+-------------------------------------------------------------------------------------------+
    | NONSTRICT_READ_WRITE  | Doctrine\\ORM\\Cache\\Persister\\Collection\\NonStrictReadWriteCachedCollectionPersister  |
    +-----------------------+-------------------------------------------------------------------------------------------+

Configuration // 配置
----------------------------
Doctrine allows you to specify configurations and some points of extension for the second-level-cache

Doctrine 允许你为二级缓存指定配置和一些扩展点。

Enable Second Level Cache // 启用二级缓存
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To enable the second-level-cache, you should provide a cache factory
``\Doctrine\ORM\Cache\DefaultCacheFactory`` is the default implementation.

为了启用二级缓存，你应该提供一个缓存工厂，``\Doctrine\ORM\Cache\DefaultCacheFactory`` 是一个默认实现。

.. code-block:: php

    <?php
    /* @var $config \Doctrine\ORM\Cache\RegionsConfiguration */
    /* @var $cache \Doctrine\Common\Cache\Cache */

    $factory = new \Doctrine\ORM\Cache\DefaultCacheFactory($config, $cache);

    // Enable second-level-cache
    $config->setSecondLevelCacheEnabled();

    // Cache factory
    $config->getSecondLevelCacheConfiguration()
        ->setCacheFactory($factory);


Cache Factory // 缓存工厂
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cache Factory is the main point of extension.

缓存工厂是主要的扩展点。

It allows you to provide a specific implementation of the following components :

它允许你提以一个以下组件的特定的实现：

* ``QueryCache`` Store and retrieve query cache results.
* ``QueryCache`` 存储和取回查询缓存结果。
* ``CachedEntityPersister`` Store and retrieve entity results.
* ``CachedEntityPersister`` 存储和取回实体结果。
* ``CachedCollectionPersister`` Store and retrieve query results.
* ``CachedCollectionPersister`` 存储和取回查询结果。
* ``EntityHydrator`` Transform an entity into a cache entry and cache entry into entities
* ``EntityHydrator`` 转换一个实体到一个缓存实体和缓存实体到实体。
* ``CollectionHydrator`` Transform a collection into a cache entry and cache entry into collection
* ``CollectionHydrator`` 转换一个集合到一个缓存实体和缓存实体到集合。

`See API Doc <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.DefaultCacheFactory.html/>`_.

`查看 API 文档 <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.DefaultCacheFactory.html/>`_。

Region Lifetime // 区域生命周期
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To specify a default lifetime for all regions or specify a different lifetime for a specific region.

为所有区域指定一个默认生命周期或为特定的区域指定一个不同的生命周期。

.. code-block:: php

    <?php
    /* @var $config \Doctrine\ORM\Configuration */
    /* @var $cacheConfig \Doctrine\ORM\Configuration */
    $cacheConfig  =  $config->getSecondLevelCacheConfiguration();
    $regionConfig =  $cacheConfig->getRegionsConfiguration();

    // Cache Region lifetime
    $regionConfig->setLifetime('my_entity_region', 3600);   // Time to live for a specific region; In seconds
    $regionConfig->setDefaultLifetime(7200);                // Default time to live; In seconds


Cache Log // 缓存日志
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
By providing a cache logger you should be able to get information about all cache operations such as hits, misses and puts.

通过提供一个缓存日志器，你应该能够获得关于所有缓存操作的信息，比如命中、未命中和放置。

``\Doctrine\ORM\Cache\Logging\StatisticsCacheLogger`` is a built-in implementation that provides basic statistics.

``\Doctrine\ORM\Cache\Logging\StatisticsCacheLogger`` 是内置的提供基础统计的实现。

 .. code-block:: php

    <?php
    /* @var $config \Doctrine\ORM\Configuration */
    $logger = new \Doctrine\ORM\Cache\Logging\StatisticsCacheLogger();

    // Cache logger
    $config->setSecondLevelCacheEnabled(true);
    $config->getSecondLevelCacheConfiguration()
        ->setCacheLogger($logger);


    // Collect cache statistics

    // Get the number of entries successfully retrieved from a specific region.
    $logger->getRegionHitCount('my_entity_region');

    // Get the number of cached entries *not* found in a specific region.
    $logger->getRegionMissCount('my_entity_region');

    // Get the number of cacheable entries put in cache.
    $logger->getRegionPutCount('my_entity_region');

    // Get the total number of put in all regions.
    $logger->getPutCount();

    // Get the total number of entries successfully retrieved from all regions.
    $logger->getHitCount();

    // Get the total number of cached entries *not* found in all regions.
    $logger->getMissCount();

If you want to get more information you should implement ``\Doctrine\ORM\Cache\Logging\CacheLogger``.
and collect all information you want.

如果你希望获得更多信息，你应该实现 ``\Doctrine\ORM\Cache\Logging\CacheLogger``。和收集你希望的所有信息。

`See API Doc <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.CacheLogger.html/>`_.

`查看 API 文档 <http://www.doctrine-project.org/api/orm/2.5/class-Doctrine.ORM.Cache.CacheLogger.html/>`_。

Entity cache definition // 实体缓存定义
---------------------------------------------
* Entity cache configuration allows you to define the caching strategy and region for an entity.
* 实体缓存配置允许你为实体定义缓存策略和区域。

  * ``usage`` Specifies the caching strategy: ``READ_ONLY``, ``NONSTRICT_READ_WRITE``, ``READ_WRITE``. see :ref:`reference-second-level-cache-mode`
  * ``usage`` 指定缓存策略：``READ_ONLY``、``NONSTRICT_READ_WRITE``、``READ_WRITE``。查看 :ref:`reference-second-level-cache-mode`
  * ``region`` Optional value that specifies the name of the second level cache region.
  * ``region`` 选项值指定二级缓存区域的名字。

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * @Entity
         * @Cache(usage="READ_ONLY", region="my_entity_region")
         */
        class Country
        {
            /**
             * @Id
             * @GeneratedValue
             * @Column(type="integer")
             */
            protected $id;

            /**
             * @Column(unique=true)
             */
            protected $name;

            // other properties and methods
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="utf-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
          <entity name="Country">
            <cache usage="READ_ONLY" region="my_entity_region" />
            <id name="id" type="integer" column="id">
              <generator strategy="IDENTITY"/>
            </id>
            <field name="name" type="string" column="name"/>
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Country:
          type: entity
          cache:
            usage : READ_ONLY
            region : my_entity_region
          id:
            id:
              type: integer
              id: true
              generator:
                strategy: IDENTITY
          fields:
            name:
              type: string


Association cache definition // 关联缓存定义
--------------------------------------------------
The most common use case is to cache entities. But we can also cache relationships.
It caches the primary keys of association and cache each element will be cached into its region.

最常见的使用案例是缓存实体。但是我们也可以缓存关联。
它缓存关联的主键和缓存每个元素将被缓存到其区域中。

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * @Entity
         * @Cache("NONSTRICT_READ_WRITE")
         */
        class State
        {
            /**
             * @Id
             * @GeneratedValue
             * @Column(type="integer")
             */
            protected $id;

            /**
             * @Column(unique=true)
             */
            protected $name;

            /**
             * @Cache("NONSTRICT_READ_WRITE")
             * @ManyToOne(targetEntity="Country")
             * @JoinColumn(name="country_id", referencedColumnName="id")
             */
            protected $country;

            /**
             * @Cache("NONSTRICT_READ_WRITE")
             * @OneToMany(targetEntity="City", mappedBy="state")
             */
            protected $cities;

            // other properties and methods
        }

    .. code-block:: xml

        <?xml version="1.0" encoding="utf-8"?>
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">
          <entity name="State">

            <cache usage="NONSTRICT_READ_WRITE" />

            <id name="id" type="integer" column="id">
              <generator strategy="IDENTITY"/>
            </id>

            <field name="name" type="string" column="name"/>
            
            <many-to-one field="country" target-entity="Country">
              <cache usage="NONSTRICT_READ_WRITE" />

              <join-columns>
                <join-column name="country_id" referenced-column-name="id"/>
              </join-columns>
            </many-to-one>

            <one-to-many field="cities" target-entity="City" mapped-by="state">
              <cache usage="NONSTRICT_READ_WRITE"/>
            </one-to-many>
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        State:
          type: entity
          cache:
            usage : NONSTRICT_READ_WRITE
          id:
            id:
              type: integer
              id: true
              generator:
                strategy: IDENTITY
          fields:
            name:
              type: string

          manyToOne:
            state:
              targetEntity: Country
              joinColumns:
                country_id:
                  referencedColumnName: id
              cache:
                usage : NONSTRICT_READ_WRITE

          oneToMany:
            cities:
              targetEntity:City
              mappedBy: state
              cache:
                usage : NONSTRICT_READ_WRITE


> Note: for this to work, the target entity must also be marked as cacheable.

> 注意：为此能工作，目标实体必须也标记为可缓存的（cacheable）。

Cache usage // 缓存用法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Basic entity cache

基础的实体缓存

.. code-block:: php

    <?php
    $em->persist(new Country($name));
    $em->flush();                         // Hit database to insert the row and put into cache

    $em->clear();                         // Clear entity manager

    $country1  = $em->find('Country', 1); // Retrieve item from cache

    $country->setName("New Name");
    $em->persist($country);
    $em->flush();                         // Hit database to update the row and update cache

    $em->clear();                         // Clear entity manager

    $country2  = $em->find('Country', 1); // Retrieve item from cache
                                          // Notice that $country1 and $country2 are not the same instance.


Association cache

关联缓存

.. code-block:: php

    <?php
    // Hit database to insert the row and put into cache
    $em->persist(new State($name, $country));
    $em->flush();

    // Clear entity manager
    $em->clear();

    // Retrieve item from cache
    $state = $em->find('State', 1);

    // Hit database to update the row and update cache entry
    $state->setName("New Name");
    $em->persist($state);
    $em->flush();

    // Create a new collection item
    $city = new City($name, $state);
    $state->addCity($city);

    // Hit database to insert new collection item,
    // put entity and collection cache into cache.
    $em->persist($city);
    $em->persist($state);
    $em->flush();

    // Clear entity manager
    $em->clear();

    // Retrieve item from cache
    $state = $em->find('State', 1);

    // Retrieve association from cache
    $country = $state->getCountry();

    // Retrieve collection from cache
    $cities = $state->getCities();

    echo $country->getName();
    echo $state->getName();

    // Retrieve each collection item from cache
    foreach ($cities as $city) {
        echo $city->getName();
    }

.. note::

    Notice that all entities should be marked as cacheable.

    注意所有实体应该标记为可缓存的（cacheable）。

Using the query cache // 使用查询缓存
-------------------------------------------

The second level cache stores the entities, associations and collections.
The query cache stores the results of the query but as identifiers, entity values are actually stored in the 2nd level cache.

二级缓存存储实体、关联和集合。
查询缓存存储查询的结果，但是作为标识符、实体的值实际上被存储在二级缓存中。

.. note::

    Query cache should always be used in conjunction with the second-level-cache for those entities which should be cached.

    为了实体能够被缓存，查询缓存应该始终与二级缓存一起使用。

.. code-block:: php

    <?php
    /* @var $em \Doctrine\ORM\EntityManager */

    // Execute database query, store query cache and entity cache
    $result1 = $em->createQuery('SELECT c FROM Country c ORDER BY c.name')
        ->setCacheable(true)
        ->getResult();

    $em->clear()

    // Check if query result is valid and load entities from cache
    $result2 = $em->createQuery('SELECT c FROM Country c ORDER BY c.name')
        ->setCacheable(true)
        ->getResult();

Cache mode // 缓存模式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Cache Mode controls how a particular query interacts with the second-level cache:

缓存模式控制特定的查询如何与二级缓存交互：

* ``Cache::MODE_GET`` - May read items from the cache, but will not add items.
* ``Cache::MODE_GET`` - 可以从缓存读取条目，但将不会添加条目。
* ``Cache::MODE_PUT`` - Will never read items from the cache, but will add items to the cache as it reads them from the database.
* ``Cache::MODE_PUT`` - 将永不从缓存读取条目，但将添加条目到缓存，因为它从数据库读取它们。
* ``Cache::MODE_NORMAL`` - May read items from the cache, and add items to the cache.
* ``Cache::MODE_NORMAL`` - 可以从缓存读取条目，并添加条目到缓存。
* ``Cache::MODE_REFRESH`` - The query will never read items from the cache, but will refresh items to the cache as it reads them from the database.
* ``Cache::MODE_REFRESH`` - 查询将永不从缓存读取条目，但将刷新（refresh）条目到缓存，因为它从数据库读取它们。

.. code-block:: php

    <?php
    /* @var $em \Doctrine\ORM\EntityManager */
    // Will refresh the query cache and all entities the cache as it reads from the database.
    $result1 = $em->createQuery('SELECT c FROM Country c ORDER BY c.name')
        ->setCacheMode(Cache::MODE_GET)
        ->setCacheable(true)
        ->getResult();

.. note::

    The the default query cache mode is ```Cache::MODE_NORMAL```

    默认的查询缓存模式是 ```Cache::MODE_NORMAL```。

DELETE / UPDATE queries // DELETE / UPDATE 查询
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DQL UPDATE / DELETE statements are ported directly into a database and bypass the second-level cache,
Entities that are already cached will NOT be invalidated.
However the cached data could be evicted using the cache API or an special query hint.

DQL UPDATE / DELETE 语句直接地移植到数据库并绕过二级缓存，已经缓存的实体将不会失效。
但是，缓存的数据可以使用缓存 API 或专门的查询暗示回收。

Execute the ``UPDATE`` and invalidate ``all cache entries`` using ``Query::HINT_CACHE_EVICT``

执行 ``UPDATE`` 并使用 ``Query::HINT_CACHE_EVICT`` 使 ``所有的缓存实体`` 失效。

.. code-block:: php

    <?php
    // Execute and invalidate
    $this->_em->createQuery("UPDATE Entity\Country u SET u.name = 'unknown' WHERE u.id = 1")
        ->setHint(Query::HINT_CACHE_EVICT, true)
        ->execute();


Execute the ``UPDATE`` and invalidate ``all cache entries`` using the cache API

执行 ``UPDATE`` 并使用缓存 API 使 ``所有的缓存实体`` 失效。

.. code-block:: php

    <?php
    // Execute
    $this->_em->createQuery("UPDATE Entity\Country u SET u.name = 'unknown' WHERE u.id = 1")
        ->execute();
    // Invoke Cache API
    $em->getCache()->evictEntityRegion('Entity\Country');


Execute the ``UPDATE`` and invalidate ``a specific cache entry`` using the cache API

执行 ``UPDATE`` 并使用缓存 API 使 ``特定的缓存实体`` 失效。

.. code-block:: php

    <?php
    // Execute
    $this->_em->createQuery("UPDATE Entity\Country u SET u.name = 'unknown' WHERE u.id = 1")
        ->execute();
    // Invoke Cache API
    $em->getCache()->evictEntity('Entity\Country', 1);

Using the repository query cache // 使用仓库查询缓存
---------------------------------------------------------

As well as ``Query Cache`` all persister queries store only identifier values for an individual query.
All persister use a single timestamps cache region keeps track of the last update for each persister,
When a query is loaded from cache, the timestamp region is checked for the last update for that persister.
Using the last update timestamps as part of the query key invalidate the cache key when an update occurs.

与 ``查询缓存`` 一样，所有的持久化器查询仅为单独的查询存储标识符值。所有持久化器使用单一时间戳缓存区域保持每个持久化器的最近更新的跟踪，
当一个查询从缓存被加载，时间戳区域检查持久化器最近的更新。当一个更新发生时。使用最近的更新时间戳作为查询键使缓存键无效。

.. code-block:: php

    <?php
    // load from database and store cache query key hashing the query + parameters + last timestamp cache region..
    $entities   = $em->getRepository('Entity\Country')->findAll();

    // load from query and entities from cache..
    $entities   = $em->getRepository('Entity\Country')->findAll();

    // update the timestamp cache region for Country
    $em->persist(new Country('zombieland'));
    $em->flush();
    $em->clear();

    // Reload from database.
    // At this point the query cache key if not logger valid, the select goes straight
    $entities   = $em->getRepository('Entity\Country')->findAll();

Cache API // 缓存 API
----------------------------

Caches are not aware of changes made by another application.
However, you can use the cache API to check / invalidate cache entries.

缓存不知道另一个应用程序所做的更改。但是，你可以使用缓存 API 检查缓存实体/使缓存实体失效。

.. code-block:: php

    <?php
    /* @var $cache \Doctrine\ORM\Cache */
    $cache = $em->getCache();

    $cache->containsEntity('Entity\State', 1)      // Check if the cache exists
    $cache->evictEntity('Entity\State', 1);        // Remove an entity from cache
    $cache->evictEntityRegion('Entity\State');     // Remove all entities from cache

    $cache->containsCollection('Entity\State', 'cities', 1);   // Check if the cache exists
    $cache->evictCollection('Entity\State', 'cities', 1);      // Remove an entity collection from cache
    $cache->evictCollectionRegion('Entity\State', 'cities');   // Remove all collections from cache

Limitations // 局限性
----------------------------

Composite primary key // 复合主键
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Composite primary key are supported by second level cache,
however when one of the keys is an association the cached entity should always be retrieved using the association identifier.
For performance reasons the cache API does not extract from composite primary key.

复合主键由二级缓存所支持，但是当由有一个键是关联，缓存实体应该始终使用关联标识符取回。
为了性能的原因，缓存 API 不能从复合主键提取。

.. code-block:: php

    <?php
    /**
     * @Entity
     */
    class Reference
    {
        /**
         * @Id
         * @ManyToOne(targetEntity="Article", inversedBy="references")
         * @JoinColumn(name="source_id", referencedColumnName="article_id")
         */
        private $source;

        /**
         * @Id
         * @ManyToOne(targetEntity="Article")
         * @JoinColumn(name="target_id", referencedColumnName="article_id")
         */
        private $target;
    }

    // Supported
    /* @var $article Article */
    $article = $em->find('Article', 1);

    // Supported
    /* @var $article Article */
    $article = $em->find('Article', $article);

    // Supported
    $id        = array('source' => 1, 'target' => 2);
    $reference = $em->find('Reference', $id);

    // NOT Supported
    $id        = array('source' => new Article(1), 'target' => new Article(2));
    $reference = $em->find('Reference', $id);

Distributed environments // 分布式的环境
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Some cache driver are not meant to be used in a distributed environment.
Load-balancer for distributing workloads across multiple computing resources
should be used in conjunction with distributed caching system such as memcached, redis, riak ...

一些查询驱动不是旨在用于分布式的环境。负载均衡器为分发工作负担跨越多个计算资源，应该与分布式的缓存系统一起使用，
比如，memcached、redis、riak ...

Caches should be used with care when using a load-balancer if you don't share the cache.
While using APC or any file based cache update occurred in a specific machine would not reflect to the cache in other machines.

当使用负载均衡器时，如果你不共享缓存，缓存应该小心使用。

Paginator // 分页器
~~~~~~~~~~~~~~~~~~~~~~~~~~

Count queries generated by ``Doctrine\ORM\Tools\Pagination\Paginator`` are not cached by second-level cache.
Although entities and query result are cached count queries will hit the database every time.

由 ``Doctrine\ORM\Tools\Pagination\Paginator`` 生成的计数查询不能通过二级缓存来缓存。
尽管实体和查询结果被缓存,但集数查询将每次命中数据库。
