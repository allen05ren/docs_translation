Working with Objects // 使用对象
=======================================

In this chapter we will help you understand the ``EntityManager``
and the ``UnitOfWork``. A Unit of Work is similar to an
object-level transaction. A new Unit of Work is implicitly started
when an EntityManager is initially created or after
``EntityManager#flush()`` has been invoked. A Unit of Work is
committed (and a new one started) by invoking
``EntityManager#flush()``.

本章我们将帮助你理解 ``EntityManager`` 和 ``UnitOfWork``。
一个 UnitOfWork 类似于一个对象级别的事务。
一个新的 UnitOfWork 是隐式开启的，当 EntityManager 最初被创建时或 ``EntityManager#flush()``
被调用后。通过调用 ``EntityManager#flush()`` 来提交一个 UnitOfWork（和开启一个新的）。

A Unit of Work can be manually closed by calling
EntityManager#close(). Any changes to objects within this Unit of
Work that have not yet been persisted are lost.

通过调用 EntityManager#close() 能够手动关闭 UnitOfWork。
在 UnitOfWork 中对对象的任何更改在未被持久化时都会丢失。

.. note::

    It is very important to understand that only
    ``EntityManager#flush()`` ever causes write operations against the
    database to be executed. Any other methods such as
    ``EntityManager#persist($entity)`` or
    ``EntityManager#remove($entity)`` only notify the UnitOfWork to
    perform these operations during flush.

    对于理解只有 ``EntityManager#flush()`` 后才引发数据库写入操作被执行是非常重要的。
    任何其他诸如 ``EntityManager#persist($entity)`` 或 ``EntityManager#remove($entity)``
    方法仅是通知 UnitOfWork 在刷新期间执行这些操作。

    Not calling ``EntityManager#flush()`` will lead to all changes
    during that request being lost.

    没有调用 ``EntityManager#flush()`` 将导致该请求期间的所有更改丢失。


Entities and the Identity Map // 实体和身份映射
------------------------------------------------------

Entities are objects with identity. Their identity has a conceptual
meaning inside your domain. In a CMS application each article has a
unique id. You can uniquely identify each article by that id.

实体是具有身份的对象。在领域内部身份具有概念上的意义。在一个 CMS（内容发布系统）应用中每篇文章有一个唯一 id。
你能通过该 id 唯一地识别每一篇文章。

Take the following example, where you find an article with the
headline "Hello World" with the ID 1234:

举一个下面例子，查找一篇带有标题“Hello World”和 ID 为1234的文章：

.. code-block:: php

    <?php
    $article = $entityManager->find('CMS\Article', 1234);
    $article->setHeadline('Hello World dude!');
    
    $article2 = $entityManager->find('CMS\Article', 1234);
    echo $article2->getHeadline();

In this case the Article is accessed from the entity manager twice,
but modified in between. Doctrine 2 realizes this and will only
ever give you access to one instance of the Article with ID 1234,
no matter how often do you retrieve it from the EntityManager and
even no matter what kind of Query method you are using (find,
Repository Finder or DQL). This is called "Identity Map" pattern,
which means Doctrine keeps a map of each entity and ids that have
been retrieved per PHP request and keeps returning you the same
instances.

本例中，从 EntityManager 访问了 Article 两次，但是做了修改。Doctrine 2
理解这个，并将只提供一个 ID 为1234的 Article 实例，不论你多经常从 EntityManager 取回它，
甚至不论你使用哪种查询方法（find、Repository Finder 或 DQL）。
这被称为“身份映射（Identity Map）”模式，它意味着 Doctrine 保存了一张每个实体和从每个 PHP 请求取得的身份（ids）的映射，
并且总是返回给你同样的实例。

In the previous example the echo prints "Hello World dude!" to the
screen. You can even verify that ``$article`` and ``$article2`` are
indeed pointing to the same instance by running the following
code:

上述例子会输出“Hello World dude!”到屏幕。你甚至能够通过运行以下代码验证那个 ``$article`` 和 ``$article2``
实际上指向了同一个实例：

.. code-block:: php

    <?php
    if ($article === $article2) {
        echo "Yes we are the same!";
    }

Sometimes you want to clear the identity map of an EntityManager to
start over. We use this regularly in our unit-tests to enforce
loading objects from the database again instead of serving them
from the identity map. You can call ``EntityManager#clear()`` to
achieve this result.

有时你想要清除 EntityManager 的身份映射至初始状态。我们经常地使用这个在我们的单元测试中以强制从
数据库载入对象而不是从该身份映射。你可以调用 ``EntityManager#clear()`` 来达到这个结果。

Entity Object Graph Traversal // 实体对象图遍历
-----------------------------------------------------

Although Doctrine allows for a complete separation of your domain
model (Entity classes) there will never be a situation where
objects are "missing" when traversing associations. You can walk
all the associations inside your entity models as deep as you
want.

尽管 Doctrine 允许你的领域模型（实体类）完全隔离，但在遍历关联时，永远不会有对象丢失的情景。
如果你想的话，你可以尽可能深入在你的实体模型内部的所有关联。

Take the following example of a single ``Article`` entity fetched
from newly opened EntityManager.

下面举一个简单的例子，从最近打开的 EntityManager 取回 ``Article`` 实体。

.. code-block:: php

    <?php
    /** @Entity */
    class Article
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;
    
        /** @Column(type="string") */
        private $headline;
    
        /** @ManyToOne(targetEntity="User") */
        private $author;
    
        /** @OneToMany(targetEntity="Comment", mappedBy="article") */
        private $comments;
    
        public function __construct()
        {
            $this->comments = new ArrayCollection();
        }
    
        public function getAuthor() { return $this->author; }
        public function getComments() { return $this->comments; }
    }
    
    $article = $em->find('Article', 1);

This code only retrieves the ``Article`` instance with id 1 executing
a single SELECT statement against the articles table in the database.
You can still access the associated properties author and comments
and the associated objects they contain.

上述代码取回 id 为1 的 ``Article`` 实例，仅在数据库中对 articles 表执行了一句 SELECT 语句。
你仍然能够访问已关联的属性 author 和 comments，以及它包含的已关联对象。

