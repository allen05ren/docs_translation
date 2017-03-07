Architecture // 架构
==========================

This chapter gives an overview of the overall architecture,
terminology and constraints of Doctrine 2. It is recommended to
read this chapter carefully.

本章给出了 Doctrine 2 的架构、术语和限制的一个总体概览。建议仔细阅读本章。

Using an Object-Relational Mapper // 使用对象关系映射器
----------------------------------------------------------

As the term ORM already hints at, Doctrine 2 aims to simplify the
translation between database rows and the PHP object model. The
primary use case for Doctrine are therefore applications that
utilize the Object-Oriented Programming Paradigm. For applications
that do not primarily work with objects Doctrine 2 is not suited very
well.

正如术语 ORM 所暗示，Doctrine 2 目的在于简化数据库行和 PHP 对象模型之间的转换。因此，
Doctrine 主要的使用案例是利用面向对象编程模型的应用程序。对于不主要使用对象的应用程序
Doctrine 2 并不是很合适。

Requirements // 必要条件
-----------------------------

Doctrine 2 requires a minimum of PHP 5.4. For greatly improved
performance it is also recommended that you use APC with PHP.

Doctrine 2 需要最低 PHP 5.4。为了更好地改善性能，也推荐你使用 APC 与 PHP 一起。

Doctrine 2 Packages // Doctrine 2 包
------------------------------------------

Doctrine 2 is divided into three main packages.

Doctrine 2 被分割进三个主要的包。

-  Common
-  Common 包
-  DBAL (includes Common)
-  DBAL 包（包含 Common 包）
-  ORM (includes DBAL+Common)
-  ORM 包（包含 DBAL+Common 包）

This manual mainly covers the ORM package, sometimes touching parts
of the underlying DBAL and Common packages. The Doctrine code base
is split in to these packages for a few reasons and they are to...

本手册主要覆盖 ORM 包，有时接触部分底层的 DBAL 和 Common 包。Doctrine 代码库划分到
这些包有几个原因，它们是...

-  ...make things more maintainable and decoupled
-  ...让事情更可维护与解耦
-  ...allow you to use the code in Doctrine Common without the ORM
   or DBAL
-  ...允许你在 ORM 或 DBAL 之外使用 Doctrine Common中的代码
-  ...allow you to use the DBAL without the ORM
-  ...允许你在 ORM 之外使用 DBAL

The Common Package // Common 包
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Common package contains highly reusable components that have no
dependencies beyond the package itself (and PHP, of course). The
root namespace of the Common package is ``Doctrine\Common``.

Common 包包含高可复用组件，它没有依赖除了自身（和 PHP，当然）之外。Common 包的根
命名空间是 ``Doctrine\Common``。

The DBAL Package // DBAL 包
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The DBAL package contains an enhanced database abstraction layer on
top of PDO but is not strongly bound to PDO. The purpose of this
layer is to provide a single API that bridges most of the
differences between the different RDBMS vendors. The root namespace
of the DBAL package is ``Doctrine\DBAL``.

DBAL 包包含一个在 PDO 之上的增强的数据库抽象层，但它不是强烈地绑定到 PDO。此抽象层的用途是
提供一个单一的 API 弥补大多数不同 RDBMS 厂商之间的差异。DBAL 包的根命名空间是
``Doctrine\DBAL``。

The ORM Package // ORM 包
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ORM package contains the object-relational mapping toolkit that
provides transparent relational persistence for plain PHP objects.
The root namespace of the ORM package is ``Doctrine\ORM``.

ORM 包包含对象关系映射工具包，它为纯 PHP 对象提供透明的关系的持久化。ORM 包的根命名空间是
``Doctrine\ORM``。

Terminology // 术语
-------------------------

Entities // 实体
~~~~~~~~~~~~~~~~~~~~~

An entity is a lightweight, persistent domain object. An entity can
be any regular PHP class observing the following restrictions:

实体是一个轻量的、持久化的领域对象。实体可以是任何遵循以下限制的常规 PHP 类：

-  An entity class must not be final or contain final methods.
-  实体类必须不能是 final 或包含 final 方法。
-  All persistent properties/field of any entity class should
   always be private or protected, otherwise lazy-loading might not
   work as expected. In case you serialize entities (for example Session)
   properties should be protected (See Serialize section below).
