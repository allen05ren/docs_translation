Doctrine Internals explained // Doctrine 内部解释
=========================================================

Object relational mapping is a complex topic and sufficiently understanding how Doctrine works internally helps you use its full power.

对象关系映射是一个复杂的话题，充分地理解 Doctrine 内部如何工作有助于你使用它全部的功能。

How Doctrine keeps track of Objects // Doctrine 如何保持对象的跟踪
-------------------------------------------------------------------------

Doctrine uses the Identity Map pattern to track objects. Whenever you fetch an
object from the database, Doctrine will keep a reference to this object inside
its UnitOfWork. The array holding all the entity references is two-levels deep
and has the keys "root entity name" and "id". Since Doctrine allows composite
keys the id is a sorted, serialized version of all the key columns.

Doctrine 使用身份映射模式来跟踪对象。一旦你从数据库取回一个对象，Doctrine 将保持一个到该对象内部
的 UnitOfWork 的引用。该数组保存所有实体引用在两个层次上，并有键“root entity name”和“id”。
因为 Doctrine 允许复合键，id 是一个所有键列排序的、序列化的的版本。

This allows Doctrine room for optimizations. If you call the EntityManager and
ask for an entity with a specific ID twice, it will return the same instance:

这允许 Doctrine 优化空间。如果你调用 EntityManager 并使用一个特定 id 查找一个实体两次，它
将返回同一个实例：

.. code-block:: php

    public function testIdentityMap()
    {
        $objectA = $this->entityManager->find('EntityName', 1);
        $objectB = $this->entityManager->find('EntityName', 1);

        $this->assertSame($objectA, $objectB)
    }

Only one SELECT query will be fired against the database here. In the second
``EntityManager#find()`` call Doctrine will check the identity map first and
doesn't need to make that database roundtrip.

这里仅针对数据库触发一个 SELECT 查询。第二次调用 ``EntityManager#find()`` Doctrine
将首先检查身份映射且无需往返数据库。

Even if you get a proxy object first then fetch the object by the same id you
will still end up with the same reference:

即使你首先得到一个代理对象，然后通过同样的 id 获取该对象，你将仍然得到同样的引用：

.. code-block:: php

    public function testIdentityMapReference()
    {
        $objectA = $this->entityManager->getReference('EntityName', 1);
        // check for proxyinterface
        $this->assertInstanceOf('Doctrine\ORM\Proxy\Proxy', $objectA);

        $objectB = $this->entityManager->find('EntityName', 1);

        $this->assertSame($objectA, $objectB)
    }

The identity map being indexed by primary keys only allows shortcuts when you
ask for objects by primary key. Assume you have the following ``persons``
table:

身份映射通过主键被索引，当你通过主键查找对象时仅允许快捷方式。假设你有以下 ``persons`` 表：

::

    id | name
    -------------
    1  | Benjamin
    2  | Bud

Take the following example where two
consecutive calls are made against a repository to fetch an entity by a set of
criteria:

看如下例子，其中针对一个 repository 的两次连续的调用以通过一组条件（criteria）获取一个实体：

.. code-block:: php

    public function testIdentityMapRepositoryFindBy()
    {
        $repository = $this->entityManager->getRepository('Person');
        $objectA = $repository->findOneBy(array('name' => 'Benjamin'));
        $objectB = $repository->findOneBy(array('name' => 'Benjamin'));

        $this->assertSame($objectA, $objectB);
    }

This query will still return the same references and `$objectA` and `$objectB`
are indeed referencing the same object. However when checking your SQL logs you
will realize that two queries have been executed against the database. Doctrine
only knows objects by id, so a query for different criteria has to go to the
database, even if it was executed just before.

这个查询将仍然返回同样的引用，并且 `$objectA` 和 `$objectB` 实际上引用了同一个对象。
然而，当检查你的 SQL 日志时你将意识到两个针对数据库的查询已经被执行。Doctrine 仅通过 id 了解对象，
所以对于不同条件（criteria）的查询必须去数据库，即便它刚刚被执行。

But instead of creating a second Person object Doctrine first gets the primary
key from the row and check if it already has an object inside the UnitOfWork
with that primary key. In our example it finds an object and decides to return
this instead of creating a new one.

但是作为生成的第二个 Person 对象的替换，Doctrine 首先从数据库行得到主键，然后检查是否它已经有一个对象内部的
UnitOfWork 带有该主键。在我们的例子中，它查找一个对象并决定返回引用替换生成的一个新的。

The identity map has a second use-case. When you call ``EntityManager#flush``
Doctrine will ask the identity map for all objects that are currently managed.
This means you don't have to call ``EntityManager#persist`` over and over again
to pass known objects to the EntityManager. This is a NO-OP for known entities,
but leads to much code written that is confusing to other developers.

身份映射有另一个用途。当你调用 ``EntityManager#flush`` Doctrine 将查找目前被托管（managed）的所有对象的身份映射。
这意味着你不必调用 ``EntityManager#persist`` 反复地传递已知对象到 EntityManager。对于已知实体这是一个空操作（NO-OP），
但是引起了大量代码书写，这正在困扰其他的开发人员。

The following code WILL update your database with the changes made to the
``Person`` object, even if you did not call ``EntityManager#persist``:

以下代码将使用对 ``Person`` 对象所做的更改更新你的数据库，即使你未调用 ``EntityManager#persist``：

.. code-block:: php

    <?php
    $user = $entityManager->find("Person", 1);
    $user->setName("Guilherme");
    $entityManager->flush();