This works by utilizing the lazy loading pattern. Instead of
passing you back a real Author instance and a collection of
comments Doctrine will create proxy instances for you. Only if you
access these proxies for the first time they will go through the
EntityManager and load their state from the database.

它通过利用懒加载模式工作。Doctrine 将为你创建代理实例来替代你传递真实的 Author 实例和 comments 集合。
只有你第一次访问这些代理时，它们才会通过 EntityManager 来从数据库载入它们的状态。

This lazy-loading process happens behind the scenes, hidden from
your code. See the following code:

懒加载的过程发生在那个情景之后，从你的代码中隐藏了。查看以下代码：

.. code-block:: php

    <?php
    $article = $em->find('Article', 1);
    
    // accessing a method of the user instance triggers the lazy-load
    // 访问 user 实例的一个方法触发了懒加载
    echo "Author: " . $article->getAuthor()->getName() . "\n";
    
    // Lazy Loading Proxies pass instanceof tests:
    // 懒加载代理通过 instanceof 测试
    if ($article->getAuthor() instanceof User) {
        // a User Proxy is a generated "UserProxy" class
        // 一个 User 代理时一个已生成的“UserProxy”类
    }
    
    // accessing the comments as an iterator triggers the lazy-load
    // 作为一个迭代器访问评论以触发懒加载
    // retrieving ALL the comments of this article from the database
    // 从数据库中取回该文章的所有评论
    // using a single SELECT statement
    // 使用一句简单的 SELECT 语句
    foreach ($article->getComments() as $comment) {
        echo $comment->getText() . "\n\n";
    }
    
    // Article::$comments passes instanceof tests for the Collection interface
    // Article::$comments 通过 instanceof 测试为 Collection 接口
    // But it will NOT pass for the ArrayCollection interface
    // 但它将不会通过 ArrayCollection 接口的测试
    if ($article->getComments() instanceof \Doctrine\Common\Collections\Collection) {
        echo "This will always be true!";
    }

A slice of the generated proxy classes code looks like the
following piece of code. A real proxy class override ALL public
methods along the lines of the ``getName()`` method shown below:

生成的代理类代码片断看起来像下面的代码块。
以下展示了一个真实的代理类重载了沿着 ``getName()`` 行的所有 public 方法：

.. code-block:: php

    <?php
    class UserProxy extends User implements Proxy
    {
        private function _load()
        {
            // lazy loading code
        }
    
        public function getName()
        {
            $this->_load();
            return parent::getName();
        }
        // .. other public methods of User
    }

.. warning::

    Traversing the object graph for parts that are lazy-loaded will
    easily trigger lots of SQL queries and will perform badly if used
    to heavily. Make sure to use DQL to fetch-join all the parts of the
    object-graph that you need as efficiently as possible.

    遍历对象图被懒加载的部分将轻易地触发大量的 SQL 查询，并且如果过度使用会表现不佳。
    确保使用 DQL 来尽可能高效地取回联结（fetch-join）所有的对象图部分。


Persisting entities // 持久化实体
----------------------------------------

An entity can be made persistent by passing it to the
``EntityManager#persist($entity)`` method. By applying the persist
operation on some entity, that entity becomes MANAGED, which means
that its persistence is from now on managed by an EntityManager. As
a result the persistent state of such an entity will subsequently
be properly synchronized with the database when
``EntityManager#flush()`` is invoked.

一个实体可以通过将它传递给 ``EntityManager#persist($entity)`` 方法来成为持久化的。
通过在一些实体上应用该持久操作，该实体变成 MANAGED，这意味着它的持久性从现在开始通过
EntityManager 来托管。当 ``EntityManager#flush()`` 被调用时，将导致实体的持久化的状态
随之被正确地与数据库同步。

.. note::

    Invoking the ``persist`` method on an entity does NOT
    cause an immediate SQL INSERT to be issued on the database.
    Doctrine applies a strategy called "transactional write-behind",
    which means that it will delay most SQL commands until
    ``EntityManager#flush()`` is invoked which will then issue all
    necessary SQL statements to synchronize your objects with the
    database in the most efficient way and a single, short transaction,
    taking care of maintaining referential integrity.

    在实体上调用 ``persist`` 方法不能引发一个立刻的 SQL INSERT 被发布在数据库上。
    Doctrine 应用了一个被称作“事务后写”的策略，这意味着它将延迟大多数 SQL 命令直到
    ``EntityManager#flush()`` 被调用，将之后发布所有必须的 SQL 语句以同步你的对象与数据库，
    用最有效的且单个、短事务、注重维护引用完整性的方式。


Example:

例子：

.. code-block:: php

    <?php
    $user = new User;
    $user->setName('Mr.Right');
    $em->persist($user);
    $em->flush();

.. note::

    Generated entity identifiers / primary keys are
    guaranteed to be available after the next successful flush
    operation that involves the entity in question. You can not rely on
    a generated identifier to be available directly after invoking
    ``persist``. The inverse is also true. You can not rely on a
    generated identifier being not available after a failed flush
    operation.

    生成的实体标识符/主键被保障在涉及讨论中的实体下一次成功的 flush 操作后是可用的。
    调用``persist``后，你不能直接依靠一个已生成标识符是可用的。反过来也是如此。
    在一个失败的刷新操作之后，你不能依靠一个已生成标识符是不可用的


The semantics of the persist operation, applied on an entity X, are
as follows:

在一个实体 X 上应用 persist 操作的含义，如下：

-  If X is a new entity, it becomes managed. The entity X will be
   entered into the database as a result of the flush operation.
-  如果 X 是一个新的实体，它变成 managed。flush 操作的结果是该实体 X 将被送进数据库中。
-  If X is a preexisting managed entity, it is ignored by the
   persist operation. However, the persist operation is cascaded to
   entities referenced by X, if the relationships from X to these
   other entities are mapped with cascade=PERSIST or cascade=ALL (see
   ":ref:`Transitive Persistence <transitive-persistence>`").