-  任何实体类的所有持久化的属性/字段应该总是 private 或 protected，否则懒加载可能
   不按预期的运行。以防万一你序列化实体（如 SESSION）属性应该被保护（查看下面序列化部分）
-  An entity class must not implement ``__clone`` or
   :doc:`do so safely <../cookbook/implementing-wakeup-or-clone>`.
-  实体类不能实现 ``__clone`` 或 :doc:`do so safely <../cookbook/implementing-wakeup-or-clone>`.
-  An entity class must not implement ``__wakeup`` or
   :doc:`do so safely <../cookbook/implementing-wakeup-or-clone>`.
   Also consider implementing
   `Serializable <http://php.net/manual/en/class.serializable.php>`_
   instead.
-  实体类不能实现 ``__wakeup`` 或 :doc:`do so safely <../cookbook/implementing-wakeup-or-clone>`。
   相反考虑实现 `Serializable <http://php.net/manual/en/class.serializable.php>`_。
-  Any two entity classes in a class hierarchy that inherit
   directly or indirectly from one another must not have a mapped
   property with the same name. That is, if B inherits from A then B
   must not have a mapped field with the same name as an already
   mapped field that is inherited from A.
-  在类层次结构中直接地或间接地相互继承的两个实体类不能有一个具有相同名称的映射属性，即如果 B 继承自 A那么 B 不能有
   继承自 A 的已经映射的字段具有相同名称的映射的字段。
-  An entity cannot make use of func_get_args() to implement variable parameters.
   Generated proxies do not support this for performance reasons and your code might
   actually fail to work when violating this restriction.
-  实体不能使用 func_get_args() 实现可变参数。生成的代理不支持它是出于性能的原因且当违反此限制时你的代码实际上无法运行。

Entities support inheritance, polymorphic associations, and
polymorphic queries. Both abstract and concrete classes can be
entities. Entities may extend non-entity classes as well as entity
classes, and non-entity classes may extend entity classes.

实体支持继承、多态关联和多态查询和。抽象的和具体的类都可以是实体。实体可能扩展非实体类和实体类，
且非实体类可能扩展实体类。

.. note::

    The constructor of an entity is only ever invoked when
    *you* construct a new instance with the *new* keyword. Doctrine
    never calls entity constructors, thus you are free to use them as
    you wish and even have it require arguments of any type.

    实体的构造器永远仅当 *你* 使用 *new* 关键字构造一个新实例时被调用。Doctrine 永远不调用
    实体的构造器，因此你可以自由使用它们按照你希望的，且甚至让它要求任何类型的参数。


Entity states // 实体状态
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An entity instance can be characterized as being NEW, MANAGED,
DETACHED or REMOVED.

实体实例可以被描述为 NEW、MANAGED、DETACHED、REMOVED。

-  A NEW entity instance has no persistent identity, and is not yet
   associated with an EntityManager and a UnitOfWork (i.e. those just
   created with the "new" operator).
-  NEW 实体实例没有持久化的身份（persistent identity），且也不与 EntityManager 和
   UnitOfWork 关联（如那些刚刚用"new"操作符创建的）。
-  A MANAGED entity instance is an instance with a persistent
   identity that is associated with an EntityManager and whose
   persistence is thus managed.
-  MANAGED 实体实例是带有一个持久化的身份（persistent identity）的实例，
   该持久化的身份是与 EntityManager 关联的，因此其的持久化是被托管（managed）的。
-  A DETACHED entity instance is an instance with a persistent
   identity that is not (or no longer) associated with an
   EntityManager and a UnitOfWork.
-  DETACHED 实体实例是带有一个持久化的身份（persistent identity）的实例，
   该持久化身份是不（或不再）与 EntityManager 关联的。
-  A REMOVED entity instance is an instance with a persistent
   identity, associated with an EntityManager, that will be removed
   from the database upon transaction commit.
-  REMOVED 实体实例是带有一个持久化的身份（persistent identity），与 EntityManager 关联的的实例，
   该实体将从数据库的事务提交中被移除。

.. _architecture_persistent_fields:

Persistent fields // 持久化的字段
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The persistent state of an entity is represented by instance
variables. An instance variable must be directly accessed only from
within the methods of the entity by the entity instance itself.
Instance variables must not be accessed by clients of the entity.
The state of the entity is available to clients only through the
entity’s methods, i.e. accessor methods (getter/setter methods) or
other business methods.

