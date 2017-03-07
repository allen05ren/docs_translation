Frequently Asked Questions // 常见问题解答
==============================================

.. note::

    This FAQ is a work in progress. We will add lots of questions and not answer them right away just to remember
    what is often asked. If you stumble across an unanswered question please write a mail to the mailing-list or
    join the #doctrine channel on Freenode IRC.

    本 FAQ 是正在进行中的工作。我们将添加许多问题且不立刻回答它们只是记注什么问题经常被问。如果你偶然发现一个悬而未决的问题请写一封邮件到邮件列表
    或加入在 Freenode IRC上的 #doctrine 频道。

Database Schema // 数据库 Schema
-------------------------------------

How do I set the charset and collation for MySQL tables? // 如何为 MySQL 表设置字符集和排序规则？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can't set these values inside the annotations, yml or xml mapping files. To make a database
work with the default charset and collation you should configure MySQL to use it as default charset,
or create the database with charset and collation details. This way they get inherited to all newly
created database tables and columns.

你不能在注释、yml 或 xml 映射文件内部设置这些值。要使数据库使用一个默认的字符集和排序规则工作，你应当配置 MySQL 使用它们作为默认字符集，
或使用具体字符集和排序规则创建数据库。这种方式所有最近创建的数据库表和列得以继承它们。

Entity Classes // 实体类
-----------------------------

I access a variable and its null, what is wrong? 我访问一个变量，它是 null，怎么了？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If this variable is a public variable then you are violating one of the criteria for entities.
All properties have to be protected or private for the proxy object pattern to work.

如果这个变量是一个 public 变量，那么你正在违反一个实体的一个规范。所有的属性必须是 protected 或 private，代理对象模式才能工作。

How can I add default values to a column? // 如何为一个列添加默认值？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine does not support to set the default values in columns through the "DEFAULT" keyword in SQL.
This is not necessary however, you can just use your class properties as default values. These are then used
upon insert:

Doctrine 不支持在列中通过 SQL中的 "DEFAULT" 关键字设置默认值。然而这是不必要的，你可以使用你的类属性作为默认值。
使用时插入：

.. code-block:: php

    class User
    {
        const STATUS_DISABLED = 0;
        const STATUS_ENABLED = 1;

        private $algorithm = "sha1";
        private $status = self:STATUS_DISABLED;
    }

.

Mapping // 映射
--------------------

Why do I get exceptions about unique constraint failures during ``$em->flush()``? // 为何我在 ``$em->flush()`` 期间得到一个唯一限制失败的异常？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine does not check if you are re-adding entities with a primary key that already exists
or adding entities to a collection twice. You have to check for both conditions yourself
in the code before calling ``$em->flush()`` if you know that unique constraint failures
can occur.

Doctrine 不检查是否你正在重新添加引用已经存在的主键的实体或添加实体到集合两次。你必须在调用 ``$em->flush()`` 以前
自己在代码中检查这两个条件，如果你知道唯一限制失败可能发生。

In `Symfony2 <http://www.symfony.com>`_ for example there is a Unique Entity Validator
to achieve this task.

在 `Symfony2 <http://www.symfony.com>`_ 的例子中有一个唯一实体验证器可以完成这项任务。

For collections you can check with ``$collection->contains($entity)`` if an entity is already
part of this collection. For a FETCH=LAZY collection this will initialize the collection,
however for FETCH=EXTRA_LAZY this method will use SQL to determine if this entity is already
part of the collection.

对于集合你可以用 ``$collection->contains($entity)`` 检查，如果一个实体已经是这个集合的一部分。对于一个 FETCH=LAZY
的集合你将初始化这个集合，然而对于 FETCH=EXTRA_LAZY 此方法将使用 SQL 判断是否该实体已经是这个集合的一部分。

Associations // 关联
--------------------------

What is wrong when I get an InvalidArgumentException "A new entity was found through the relationship.."? // 得到一个 InvalidArgumentException "A new entity was found through the relationship.." 异常是什么错误？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This exception is thrown during ``EntityManager#flush()`` when there exists an object in the identity map
that contains a reference to an object that Doctrine does not know about. Say for example you grab
a "User"-entity from the database with a specific id and set a completely new object into one of the associations
of the User object. If you then call ``EntityManager#flush()`` without letting Doctrine know about
this new object using ``EntityManager#persist($newObject)`` you will see this exception.

当在身份映射中存在的一个对象，包含了一个 Doctrine 并不知道的一个对象的引用时在 ``EntityManager#flush()`` 期间会抛出这个异常。举例子说，
你使用一个特定的 id从数据库中抓取了一个“User”实体并且完全地设置了一个新对象到这个 User对象的其中一个关联中。如果你之后调用
``EntityManager#flush()`` 没有使用 ``EntityManager#persist($newObject)`` 让 Doctrine 知道这个新对象，你将看到这个异常。

You can solve this exception by:

你可以解决这个异常，通过：

* Calling ``EntityManager#persist($newObject)`` on the new object
* 在该新对象上调用 ``EntityManager#persist($newObject)``
* Using cascade=persist on the association that contains the new object
* 在包含该新对象的那个关联上使用 cascade=persist

How can I filter an association? // 如何过滤一个关联？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Natively you can't filter associations in 2.0 and 2.1. You should use DQL queries to query for the filtered set of entities.

在2.0和2.1版本中，原生地你不能过滤关联。你应该使用 DQL 查询来查询获得已过滤的实体集合。

I call clear() on a One-To-Many collection but the entities are not deleted // 我在一个 One-To-Many 上调用 clear()，但这些实体没有被删除
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is an expected behavior that has to do with the inverse/owning side handling of Doctrine.
By definition a One-To-Many association is on the inverse side, that means changes to it
will not be recognized by Doctrine.

这是一个预期的行为，必须使用 Doctrine 的 inverse/owning 侧处理。

If you want to perform the equivalent of the clear operation you have to iterate the
collection and set the owning side many-to-one reference to NULL as well to detach all entities
from the collection. This will trigger the appropriate UPDATE statements on the database.

如果你想执行该 clear 操作等价的操作你必须迭代这个集合并设置 owning 侧 many-to-one 引用至 NULL 就像从这个集合分离（detach）
所有实体那样。这将触发适当的 UPDATE 语句在数据库上。

How can I add columns to a many-to-many table? // 如何为一个 many-to-many 的表添加列？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The many-to-many association is only supporting foreign keys in the table definition
To work with many-to-many tables containing extra columns you have to use the
foreign keys as primary keys feature of Doctrine introduced in version 2.1.

many-to-many 关联仅在表中定义支持外键。要使用包含了额外列的 many-to-many 表，您必须使用外键作为主键特性，
Doctrine在2.1版本中引入该特性。

See :doc:`the tutorial on composite primary keys for more information<../tutorials/composite-primary-keys>`.

查看 :doc:`教程复合主键 <../tutorials/composite-primary-keys>` 获取更多信息。

How can i paginate fetch-joined collections? // 如何分页 fetch-joined 集合？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you are issuing a DQL statement that fetches a collection as well you cannot easily iterate
over this collection using a LIMIT statement (or vendor equivalent).

如果你正在发布一个 DQL 语句取回一个集合，就像你不能轻易地使用一个 LIMIT 语句（或供应商等效的）迭代这个集合一样。

Doctrine does not offer a solution for this out of the box but there are several extensions
that do:

Doctrine没有为此提供一个开箱即用的解决方案但是这里有几个扩展可以做到：

* `DoctrineExtensions <http://github.com/beberlei/DoctrineExtensions>`_
* `Pagerfanta <http://github.com/whiteoctober/pagerfanta>`_

Why does pagination not work correctly with fetch joins? // 为何使用 fetch joins 分页没有正确地执行？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Pagination in Doctrine uses a LIMIT clause (or vendor equivalent) to restrict the results.
However when fetch-joining this is not returning the correct number of results since joining
with a one-to-many or many-to-many association multiplies the number of rows by the number
of associated entities.

在 Doctrine 中分页使用一个 LIMIT 子句（或供应商等效的）来限定结果。然而当 fetch-joining 时这不返回正确数量的结果，
因为与 one-to-many or many-to-many 的关联 joining 将行数乘以关联实体的数量。

See the previous question for a solution to this task.

查看前面那个问题为此给出的解决方案。

Inheritance // 继承
------------------------

Can I use Inheritance with Doctrine 2? // 我可以使用 Doctrine 2 的继承吗？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
 
Yes, you can use Single- or Joined-Table Inheritance in Doctrine 2.

是的，在 Doctrine 2中，你可以使用 Single- 或 Joined-表继承。

See the documentation chapter on :doc:`inheritance mapping <inheritance-mapping>` for
the details.

更多详情请查看文档的相关章节 :doc:`继承映射 <inheritance-mapping>`。

Why does Doctrine not create proxy objects for my inheritance hierarchy? // 为何 Doctrine 不为我的继承层次结构创建代理对象？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you set a many-to-one or one-to-one association target-entity to any parent class of
an inheritance hierarchy Doctrine does not know what PHP class the foreign is actually of.
To find this out it has to execute a SQL query to look this information up in the database.

如果你设置一个 many-to-one 或 one-to-one 关联的目标实体（target-entity）至继承层次结构的任何父类，Doctrine
不能知道实际上的外部的 PHP 类是什么。要寻找这个，它必须执行一个 SQL 查询来查找这个信息在数据库中。

EntityGenerator // 实体生成器（EntityGenerator）
---------------------------------------------------