-  如果 X 是预先存在的 managed 实体，persist 操作会被忽略。然而，如果从 X 至那些其他实体的关联使用
   cascade=PERSIST 或 cascade=ALL (查看 ":ref:`传播持久化 <transitive-persistence>`")被映射，
   persist 操作被级联到 X 引用的那些实体。
-  If X is a removed entity, it becomes managed.
-  如果 X 是一个 removed 实体，它变成 managed。
-  If X is a detached entity, an exception will be thrown on
   flush.
-  如果 X 是一个 detached 实体，在 flush 时一个异常将被抛出。

Removing entities // 移除实体
------------------------------------

An entity can be removed from persistent storage by passing it to
the ``EntityManager#remove($entity)`` method. By applying the
``remove`` operation on some entity, that entity becomes REMOVED,
which means that its persistent state will be deleted once
``EntityManager#flush()`` is invoked.

一个实体可以通过将它传递给 ``EntityManager#remove($entity)`` 方法从持久化的存储中移除。
通过在一些实体上应用该 ``remove`` 操作，该实体变成 REMOVED，这意味着一旦 ``EntityManager#flush()``
被调用，它的持久化的状态将被删除。

.. note::

    Just like ``persist``, invoking ``remove`` on an entity
    does NOT cause an immediate SQL DELETE to be issued on the
    database. The entity will be deleted on the next invocation of
    ``EntityManager#flush()`` that involves that entity. This
    means that entities scheduled for removal can still be queried
    for and appear in query and collection results. See
    the section on :ref:`Database and UnitOfWork Out-Of-Sync <workingobjects_database_uow_outofsync>`
    for more information.
    
    类似 ``persist``，在实体上调用 ``remove`` 不会引起立刻的 SQL DELETE 被发布在数据库上。
    在涉及的那个实体上，下一次调用 ``EntityManager#flush()`` 该实体将会被删除。
    这意味着那些列入移除计划的实体仍然能被查询并出现在查询和集合结果中。查看该部分详细信息 :ref:`数据库 与 UnitOfWork Out-Of-Sync <workingobjects_database_uow_outofsync>`。

Example:

例子：

.. code-block:: php

    <?php
    $em->remove($user);
    $em->flush();

The semantics of the remove operation, applied to an entity X are
as follows:

在一个实体 X 上应用 remove 操作的含义，如下：

-  If X is a new entity, it is ignored by the remove operation.
   However, the remove operation is cascaded to entities referenced by
   X, if the relationship from X to these other entities is mapped
   with cascade=REMOVE or cascade=ALL (see ":ref:`Transitive Persistence <transitive-persistence>`").
-  如果 X 是一个新的实体，remove 操作会被忽略。然而，如果从 X 至那些其他实体的关联使用
   cascade=REMOVE 或 cascade=ALL (查看 ":ref:`传播持久化 <transitive-persistence>`")被映射，
   remove 操作被级联到 X 引用的那些实体。
-  If X is a managed entity, the remove operation causes it to
   become removed. The remove operation is cascaded to entities
   referenced by X, if the relationships from X to these other
   entities is mapped with cascade=REMOVE or cascade=ALL (see
   ":ref:`Transitive Persistence <transitive-persistence>`").
-  如果 X 是一个 managed 实体，remove 操作导致它变为 removed。如果从 X 至那些其他实体的关联使用
   cascade=REMOVE 或 cascade=ALL (查看 ":ref:`传播持久化 <transitive-persistence>`")被映射，
   remove 操作被级联到 X 引用的那些实体。
-  If X is a detached entity, an InvalidArgumentException will be
   thrown.
-  如果 X 是一个 detached 实体，一个 InvalidArgumentException 异常将被抛出。
-  If X is a removed entity, it is ignored by the remove operation.
-  如果 X 是一个 removed 实体，remove 操作会被忽略。
-  A removed entity X will be removed from the database as a result
   of the flush operation.
-  刷新操作的结果是一个 removed 实体 X 将从数据库被移除。

After an entity has been removed its in-memory state is the same as
before the removal, except for generated identifiers.

实体已经被移除后，它在内存中的状态和移除前一样，除了生成的标识符。


Removing an entity will also automatically delete any existing
records in many-to-many join tables that link this entity. The
action taken depends on the value of the ``@joinColumn`` mapping
attribute "onDelete". Either Doctrine issues a dedicated ``DELETE``
statement for records of each join table or it depends on the
foreign key semantics of onDelete="CASCADE".

移除一个实体将同时自动删除在 many-to-many 中链接至该实体的联结表的任何已存在记录。
该操作依赖于 ``@joinColumn`` 映射属性 "onDelete" 的值。Doctrine 发布了一个专门的
``DELETE`` 语句以记录每一张联结表或者它依赖外键的 onDelete="CASCADE" 含义。

Deleting an object with all its associated objects can be achieved
in multiple ways with very different performance impacts.

删除对象及其关联的对象可以被实现在多种具有不同性能影响的方式中。

1. If an association is marked as ``CASCADE=REMOVE`` Doctrine 2
   will fetch this association. If its a Single association it will
   pass this entity to ``EntityManager#remove()``. If the association
   is a collection, Doctrine will loop over all its elements and
   pass them to``EntityManager#remove()``.
   In both cases the cascade remove semantics are applied recursively.
   For large object graphs this removal strategy can be very costly.
   如果一个关联被标记为 ``CASCADE=REMOVE`` ，Doctrine 2将取回此关联。
   如果它是个简单关联，会将该实体传递给 ``EntityManager#remove()``。如果该关联是一个集合，
   Doctrine 将循环它的所有元素并传递给 ``EntityManager#remove()``。
   在这两种情况下，级联删除含义被递归应用。对于大对象图这种移除策略开销很大。
2. Using a DQL ``DELETE`` statement allows you to delete multiple
   entities of a type with a single command and without hydrating
   these entities. This can be very efficient to delete large object
   graphs from the database.
   使用 DQL ``DELETE`` 语句允许你删除一个类型的多数实体，使用一个简单的命令并且无 hydrating 这些实体。
   这能高效地从数据库中删除大对象图。