How Doctrine Detects Changes // Doctrine 如何侦测变更
------------------------------------------------------------

Doctrine is a data-mapper that tries to achieve persistence-ignorance (PI).
This means you map php objects into a relational database that don't
necessarily know about the database at all. A natural question would now be,
"how does Doctrine even detect objects have changed?". 

Doctrine 是一个数据映射器，它尝试达到透明的持久化（persistence-ignorance）。这意味着你映射
PHP 数组到一个关系数据库，可以不见得知道有关的数据库。现在自然有个问题，“Doctrine 如何侦测数据
已变更？”。

For this Doctrine keeps a second map inside the UnitOfWork. Whenever you fetch
an object from the database Doctrine will keep a copy of all the properties and
associations inside the UnitOfWork. Because variables in the PHP language are
subject to "copy-on-write" the memory usage of a PHP request that only reads
objects from the database is the same as if Doctrine did not keep this variable
copy. Only if you start changing variables PHP will create new variables internally
that consume new memory.

为此，Doctrine 保持了第二映射在 UnitOfWork 内部。一旦你从数据库取回一个对象，Doctrine 将保持所有
属性和关联的一个拷贝在 UnitOfWork 内部。因为变量在 PHP 语言中是受限于“写时拷贝（copy-on-write）”
一个仅从数据库读取对象的 PHP 请求的内存使用率与 Doctrine 不保持该变量拷贝是相同的。仅当你开始改变
变量，PHP 将内部创建新变量，这消耗新的内存。

Now whenever you call ``EntityManager#flush`` Doctrine will iterate over the
Identity Map and for each object compares the original property and association
values with the values that are currently set on the object. If changes are
detected then the object is queued for a SQL UPDATE operation. Only the fields
that actually changed are updated.

现在一旦你调用 ``EntityManager#flush`` Doctrine 将迭代身份映射且对每一个对象使用当前设置
在该对象的值与原始属性和关联的值比较。如果变更被侦测到，那么该对象被队列等待一个 SQL UPDATE 操作。
仅实际变更的字段被更新。

This process has an obvious performance impact. The larger the size of the
UnitOfWork is, the longer this computation takes. There are several ways to
optimize the performance of the Flush Operation:

这个过程有明显的性能影响。UnitOfWork 的大小越大，这个计算的耗费越久。
这里有几个方式来优化 Flush 操作的性能：

- Mark entities as read only. These entities can only be inserted or removed,
  but are never updated. They are omitted in the changeset calculation.
- 让实体只读。这些实体仅能被插入或移除，但是永不被更新。在变更集的计算中它们被忽略。
- Temporarily mark entities as read only. If you have a very large UnitOfWork
  but know that a large set of entities has not changed, just mark them as read
  only with ``$entityManager->getUnitOfWork()->markReadOnly($entity)``.
- 临时让实体只读。如果你有一个非常大的 UnitOfWork，但是知道它没有修改，就用 ``$entityManager->getUnitOfWork()->markReadOnly($entity)``
  标记它们为只读。
- Flush only a single entity with ``$entityManager->flush($entity)``.
- 使用 ``$entityManager->flush($entity)`` 仅 flush 单个实体。
- Use :doc:`Change Tracking Policies <change-tracking-policies>` to use more
  explicit strategies of notifying the UnitOfWork what objects/properties
  changed.
- 使用 :doc:`变更跟踪策略 <change-tracking-policies>` 来使用更明确的策略通知 UnitOfWork 那些对象/属性修改了。


Query Internals // 查询内部
-----------------------------------

The different ORM Layers // 不同的 ORM 层次
--------------------------------------------------

Doctrine ships with a set of layers with different responsibilities. This
section gives a short explanation of each layer.

Doctrine 附带了一套具有不同职责的层次。本节给出每一个层次简短说明。

Hydration
~~~~~~~~~

Responsible for creating a final result from a raw database statement and a
result-set mapping object. The developer can choose which kind of result he
wishes to be hydrated. Default result-types include:

负责从原始数据库语句和结果集映射对象创建一个最终结果。开发者可以选择它希望被 hydrated 的某种结果。
默认的结果类型包括：

- SQL to Entities
- SQL 至实体
- SQL to structured Arrays
- SQL 至结构化数组
- SQL to simple scalar result arrays
- SQL 至简单标量结果数组
- SQL to a single result variable
- SQL 至单个结果变量

Hydration to entities and arrays is one of most complex parts of Doctrine
algorithm-wise. It can build results with for example:

Hydration 实体和数组是Doctrine 聪明的算法最复杂的一部分。使用以下例子构建结果：

- Single table selects
- 单一表选择
- Joins with n:1 or 1:n cardinality, grouping belonging to the same parent.
- 用 n:1 or 1:n 基数联结，组合属于同一父。
- Mixed results of objects and scalar values
- 混合对象结果和标量值
- Hydration of results by a given scalar value as key.
- 通过给定标量值作为键 Hydration 结果。

Persisters
~~~~~~~~~~

tbr

UnitOfWork
~~~~~~~~~~

tbr

ResultSetMapping
~~~~~~~~~~~~~~~~

tbr

DQL Parser
~~~~~~~~~~

tbr

SQLWalker
~~~~~~~~~

tbr

EntityManager
~~~~~~~~~~~~~

tbr

ClassMetadataFactory
~~~~~~~~~~~~~~~~~~~~

tbr