Why does the EntityGenerator not do X? // 为何 EntityGenerator 不能做 X？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The EntityGenerator is not a full fledged code-generator that solves all tasks. Code-Generation
is not a first-class priority in Doctrine 2 anymore (compared to Doctrine 1). The EntityGenerator
is supposed to kick-start you, but not towards 100%.

EntityGenerator 不是充分成熟为解决所有任务的代码生成器。在 Doctrine 2 中代码生成不再是第一类（first-class）优先（相比于 Doctrine 1）。
EntityGenerator 被设想为你提供最初的动力，而不是100%。

Why does the EntityGenerator not generate inheritance correctly? // 为何 EntityGenerator 不能正确地生成继承？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Just from the details of the discriminator map the EntityGenerator cannot guess the inheritance hierarchy.
This is why the generation of inherited entities does not fully work. You have to adjust some additional
code to get this one working correctly.

只是从辨别器映射的详情 EntityGenerator 不能猜测继承层次结构。这就是为何生成的继承实体不能充分地工作的原因。你必须调整一些额外代码
以让它正确地工作。

Performance // 性能
------------------------

Why is an extra SQL query executed every time I fetch an entity with a one-to-one relation? // 为何我每次用 one-to-one 关联取回实体有额外 SQL 查询被执行？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If Doctrine detects that you are fetching an inverse side one-to-one association
it has to execute an additional query to load this object, because it cannot know
if there is no such object (setting null) or if it should set a proxy and which id this proxy has.

如果 Doctrine 侦测到你正在取回一个 inverse 侧 one-to-one 关联，它必须执行一个额外的查询来载入这个对象，因为它不能知道
是否没有这样的对象（设置 null)或是否它应该设置一个代理且此代理有哪个 id。

To solve this problem currently a query has to be executed to find out this information.

目前解决此问题必须执行一个查询来找到这个信息。

Doctrine Query Language // Doctrine 查询语言
------------------------------------------------

What is DQL? // 什么是 DQL？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DQL stands for Doctrine Query Language, a query language that very much looks like SQL
but has some important benefits when using Doctrine:

DQL 代表 Doctrine 查询语言（ Doctrine Query Language），一个非常类似 SQL 的查询语言，但使用 Doctrine 时有几个重要的优势：

-  It uses class names and fields instead of tables and columns, separating concerns between backend and your object model.
-  它使用类名和字段替代表和列，分离后端和你的对象模型的关注点。
-  It utilizes the metadata defined to offer a range of shortcuts when writing. For example you do not have to specify the ON clause of joins, since Doctrine already knows about them.
-  它利用元数据定义提供一系列快捷方式当书写时。例如，你不必指定 join 的 ON 子句，因为 Doctrine 已经理解了这些。
-  It adds some functionality that is related to object management and transforms them into SQL.
-  它添加了一些对象管理相关的和转换它们为 SQL 的功能。

It also has some drawbacks of course:

当然，它也有一些劣势：

-  The syntax is slightly different to SQL so you have to learn and remember the differences.
-  语法稍微不同于 SQL，所以你必须学习和记住这些不同。
-  To be vendor independent it can only implement a subset of all the existing SQL dialects. Vendor specific functionality and optimizations cannot be used through DQL unless implemented by you explicitly.
-  为了独立于供应商（数据库供应商）它只能实现所有现有的 SQL 方言的一个子集。供应商特定的功能和优化不能通过 DQL 使用除非你显式实现。
-  For some DQL constructs subselects are used which are known to be slow in MySQL.
-  对于一些 DQL 构造 subselects 已知在 MySQL 中使用会比较慢。

Can I sort by a function (for example ORDER BY RAND()) in DQL? // 在 DQL 中，我可以通过一个函数（如 ORDER BY RAND()）排序吗？
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

No, it is not supported to sort by function in DQL. If you need this functionality you should either
use a native-query or come up with another solution. As a side note: Sorting with ORDER BY RAND() is painfully slow
starting with 1000 rows.

不，在 DQL 中不支持通过函数排序。如果你需要这个功能你应该使用原生查询或想出其他解决方案。
作为边注：使用 ORDER BY RAND() 排序是非常地慢，超过1000行时。

A Query fails, how can I debug it? // 查询失败，我如何 debug 它？
--------------------------------------------------------------------

First, if you are using the QueryBuilder you can use
``$queryBuilder->getDQL()`` to get the DQL string of this query. The
corresponding SQL you can get from the Query instance by calling
``$query->getSQL()``.

首先，如果你正在使用 QueryBuilder 你可以使用 ``$queryBuilder->getDQL()`` 来获得此查询的 DQL 字符串。
相应的 SQL 你可以从这个查询实例获得，通过调用 ``$query->getSQL()``。

.. code-block:: php

    <?php
    $dql = "SELECT u FROM User u";
    $query = $entityManager->createQuery($dql);
    var_dump($query->getSQL());

    $qb = $entityManager->createQueryBuilder();
    $qb->select('u')->from('User', 'u');
    var_dump($qb->getDQL());