3. Using foreign key semantics ``onDelete="CASCADE"`` can force the
   database to remove all associated objects internally. This strategy
   is a bit tricky to get right but can be very powerful and fast. You
   should be aware however that using strategy 1 (``CASCADE=REMOVE``)
   completely by-passes any foreign key ``onDelete=CASCADE`` option,
   because Doctrine will fetch and remove all associated entities
   explicitly nevertheless.
   使用外键 ``onDelete="CASCADE"`` 含义能强制数据库移除所有相关联内部对象。
   这种策略有点“狡猾”但很强大和高效。你应该意识到使用策略1（``CASCADE=REMOVE``）完全绕过了任何外键 ``onDelete=CASCADE`` 选项，
   因为 Doctrine 将再明确不过地取回和删除所有相关联实体。

Detaching entities // 分离实体
-------------------------------------

An entity is detached from an EntityManager and thus no longer
managed by invoking the ``EntityManager#detach($entity)`` method on
it or by cascading the detach operation to it. Changes made to the
detached entity, if any (including removal of the entity), will not
be synchronized to the database after the entity has been
detached.

一个实体通过在其上调用 ``EntityManager#detach($entity)`` 方法或通过级联的 detach 操作从 EntityManager 分离，
并因此不再 managed。实体被分离后，对 detached 实体的任何更改（包括移除它）将不会被同步至数据库。

Doctrine will not hold on to any references to a detached entity.

Doctrine 将不会保存对一个 detached 实体的任何引用。

Example:

例子：

.. code-block:: php

    <?php
    $em->detach($entity);

The semantics of the detach operation, applied to an entity X are
as follows:

在一个实体 X 上应用 detach 操作的含义，如下：

-  If X is a managed entity, the detach operation causes it to
   become detached. The detach operation is cascaded to entities
   referenced by X, if the relationships from X to these other
   entities is mapped with cascade=DETACH or cascade=ALL (see
   ":ref:`Transitive Persistence <transitive-persistence>`"). Entities which previously referenced X
   will continue to reference X.
-  如果 X 是一个 managed 实体，detach 操作导致它变为 detached。
   如果从 X 至那些其他实体的关联使用 cascade=DETACH or cascade=ALL (see
   ":ref:`传播持久化 <transitive-persistence>`") 被映射，
   detach 操作被级联到 X 引用的那些实体。之前引用了 X 的实体将继续引用 X。
-  If X is a new or detached entity, it is ignored by the detach
   operation.
-  如果 X 是一个 new 或 detached 实体，detach 操作会被忽略。
-  If X is a removed entity, the detach operation is cascaded to
   entities referenced by X, if the relationships from X to these
   other entities is mapped with cascade=DETACH or cascade=ALL (see
   ":ref:`Transitive Persistence <transitive-persistence>`"). Entities which previously referenced X
   will continue to reference X.
-  如果 X 是一个 removed 实体，如果从 X 至那些其他实体的关联使用 cascade=DETACH or cascade=ALL (see
   ":ref:`传播持久化 <transitive-persistence>`") 被映射，
   detach 操作被级联到 X 引用的那些实体。之前引用了 X 的实体将继续引用 X。

There are several situations in which an entity is detached
automatically without invoking the ``detach`` method:

这里有几个情景一个实体会被自动 detached 而不需要调用 ``detach`` 方法：

-  When ``EntityManager#clear()`` is invoked, all entities that are
   currently managed by the EntityManager instance become detached.
-  当 ``EntityManager#clear()`` 被调用，通过该 EntityManager 实例托管的当前所有实体变成 detached。
-  When serializing an entity. The entity retrieved upon subsequent
   unserialization will be detached (This is the case for all entities
   that are serialized and stored in some cache, i.e. when using the
   Query Result Cache).
-  当序列化一个实体时。随后反序列化取回该实体，该实体将 detached（对于所有实体被序列化并存储在一些缓存中，存在此情况，
   比如当使用查询结果缓存时）。

The ``detach`` operation is usually not as frequently needed and
used as ``persist`` and ``remove``.

``detach``  操作通常不是经常需要相对于使用 ``persist`` 和 ``remove``。

Merging entities // 合并实体
-----------------------------------

Merging entities refers to the merging of (usually detached)
entities into the context of an EntityManager so that they become
managed again. To merge the state of an entity into an
EntityManager use the ``EntityManager#merge($entity)`` method. The
state of the passed entity will be merged into a managed copy of
this entity and this copy will subsequently be returned.

合并实体是指实体（通常 detached）合并入 EntityManager 上下文中，因此它们又变成 managed。
使用 ``EntityManager#merge($entity)`` 方法以 merge 一个实体的状态至 EntityManager。
传递的实体的状态将被合并入该实体的一个 managed 拷贝且该拷贝随后被返回。

Example:

例子：

.. code-block:: php

    <?php
    $detachedEntity = unserialize($serializedEntity); // some detached entity // 一些 detached 的实体
    $entity = $em->merge($detachedEntity);
    // $entity now refers to the fully managed copy returned by the merge operation.
    // $entity 现在是指通过合并操作返回的完全地 managed 拷贝。
    // The EntityManager $em now manages the persistence of $entity as usual.
    // EntityManager $em 现在可以像往常一样管理 $entity 的持久化。

.. note::

    When you want to serialize/unserialize entities you
    have to make all entity properties protected, never private. The
    reason for this is, if you serialize a class that was a proxy
    instance before, the private variables won't be serialized and a
    PHP Notice is thrown.

    当你想要序列化/反序列化实体时，你不得不让实体的所有属性是受保护的，永远不要是私有的。
    之所以这样的原因是，如果你序列化的类之前是一个代理实例，私有变量不能被序列化且会抛出一个 PHP Notice。

The semantics of the merge operation, applied to an entity X, are
as follows:

在一个实体 X 应用 merge 操作的含义，如下：

-  If X is a detached entity, the state of X is copied onto a
   pre-existing managed entity instance X' of the same identity.
-  如果 X 是 一个 detached 实体，X 的状态被拷贝至一个预先存在的拥有相同身份的 managed 实体实例 X’。
-  If X is a new entity instance, a new managed copy X' will be
   created and the state of X is copied onto this managed instance.