实体的持久化的状态是通过实例变量被表示。实体变量必须仅通过该实体实例自身的内置方法
被直接地访问。

Collection-valued persistent fields and properties must be defined
in terms of the ``Doctrine\Common\Collections\Collection``
interface. The collection implementation type may be used by the
application to initialize fields or properties before the entity is
made persistent. Once the entity becomes managed (or detached),
subsequent access must be through the interface type.

持久化的字段和和属性的值集合（Collection-valued）必须依据 ``Doctrine\Common\Collections\Collection``
接口来定义。应用程序可以使用集合实现类型在实体被持久化前初始化字段或属性。一旦
实体变成 MANAGED（或 DETACHED），随后的访问必须通过该接口类型。

Serializing entities // 序列化实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Serializing entities can be problematic and is not really
recommended, at least not as long as an entity instance still holds
references to proxy objects or is still managed by an
EntityManager. If you intend to serialize (and unserialize) entity
instances that still hold references to proxy objects you may run
into problems with private properties because of technical
limitations. Proxy objects implement ``__sleep`` and it is not
possible for ``__sleep`` to return names of private properties in
parent classes. On the other hand it is not a solution for proxy
objects to implement ``Serializable`` because Serializable does not
work well with any potential cyclic object references (at least we
did not find a way yet, if you did, please contact us).

序列化实体可能有问题并且实际上不推荐，至少只要实体实例仍持有代理对象的引用或仍由一个
EntityManager 管理就不推荐序列化。如果你打算序列化（和反序列化）仍持有代理对象的引用的实体实例，
你可能遇到 private 属性的问题，由于技术上的限制。代理对象实现了 ``__sleep`` 魔术方法，
且 ``__sleep`` 返回父类中 private 属性名称是不可能的。另一方面，使代理对象实现
``Serializable`` 不是解决方案，因为 Serializable 不能很好地与任何潜在的循环对象引用工作
（至少我们还没有找到一个方法，如果你做到了，请联系我们）。

The EntityManager // 实体管理器（EntityManager）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``EntityManager`` class is a central access point to the ORM
functionality provided by Doctrine 2. The ``EntityManager`` API is
used to manage the persistence of your objects and to query for
persistent objects.

``EntityManager`` 类是一个由 Doctrine 2 提供的 ORM 功能的中心访问点。
``EntityManager`` API 被用于管理你的对象持久化和查询持久化的对象。

Transactional write-behind // 事务后写
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An ``EntityManager`` and the underlying ``UnitOfWork`` employ a
strategy called "transactional write-behind" that delays the
execution of SQL statements in order to execute them in the most
efficient way and to execute them at the end of a transaction so
that all write locks are quickly released. You should see Doctrine
as a tool to synchronize your in-memory objects with the database
in well defined units of work. Work with your objects and modify
them as usual and when you're done call ``EntityManager#flush()``
to make your changes persistent.

``EntityManager`` 和其底层 ``UnitOfWork`` 使用一个称为“事务后写”的策略，该
策略延迟 SQL 语句的执行以便以最有效的方式执行它们并且在事务的最后才执行它们，因此所有的
写入锁很快被释放。你应该把 Doctrine 看作是一种工具，在定义好的 UnitOfWork 中
同步你的内存的对象和数据库。使用你的对象且如往常一样修改它们，当你完成后，调用
``EntityManager#flush()`` 以使你的变更得以持久化。

The Unit of Work // UnitOfWork
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Internally an ``EntityManager`` uses a ``UnitOfWork``, which is a
typical implementation of the
`Unit of Work pattern <http://martinfowler.com/eaaCatalog/unitOfWork.html>`_,
to keep track of all the things that need to be done the next time
``flush`` is invoked. You usually do not directly interact with a
``UnitOfWork`` but with the ``EntityManager`` instead.

``EntityManager`` 内部使用 ``UnitOfWork``，``UnitOfWork`` 是
`Unit of Work pattern <http://martinfowler.com/eaaCatalog/unitOfWork.html>`_
的一个典型实现，用于保持跟踪下回调用 ``flush`` 时所需完成的所有事情。你通常不直接与
``UnitOfWork`` 打交道，而是 ``EntityManager``。