-  如果 X 是一个 new 实体实例，一个 new managed 拷贝 X‘ 将被创建并且 X 的状态被拷贝至该 managed 实例。
-  If X is a removed entity instance, an InvalidArgumentException
   will be thrown.
-  如果 X 是一个 removed 实体实例，一个 InvalidArgumentException 异常将被抛出。
-  If X is a managed entity, it is ignored by the merge operation,
   however, the merge operation is cascaded to entities referenced by
   relationships from X if these relationships have been mapped with
   the cascade element value MERGE or ALL (see ":ref:`Transitive Persistence <transitive-persistence>`").
-  如果 X 是一个 managed 实体，merge 操作将被忽略，然而，假如这些关联使用级联元素值 MERGE or ALL 
   （查看 ":ref:`传播持久化 <transitive-persistence>`"），merge 操作被级联到由 X 引用的关联。
-  For all entities Y referenced by relationships from X having the
   cascade element value MERGE or ALL, Y is merged recursively as Y'.
   For all such Y referenced by X, X' is set to reference Y'. (Note
   that if X is managed then X is the same object as X'.)
-  对于所有来自拥有级联元素值 MERGE 或 ALL 的X 的关联引用的实体 Y，Y 递归地被合并为 Y‘。
-  If X is an entity merged to X', with a reference to another
   entity Y, where cascade=MERGE or cascade=ALL is not specified, then
   navigation of the same association from X' yields a reference to a
   managed object Y' with the same persistent identity as Y.
-  如果 X 是一个 的 merged 的实体 X’，并带有另外实体 Y 的引用， cascade=MERGE 或 cascade=ALL 未被指定的情况下，
   那么来自 X' 的同一关联的导航用 Y 相同的持久化的身份产出（yield）一个 managed 对象 Y' 的引用。

The ``merge`` operation will throw an ``OptimisticLockException``
if the entity being merged uses optimistic locking through a
version field and the versions of the entity being merged and the
managed copy don't match. This usually means that the entity has
been modified while being detached.

如果正在被合并的实体通过一个版本字段使用乐观锁，并且该实体的该版本正在被合并而且 managed 拷贝不匹配，
``merge`` 操作将抛出一个 ``OptimisticLockException`` 异常。
这通常意味着该实体已经被修改当正在被 detached 时。

The ``merge`` operation is usually not as frequently needed and
used as ``persist`` and ``remove``. The most common scenario for
the ``merge`` operation is to reattach entities to an EntityManager
that come from some cache (and are therefore detached) and you want
to modify and persist such an entity.

``merge`` 操作通常不经常需要相较于使用 ``persist`` 和 ``remove``。最通常的使用``merge`` 操作情景是
用于重新附上来自一些缓存（因此而 detached 的）实体至 EntityManager 并且你想像实体一样修改和持久它。

.. warning::

    If you need to perform multiple merges of entities that share certain subparts
    of their object-graphs and cascade merge, then you have to call ``EntityManager#clear()`` between the
    successive calls to ``EntityManager#merge()``. Otherwise you might end up with
    multiple copies of the "same" object in the database, however with different ids.

    如果你需要执行实体的多合并，在共享某些对象图（object-graphs）的子部分和级联合并，那么你不得不在连续调用
    ``EntityManager#merge()``之间调用一次 ``EntityManager#clear()``。否则，你可能以在数据库中存在多个“相同”对象
    的拷贝而告终，但是拥有不同的 ids。

.. note::

    If you load some detached entities from a cache and you do
    not need to persist or delete them or otherwise make use of them
    without the need for persistence services there is no need to use
    ``merge``. I.e. you can simply pass detached objects from a cache
    directly to the view.

    如果从缓存中载入一些 detached 实体，并且你不需要持久或删除它们或者相反让它们可以使用没有持久化服务需要
    就没有必要使用 ``merge``。例如，你可以简单地从缓存中直接传递一个 detached 对象给视图。

Synchronization with the Database // 与数据库同步
------------------------------------------------------

The state of persistent entities is synchronized with the database
on flush of an ``EntityManager`` which commits the underlying
``UnitOfWork``. The synchronization involves writing any updates to
persistent entities and their relationships to the database.
Thereby bidirectional relationships are persisted based on the
references held by the owning side of the relationship as explained
in the Association Mapping chapter.

持久化的实体的状态在 ``EntityManager`` flush 后与数据库同步，它提交了一个底层的 ``UnitOfWork``。
同步涉及写入任何更新至持久化的实体和它们的关联至数据库。
从而，双向的关联被持久，基于由在关联映射章节中解释的关联的 owning 一侧保存的引用。

When ``EntityManager#flush()`` is called, Doctrine inspects all
managed, new and removed entities and will perform the following
operations.

当 ``EntityManager#flush()`` 被调用，Doctrine 检查所有的 managed、 new 、 removed 实体并
执行以下的操作。

.. _workingobjects_database_uow_outofsync:

Effects of Database and UnitOfWork being Out-Of-Sync // 数据库与 UnitOfWork 是未同步（Out-Of-Sync）的影响
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As soon as you begin to change the state of entities, call persist or remove the
contents of the UnitOfWork and the database will drive out of sync. They can
only be synchronized by calling ``EntityManager#flush()``. This section
describes the effects of database and UnitOfWork being out of sync.

一旦你开始更改实体的状态，调用 persist 或 remove UnitOfWork 的内容，数据库将驱动 out of sync。
它们仅能通过调用 ``EntityManager#flush()`` 被同步。
这部分描述数据库与 UnitOfWork 是未同步（Out-Of-Sync）的影响。

-  Entities that are scheduled for removal can still be queried from the database.
   They are returned from DQL and Repository queries and are visible in collections.
-  被计划移除的实体仍能从数据库被查询。它们从 DQL 或 Repository 查询被返回，并且在集合中可见。
-  Entities that are passed to ``EntityManager#persist`` do not turn up in query
   results.
-  被传递至 ``EntityManager#persist`` 的实体不能出现在查询结果中。
-  Entities that have changed will not be overwritten with the state from the database.
   This is because the identity map will detect the construction of an already existing
   entity and assumes its the most up to date version.
-  已经变更的实体将不被使用来自数据库的状态重写。这是因为身份映射将检测已存在实体的构造并假定它是最新的版本。

``EntityManager#flush()`` is never called implicitly by Doctrine. You always have to trigger it manually.

``EntityManager#flush()`` 永远不会通过 Doctrine 被隐式地调用。你必须总是手动触发它。

Synchronizing New and Managed Entities // 同步 New 和 Managed 实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The flush operation applies to a managed entity with the following
semantics:

应用于 managed 实体的 flush 操作拥有如下含义：

-  The entity itself is synchronized to the database using a SQL
   UPDATE statement, only if at least one persistent field has
   changed.
-  实体自身使用 SQL UPDATE 语句被同步至数据库，仅当最少一个持久化的字段已经变更时。
-  No SQL updates are executed if the entity did not change.
-  无 SQL 更新被执行，如果实体没有变更的话。

The flush operation applies to a new entity with the following
semantics:

应用于 new 实体的 flush 操作拥有如下含义：


-  The entity itself is synchronized to the database using a SQL
   INSERT statement.
-  实体自身使用 SQL INSERT 语句被同步至数据库。

For all (initialized) relationships of the new or managed entity
the following semantics apply to each associated entity X:

对于 new 或 managed 实体的所有（已初始化）关联，以下含义应用于每一个相关的实体 X：

-  If X is new and persist operations are configured to cascade on
   the relationship, X will be persisted.
-  如果 X 是 new 和 persist 操作被配置到级联该关联上，X 将被持久。
-  If X is new and no persist operations are configured to cascade
   on the relationship, an exception will be thrown as this indicates
   a programming error.
-  如果 X 是 new 和 非 persist 操作被配置至级联在该关联上，一个异常将被抛出这表明了一个编程错误。
-  If X is removed and persist operations are configured to cascade
   on the relationship, an exception will be thrown as this indicates
   a programming error (X would be re-persisted by the cascade).
-  如果 X 是 removed 和 persist 操作被配置至级联在该关联上，一个异常将被抛出这表明了一个编程错误。
   （X 大概被重新持久由该级联）。
-  If X is detached and persist operations are configured to
   cascade on the relationship, an exception will be thrown (This is
   semantically the same as passing X to persist()).
-  如果 X 是 detached 和 persist 操作被配置至级联在该关联上，一个异常将被抛出（这是
   语义上于传递 X 至 persist()相同）。

Synchronizing Removed Entities // 同步 Removed 实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The flush operation applies to a removed entity by deleting its
persistent state from the database. No cascade options are relevant
for removed entities on flush, the cascade remove option is already
executed during ``EntityManager#remove($entity)``.

flush 操作应用到一个 removed 实体以从数据库中删除它的持久化的状态。
对于 removed 实体在 flush 上无 cascade 选项是有意义的,该 cascade remove 选项
已经在 ``EntityManager#remove($entity)`` 期间被执行。

The size of a Unit of Work // UnitofWork 的大小
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The size of a Unit of Work mainly refers to the number of managed
entities at a particular point in time.

UnitOfWork 的大小主要是指在特定的时间点上 managed 实体的数量。

The cost of flushing // 刷新（flushing）的开销
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

How costly a flush operation is, mainly depends on two factors:

一个 flush 操作有多大的开销，主要依赖于两点：

-  The size of the EntityManager's current UnitOfWork.
-  EntityManager 的当前 UnitOfWork 的大小。
-  The configured change tracking policies
-  配置的变更跟踪策略。

You can get the size of a UnitOfWork as follows:

你可以获得 UnitOfWork 的大小，如下：

.. code-block:: php

    <?php
    $uowSize = $em->getUnitOfWork()->size();

The size represents the number of managed entities in the Unit of
Work. This size affects the performance of flush() operations due
to change tracking (see "Change Tracking Policies") and, of course,
memory consumption, so you may want to check it from time to time
during development.

大小代表了在 UnitOfWork 中 managed 实体的数量。
此大小由于变更跟踪（查看“变更跟踪策略”），当然还有内存消耗从而影响 flush() 操作的性能，
所以你可能想在开发期间实时检测它。

.. note::

    Do not invoke ``flush`` after every change to an entity
    or every single invocation of persist/remove/merge/... This is an
    anti-pattern and unnecessarily reduces the performance of your
    application. Instead, form units of work that operate on your
    objects and call ``flush`` when you are done. While serving a
    single HTTP request there should be usually no need for invoking
    ``flush`` more than 0-2 times.

    不要在对实体的每个变更或每个单一 persist/remove/merge/..的调用之后就调用 ``flush``。
    这是反模式和不可避免地降低你应用程序的性能。相反，应当从 UnitOfWork 操作你的对象，并在
    完成操作后调用 ``flush``。在单一 HTTP 请求服务期间通常不需要调用 ``flush`` 超过0-2次。


Direct access to a Unit of Work // 直接访问一个 UnitOfWork
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can get direct access to the Unit of Work by calling
``EntityManager#getUnitOfWork()``. This will return the UnitOfWork
instance the EntityManager is currently using.

你能够通过调用 ``EntityManager#getUnitOfWork()`` 直接访问 UnitOfWork。
它将返回一个 EntityManager 当前正在使用的 UnitOfWork 实例。

.. code-block:: php

    <?php
    $uow = $em->getUnitOfWork();

.. note::

    Directly manipulating a UnitOfWork is not recommended.
    When working directly with the UnitOfWork API, respect methods
    marked as INTERNAL by not using them and carefully read the API
    documentation.

    并不推荐直接地维护一个 UnitOfWork。
    当直接地使用 UnitOfWork API 工作，请遵循不使用标记为 INTERNAL 的方法并仔细阅读 API 文档。


Entity State // 实体状态
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As outlined in the architecture overview an entity can be in one of
four possible states: NEW, MANAGED, REMOVED, DETACHED. If you
explicitly need to find out what the current state of an entity is
in the context of a certain ``EntityManager`` you can ask the
underlying ``UnitOfWork``:

作为在体系结构概述的要点，一个实体能够处于四种可能状态之一： NEW、MANAGED、REMOVED、DETACHED。
如果你明显需要在某个 ``EntityManager`` 的上下文中查找一个实体当前的状态，你可以询问底层的 ``UnitOfWork``。

.. code-block:: php

    <?php
    switch ($em->getUnitOfWork()->getEntityState($entity)) {
        case UnitOfWork::STATE_MANAGED:
            ...
        case UnitOfWork::STATE_REMOVED:
            ...
        case UnitOfWork::STATE_DETACHED:
            ...
        case UnitOfWork::STATE_NEW:
            ...
    }

An entity is in MANAGED state if it is associated with an
``EntityManager`` and it is not REMOVED.

如果一个实体被关联至一个 ``EntityManager``，那么它处在 MANAGED 状态而不是 REMOVED 状态。

An entity is in REMOVED state after it has been passed to
``EntityManager#remove()`` until the next flush operation of the
same EntityManager. A REMOVED entity is still associated with an
``EntityManager`` until the next flush operation.

一个实体被传递至 ``EntityManager#remove()`` 之后直到下一次 flush 操作，该实体处于 REMOVED 状态。
一个 REMOVED 状态的实体仍然被关联至一个 ``EntityManager`` 直到下一次 flush 操作。

An entity is in DETACHED state if it has persistent state and
identity but is currently not associated with an
``EntityManager``.

如果一个实体有了持久化的状态和身份但当前没有被关联至一个 ``EntityManager``，那么它处在 DETACHED 状态。

An entity is in NEW state if has no persistent state and identity
and is not associated with an ``EntityManager`` (for example those
just created via the "new" operator).

如果一个实体没有持久化的状态和身份且没有被关联至一个 ``EntityManager`` （例如，那些刚刚通过“new”操作符创建的），
那么它处在 NEW 状态。

Querying // 查询
-----------------------

Doctrine 2 provides the following ways, in increasing level of
power and flexibility, to query for persistent objects. You should
always start with the simplest one that suits your needs.

Doctrine 2 提供了以下的方式，来提升查询持久化的对象的能力水平和灵活性。
你应当总是从一个最简单的合适你需要的方式开始。

By Primary Key // 通过主键
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most basic way to query for a persistent object is by its
identifier / primary key using the
``EntityManager#find($entityName, $id)`` method. Here is an
example:

最基础的查询持久化的对象的方式是通过它们的标识符/主键，使用 ``EntityManager#find($entityName, $id)`` 方法。
这里有个例子：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    $user = $em->find('MyProject\Domain\User', $id);

The return value is either the found entity instance or null if no
instance could be found with the given identifier.

它将返回找到的实体实例或如果使用给定标识符未能找到实例的话返回 null。

Essentially, ``EntityManager#find()`` is just a shortcut for the
following:

本质上，``EntityManager#find()`` 是一个快捷方式：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    $user = $em->getRepository('MyProject\Domain\User')->find($id);

``EntityManager#getRepository($entityName)`` returns a repository
object which provides many ways to retrieve entities of the
specified type. By default, the repository instance is of type
``Doctrine\ORM\EntityRepository``. You can also use custom
repository classes as shown later.

``EntityManager#getRepository($entityName)`` 返回一个 repository 对象，该对象提供了多种方式
来取回指定类型的实体。默认地，该 repository 实例的类型是 ``Doctrine\ORM\EntityRepository``。
你也可以使用自定义的 repository 类，这将在后面展示。

By Simple Conditions // 通过简单的条件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To query for one or more entities based on several conditions that
form a logical conjunction, use the ``findBy`` and ``findOneBy``
methods on a repository as follows:

基于几个逻辑与形式的条件来查询一个或多个实体，像如下在一个 repository 上使用 ``findBy`` and ``findOneBy`` 方法：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    
    // All users that are 20 years old
    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20));
    
    // All users that are 20 years old and have a surname of 'Miller'
    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20, 'surname' => 'Miller'));
    
    // A single user by its nickname
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('nickname' => 'romanb'));

You can also load by owning side associations through the repository:

你也能够通过该 repository 由 owning 侧关联载入：

.. code-block:: php

    <?php
    $number = $em->find('MyProject\Domain\Phonenumber', 1234);
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('phone' => $number->getId()));

The ``EntityRepository#findBy()`` method additionally accepts orderings, limit and offset as second to fourth parameters:

``EntityRepository#findBy()`` 方法额外接受 orderings、limit、offset 作为第2至4参数：

.. code-block:: php

    <?php
    $tenUsers = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => 20), array('name' => 'ASC'), 10, 0);

If you pass an array of values Doctrine will convert the query into a WHERE field IN (..) query automatically:

如果你传递了一个值的数组，Doctrine 将自动转换查询至一个 WHERE field IN （..）查询：

.. code-block:: php

    <?php
    $users = $em->getRepository('MyProject\Domain\User')->findBy(array('age' => array(20, 30, 40)));
    // translates roughly to: SELECT * FROM users WHERE age IN (20, 30, 40)

An EntityRepository also provides a mechanism for more concise
calls through its use of ``__call``. Thus, the following two
examples are equivalent:

EntityRepository 也提供了一个机制来更简略的调用，通过使用 ``__call`` 魔术方法。
因此，以下两个例子是等价的：

.. code-block:: php

    <?php
    // A single user by its nickname
    $user = $em->getRepository('MyProject\Domain\User')->findOneBy(array('nickname' => 'romanb'));
    
    // A single user by its nickname (__call magic)
    $user = $em->getRepository('MyProject\Domain\User')->findOneByNickname('romanb');

Additionally, you can just count the result of the provided conditions when you don't really need the data:

另外，你也能只是 count 给定条件的结果，当你不真正需要那些数据时。

.. code-block:: php

    <?php
    // Check there is no user with nickname
    $availableNickname = 0 === $em->getRepository('MyProject\Domain\User')->count(['nickname' => 'nonexistent']);

By Criteria // 通过 Criteria
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.3

The Repository implement the ``Doctrine\Common\Collections\Selectable``
interface. That means you can build ``Doctrine\Common\Collections\Criteria``
and pass them to the ``matching($criteria)`` method.

Repository 实现了 ``Doctrine\Common\Collections\Selectable`` 接口。
这意味着你能够构建 ``Doctrine\Common\Collections\Criteria``，然后将它们传递给 ``matching($criteria)`` 方法。

See section `Filtering collections` of chapter :doc:`Working with Associations <working-with-associations>`

查看 :doc:`使用关联 <working-with-associations>` 章节的 `过滤集合` 部分。

By Eager Loading // 通过预加载（Eager Loading）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whenever you query for an entity that has persistent associations
and these associations are mapped as EAGER, they will automatically
be loaded together with the entity being queried and is thus
immediately available to your application.

每当你查询一个具有持久化的关联的实体，且这些关联作为 EAGER 被映射，它们将自动地与正在被查询的实体一起被载入，
并因此你的应用程序可立刻使用它们。

By Lazy Loading // 通过懒加载
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whenever you have a managed entity instance at hand, you can
traverse and use any associations of that entity that are
configured LAZY as if they were in-memory already. Doctrine will
automatically load the associated objects on demand through the
concept of lazy-loading.

每当你有个 managed 实体实例，你可以遍历并使用任何被配置为 LAZY 的实体的关联，如同它们已经在内存中一样。
Doctrine 将自动地按需载入这些已关联的对象，通过懒加载的概念。

By DQL // 通过 DQL
~~~~~~~~~~~~~~~~~~~~~~~~~

The most powerful and flexible method to query for persistent
objects is the Doctrine Query Language, an object query language.
DQL enables you to query for persistent objects in the language of
objects. DQL understands classes, fields, inheritance and
associations. DQL is syntactically very similar to the familiar SQL
but *it is not SQL*.

最强大和灵活的查询持久化的对象的方法是 Doctrine 查询语言，它是一个对象查询语言。
DQL 使你能够在对象的语言中查询持久化的对象。DQL 理解类、字段、继承和关联。
DQL 在语法上非常类似熟悉的 SQL，但是*它不是 SQL*。

A DQL query is represented by an instance of the
``Doctrine\ORM\Query`` class. You create a query using
``EntityManager#createQuery($dql)``. Here is a simple example:

DQL 查询通过 ``Doctrine\ORM\Query`` 类的一个实例被表示。使用
``EntityManager#createQuery($dql)`` 创建一个查询。这有个简单例子：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    
    // All users with an age between 20 and 30 (inclusive).
    $q = $em->createQuery("select u from MyDomain\Model\User u where u.age >= 20 and u.age <= 30");
    $users = $q->getResult();

Note that this query contains no knowledge about the relational
schema, only about the object model. DQL supports positional as
well as named parameters, many functions, (fetch) joins,
aggregates, subqueries and much more. Detailed information about
DQL and its syntax as well as the Doctrine class can be found in
:doc:`the dedicated chapter <dql-doctrine-query-language>`.
For programmatically building up queries based on conditions that
are only known at runtime, Doctrine provides the special
``Doctrine\ORM\QueryBuilder`` class. More information on
constructing queries with a QueryBuilder can be found
:doc:`in Query Builder chapter <query-builder>`.

注意这个查询不包括关于关系数据库的知识，仅关于对象模型。DQL 支持诸如命名参数、
多数函数、（取回）联结、聚合，子查询等等方面。有关 DQL 的详细信息和语法包括 Doctrine
类可以在 :doc:`专门的章节 <dql-doctrine-query-language>` 找到。对于以编程方式
在运行时构建基于已知条件的查询，Doctrine 提供了专门的 ``Doctrine\ORM\QueryBuilder`` 类。
更多的使用 QueryBuilder 构造查询信息可以在 :doc:`查询构造 <query-builder>` 章节找到。

By Native Queries // 通过原生的查询
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As an alternative to DQL or as a fallback for special SQL
statements native queries can be used. Native queries are built by
using a hand-crafted SQL query and a ResultSetMapping that
describes how the SQL result set should be transformed by Doctrine.
More information about native queries can be found in
:doc:`the dedicated chapter <native-sql>`.

作为一个 DQL 的替代或作为对专用 SQL 语句的后备，原生查询能够可以被使用。原生的查询是通过
手写 SQL 查询构建，ResultSetMapping 描述了 SQL 的结果集应该如何通过 Doctrine 被转换。
更多关于原生查询的信息可以在 :doc:`专门的章节 <native-sql>` 被找到。

Custom Repositories // 自定义 Repositories
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default the EntityManager returns a default implementation of
``Doctrine\ORM\EntityRepository`` when you call
``EntityManager#getRepository($entityClass)``. You can overwrite
this behaviour by specifying the class name of your own Entity
Repository in the Annotation, XML or YAML metadata. In large
applications that require lots of specialized DQL queries using a
custom repository is one recommended way of grouping these queries
in a central location.

默认情况下，当你调用 ``EntityManager#getRepository($entityClass)`` 的时候，
EntityManager 返回一个 ``Doctrine\ORM\EntityRepository`` 的默认实现。
你可以在注释块、 XML 或 YAML 元数据中通过指定你自己的 Entity Repository 类名来覆盖此行为。
在需要大量专门的 DQL 查询的大型应用中，在一个集中位置使用自定义 repository 来组织这些查询是被推荐的方式。

.. code-block:: php

    <?php
    namespace MyDomain\Model;
    
    use Doctrine\ORM\EntityRepository;
    use Doctrine\ORM\Mapping as ORM;
    
    /**
     * @ORM\Entity(repositoryClass="MyDomain\Model\UserRepository")
     */
    class User
    {
    
    }
    
    class UserRepository extends EntityRepository
    {
        public function getAllAdminUsers()
        {
            return $this->_em->createQuery('SELECT u FROM MyDomain\Model\User u WHERE u.status = "admin"')
                             ->getResult();
        }
    }

You can access your repository now by calling:

你现在可以通过调用访问你的 repository：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    
    $admins = $em->getRepository('MyDomain\Model\User')->getAllAdminUsers();


