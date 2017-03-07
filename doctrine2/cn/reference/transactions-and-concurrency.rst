Transactions and Concurrency // 事务与并发
=================================================

.. _transactions-and-concurrency_transaction-demarcation:

Transaction Demarcation // 事务界限
------------------------------------------

Transaction demarcation is the task of defining your transaction
boundaries. Proper transaction demarcation is very important
because if not done properly it can negatively affect the
performance of your application. Many databases and database
abstraction layers like PDO by default operate in auto-commit mode,
which means that every single SQL statement is wrapped in a small
transaction. Without any explicit transaction demarcation from your
side, this quickly results in poor performance because transactions
are not cheap.

事务界限是定义你的事务边界的工作。恰当的事务界限是非常重要的，因为如果不这么做会给你的应用程序
带来负面的影响。多数数据库和数据库抽象层如 PDO 默认运作在自动提交（auto-commit）模式，
这意味着每一个单一 SQL 语句被封装在一个小的事务中。没有任何明确来自你这一侧的事务界限，将很快导致
不良性能，因为事务不是“廉价的”。

For the most part, Doctrine 2 already takes care of proper
transaction demarcation for you: All the write operations
(INSERT/UPDATE/DELETE) are queued until ``EntityManager#flush()``
is invoked which wraps all of these changes in a single
transaction.

对于大多数情况，Doctrine 2 已经为你适当地考虑了：所有的写操作（INSERT/UPDATE/DELETE）
被队列直到 ``EntityManager#flush()`` 被调用，封装了所有变更在单个事务中。

However, Doctrine 2 also allows (and encourages) you to take over
and control transaction demarcation yourself.

然而，Doctrine 2 也允许（且鼓励）你自己接管并控制事务界限。

These are two ways to deal with transactions when using the
Doctrine ORM and are now described in more detail.

当使用 Doctrine ORM 时有两个方式处理事务，现在就详细地介绍它们。

.. _transactions-and-concurrency_approach-implicitly:

Approach 1: Implicitly // 方法1：隐式地
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The first approach is to use the implicit transaction handling
provided by the Doctrine ORM EntityManager. Given the following
code snippet, without any explicit transaction demarcation:

第一个方法是使用通过 Doctrine ORM 的 EntityManager 提供的隐式事务处理。考虑以下代码片断，
没有任何显式事务界限：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    $user = new User;
    $user->setName('George');
    $em->persist($user);
    $em->flush();

Since we do not do any custom transaction demarcation in the above
code, ``EntityManager#flush()`` will begin and commit/rollback a
transaction. This behavior is made possible by the aggregation of
the DML operations by the Doctrine ORM and is sufficient if all the
data manipulation that is part of a unit of work happens through
the domain model and thus the ORM.

因为我们没有做任何自定义事务界限在上述代码中，``EntityManager#flush()`` 将开启并
提交/回滚一个事务。通过 Doctrine ORM 的 DML（数据操作语言）操作的聚合使此行为成为可能。
并且它是足够的，如果作为 UnitOfWork 的一部分的所有数据操作通过领域模型并从而通过 ORM产生。

.. _transactions-and-concurrency_approach-explicitly:

Approach 2: Explicitly // 方法2：显式地
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The explicit alternative is to use the ``Doctrine\DBAL\Connection``
API directly to control the transaction boundaries. The code then
looks like this:

显式的方案是使用 ``Doctrine\DBAL\Connection`` API 直接地控制事务边界。
代码类似这样：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    $em->getConnection()->beginTransaction(); // suspend auto-commit
    try {
        //... do some work
        $user = new User;
        $user->setName('George');
        $em->persist($user);
        $em->flush();
        $em->getConnection()->commit();
    } catch (Exception $e) {
        $em->getConnection()->rollBack();
        throw $e;
    }

Explicit transaction demarcation is required when you want to
include custom DBAL operations in a unit of work or when you want
to make use of some methods of the ``EntityManager`` API that
require an active transaction. Such methods will throw a
``TransactionRequiredException`` to inform you of that
requirement.

显式的事务界限被需要，当你想要包含自定义 DBAL 操作在 UnitOfWork 中或当你想要使用 ``EntityManager``
的一些方法而这些方法需要一个活动的事务时，需要显式的事务界限。这些方法将抛出一个 ``TransactionRequiredException``
异常来告诉你该要求。

A more convenient alternative for explicit transaction demarcation is the use
of provided control abstractions in the form of
``Connection#transactional($func)`` and ``EntityManager#transactional($func)``.
When used, these control abstractions ensure that you never forget to rollback
the transaction, in addition to the obvious code reduction. An example that is
functionally equivalent to the previously shown code looks as follows:

对于显式的事务界限一个更省事的方案是使用在 ``Connection#transactional($func)`` 和 ``EntityManager#transactional($func)``
结构中提供的控制抽象。当使用时，这些控制抽象确保你永远不会忘记回滚事务，除外明显的减少了代码。
一个功能与前面展示代码等价的例子看起来如下：

.. code-block:: php

    <?php
    // $em instanceof EntityManager
    $em->transactional(function($em) {
        //... do some work
        $user = new User;
        $user->setName('George');
        $em->persist($user);
    });

.. warning::

    For historical reasons, ``EntityManager#transactional($func)`` will return
    ``true`` whenever the return value of ``$func`` is loosely false.
    Some examples of this include ``array()``, ``"0"``, ``""``, ``0``, and
    ``null``.

    由于历史的原因，``EntityManager#transactional($func)`` 将返回 ``true``，
    每当 ``$func`` 的返回值是不严格地 false 时。比如 ``array()``、``"0"``、``""``、``0`` 和
    ``null``。

The difference between ``Connection#transactional($func)`` and
``EntityManager#transactional($func)`` is that the latter
abstraction flushes the ``EntityManager`` prior to transaction
commit and rolls back the transaction when an
exception occurs.

``Connection#transactional($func)`` 和 ``EntityManager#transactional($func)``
之间的不同是，后者的抽象早于事务提交刷新 ``EntityManager`` 并当一个异常发生时回滚事务。

.. _transactions-and-concurrency_exception-handling:

Exception Handling // 异常处理
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When using implicit transaction demarcation and an exception occurs
during ``EntityManager#flush()``, the transaction is automatically
rolled back and the ``EntityManager`` closed.

当使用隐式事务界限且在 ``EntityManager#flush()`` 期间发生了一个异常，该事务会自动地被
回滚并关闭 ``EntityManager``。

When using explicit transaction demarcation and an exception
occurs, the transaction should be rolled back immediately and the
``EntityManager`` closed by invoking ``EntityManager#close()`` and
subsequently discarded, as demonstrated in the example above. This
can be handled elegantly by the control abstractions shown earlier.
Note that when catching ``Exception`` you should generally re-throw
the exception. If you intend to recover from some exceptions, catch
them explicitly in earlier catch blocks (but do not forget to
rollback the transaction and close the ``EntityManager`` there as
well). All other best practices of exception handling apply
similarly (i.e. either log or re-throw, not both, etc.).

当使用显式事务界限且一个异常发生，该事务应该立刻被回滚且通过调用 ``EntityManager#close()``
关闭 ``EntityManager``并随后丢弃，就像上面的例子演示的那样。这可以通过之前展示的控制抽象
优雅地被处理。注意，当捕获 ``异常`` 时，你通常应该重新抛出该异常。如果你打算从一些异常恢复，
在更早的 catch 块明确地捕获它们（但是同样不要忘记回滚该事务并关闭 ``EntityManager``）。
所有其他的异常处理最佳实践同样地适用（如，日志、重新抛出、或两者都不等等）。


As a result of this procedure, all previously managed or removed
instances of the ``EntityManager`` become detached. The state of
the detached objects will be the state at the point at which the
transaction was rolled back. The state of the objects is in no way
rolled back and thus the objects are now out of synch with the
database. The application can continue to use the detached objects,
knowing that their state is potentially no longer accurate.

作为该过程的一个结果，所有先前 ``EntityManager`` managed 或 removed 的实例变为
detached。这些 detached 对象的状态将是该事务被回滚的那个点的状态。这些对象的状态是
决不被回滚且因此这些对象现在不与数据库同步。应重程序可以继续使用这些 detached 对象，但应该知道它们的
状态可能地不再精确。

If you intend to start another unit of work after an exception has
occurred you should do that with a new ``EntityManager``.

如果你打算在一个异常已经发生后开启另一个 UnitOfWork，你应该使用一个新的 ``EntityManager``。

.. _transactions-and-concurrency_locking-support:

Locking Support // 锁支持
--------------------------------

Doctrine 2 offers support for Pessimistic- and Optimistic-locking
strategies natively. This allows to take very fine-grained control
over what kind of locking is required for your Entities in your
application.

Doctrine 2 天然地对悲观锁和乐观锁提供支持。在你的应用程序中，这允许获得非常细粒度上的控制你的实体
使用何种锁。

.. _transactions-and-concurrency_optimistic-locking:

Optimistic Locking // 乐观锁
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Database transactions are fine for concurrency control during a
single request. However, a database transaction should not span
across requests, the so-called "user think time". Therefore a
long-running "business transaction" that spans multiple requests
needs to involve several database transactions. Thus, database
transactions alone can no longer control concurrency during such a
long-running business transaction. Concurrency control becomes the
partial responsibility of the application itself.

数据库事务在单请求中对并发的控制很好。然而，一个数据库事务不应该跨越请求，
就是所谓的“用户思考时间”。因此，一个长时间运行的跨越多请求的“业务事务”
需要涉及几个数据库事务。因此，在这样一个长时间运行的业务事务中单独的数据库事务
不能再控制并发性。并发控制变成应用程序自身的部分责任。

Doctrine has integrated support for automatic optimistic locking
via a version field. In this approach any entity that should be
protected against concurrent modifications during long-running
business transactions gets a version field that is either a simple
number (mapping type: integer) or a timestamp (mapping type:
datetime). When changes to such an entity are persisted at the end
of a long-running conversation the version of the entity is
compared to the version in the database and if they don't match, an
``OptimisticLockException`` is thrown, indicating that the entity
has been modified by someone else already.

Doctrine 通过一个版本字段已经整合了对自动乐观锁的支持。在这个方法中，
任何实体对并发的修改应该被保护，在长时间运行的业务事务期间得到一个简单数字
（映射类型：integer）或时间戳（映射类型：datetime）版本字段。当这样一个实体的变更被
持久时，在一个长时间会话的最后，实体的版本与在数据库中的版本被比较，并且如果它们不匹配，
一个 ``OptimisticLockException`` 异常被抛出，指示该实体已经被其他人修改了。

You designate a version field in an entity as follows. In this
example we'll use an integer.

在实体中，你像如下这样指派一个版本字段，本例中我们将使用一个整数。

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            // ...
            /** @Version @Column(type="integer") */
            private $version;
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="User">
            <field name="version" type="integer" version="true" />
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          fields:
            version:
              version:
                type: integer

Alternatively a datetime type can be used (which maps to a SQL
timestamp or datetime):

或者一个 datetime 类型可以被使用（它映射至 SQL timestamp 或 datetime）：

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            // ...
            /** @Version @Column(type="datetime") */
            private $version;
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="User">
            <field name="version" type="datetime" version="true" />
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          fields:
            version:
              version:
                type: datetime

Version numbers (not timestamps) should however be preferred as
they can not potentially conflict in a highly concurrent
environment, unlike timestamps where this is a possibility,
depending on the resolution of the timestamp on the particular
database platform.

然而，版本数字（不是时间戳）应该被推荐。因为它们不可能冲突在一个高并发环境中，不像时间戳
可能冲突，时间戳依赖于特定数据库平台上的时间精确度。

When a version conflict is encountered during
``EntityManager#flush()``, an ``OptimisticLockException`` is thrown
and the active transaction rolled back (or marked for rollback).
This exception can be caught and handled. Potential responses to an
OptimisticLockException are to present the conflict to the user or
to refresh or reload objects in a new transaction and then retrying
the transaction.

当一个版本冲突被遭遇在 ``EntityManager#flush()`` 期间，一个 ``OptimisticLockException``
异常被抛出且活动的事务被回滚（或标记为回滚）。该异常能够被捕获和处理。潜在的响应一个
OptimisticLockException 异常呈现该冲突让用户或刷新或重新载入对象在一个新事务中，然后
重试该事务。

With PHP promoting a share-nothing architecture, the time between
showing an update form and actually modifying the entity can in the
worst scenario be as long as your applications session timeout. If
changes happen to the entity in that time frame you want to know
directly when retrieving the entity that you will hit an optimistic
locking exception:

使用 PHP 推崇的无共享架构，在最糟糕的情况下，显示一个更新形成和真实地修改实体之间的时间
可能长达你的应用程序会话超时时间。如果对一个实体的变更发生在此时间帧中，你想直接地知道
当取回该实体时，你将命中一个乐观锁异常。


You can always verify the version of an entity during a request
either when calling ``EntityManager#find()``:

你可以总是检查一个实体的版本在一个请求期间或当调用 ``EntityManager#find()`` 时：

.. code-block:: php

    <?php
    use Doctrine\DBAL\LockMode;
    use Doctrine\ORM\OptimisticLockException;
    
    $theEntityId = 1;
    $expectedVersion = 184;
    
    try {
        $entity = $em->find('User', $theEntityId, LockMode::OPTIMISTIC, $expectedVersion);
    
        // do the work
    
        $em->flush();
    } catch(OptimisticLockException $e) {
        echo "Sorry, but someone else has already changed this entity. Please apply the changes again!";
    }

Or you can use ``EntityManager#lock()`` to find out:

或者你可以使用 ``EntityManager#lock()`` 找出：

.. code-block:: php

    <?php
    use Doctrine\DBAL\LockMode;
    use Doctrine\ORM\OptimisticLockException;
    
    $theEntityId = 1;
    $expectedVersion = 184;
    
    $entity = $em->find('User', $theEntityId);
    
    try {
        // assert version
        $em->lock($entity, LockMode::OPTIMISTIC, $expectedVersion);
    
    } catch(OptimisticLockException $e) {
        echo "Sorry, but someone else has already changed this entity. Please apply the changes again!";
    }

Important Implementation Notes // 重要的实现说明
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can easily get the optimistic locking workflow wrong if you
compare the wrong versions. Say you have Alice and Bob editing a
hypothetical blog post:

你可以很容易地得到一个乐观锁工作流错误，如果你比较一个错误的版本。假定你有一个 Alice 和
Bob 在编辑的假设的博客帖子：

-  Alice reads the headline of the blog post being "Foo", at
   optimistic lock version 1 (GET Request)
-  Alice 读取了该博客帖子的标题为“Foo”，在乐观锁版本1（GET 请求）
-  Bob reads the headline of the blog post being "Foo", at
   optimistic lock version 1 (GET Request)
-  Bob 同样读取了该博客帖子的标题为“Foo”，在乐观锁版本1（GET 请求）
-  Bob updates the headline to "Bar", upgrading the optimistic lock
   version to 2 (POST Request of a Form)
-  Bob 更新该标题为“Bar”，更新该乐观锁版本至2（一个 POST 表单请求）
-  Alice updates the headline to "Baz", ... (POST Request of a
   Form)
-  Alice 更新该标题为“Baz”，...（一个 POST 表单请求）

Now at the last stage of this scenario the blog post has to be read
again from the database before Alice's headline can be applied. At
this point you will want to check if the blog post is still at
version 1 (which it is not in this scenario).

现在此情况下最后一个阶段，在 Alice 的标题能够被应用之前该博客帖子不得不从数据库中再被读取。
在这个点上你将想要检查是否该博客帖子仍然处在版本1（它并不在此情况中）。

Using optimistic locking correctly, you *have* to add the version
as an additional hidden field (or into the SESSION for more
safety). Otherwise you cannot verify the version is still the one
being originally read from the database when Alice performed her
GET request for the blog post. If this happens you might see lost
updates you wanted to prevent with Optimistic Locking.

正确地使用乐观锁，你*必须*添加一个版本作为一个额外的隐藏字段（或放进 SESSION 中为了更安全）。
否则，你不能验证该版本仍然是那个从数据库读取的原始版本，当 Alice 执行她的 GET 请求该博客帖子的时候。
如果发生这种情况，你希望使用乐观锁阻止可能看到的更新丢失。

See the example code, The form (GET Request):

查看例子代码，一个表单（GET 请求）：

.. code-block:: php

    <?php
    $post = $em->find('BlogPost', 123456);
    
    echo '<input type="hidden" name="id" value="' . $post->getId() . '" />';
    echo '<input type="hidden" name="version" value="' . $post->getCurrentVersion() . '" />';

And the change headline action (POST Request):

和更改标题的操作（POST 请求）：

.. code-block:: php

    <?php
    $postId = (int)$_GET['id'];
    $postVersion = (int)$_GET['version'];
    
    $post = $em->find('BlogPost', $postId, \Doctrine\DBAL\LockMode::OPTIMISTIC, $postVersion);

.. _transactions-and-concurrency_pessimistic-locking:

Pessimistic Locking // 悲观锁
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine 2 supports Pessimistic Locking at the database level. No
attempt is being made to implement pessimistic locking inside
Doctrine, rather vendor-specific and ANSI-SQL commands are used to
acquire row-level locks. Every Entity can be part of a pessimistic
lock, there is no special metadata required to use this feature.

Doctrine 2 在数据库层面上支持悲观锁。无试图在 Doctrine 内部实现悲观锁，而是使用供应商指定和
ANSI-SQL 命令获得行级别（row-level）的锁。每个实体都可以是悲观锁的一部分，
使用该特性无特定的元数据需要。

However for Pessimistic Locking to work you have to disable the
Auto-Commit Mode of your Database and start a transaction around
your pessimistic lock use-case using the "Approach 2: Explicit
Transaction Demarcation" described above. Doctrine 2 will throw an
Exception if you attempt to acquire an pessimistic lock and no
transaction is running.

然而，为使悲观锁得以工作你必须禁用你的数据库的自动提交（Auto-Commit）模式，并使用上述
“方法2：显式的事务界限”所描述的那样围绕你的悲观锁用例开启一个事务。Doctrine 2 将抛出
一个异常，如果你试图获得一个悲观锁并无事务正在运行。

Doctrine 2 currently supports two pessimistic lock modes:

Doctrine 2 目前支持两种悲观锁模式：

-  Pessimistic Write
   (``Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE``), locks the
   underlying database rows for concurrent Read and Write Operations.
-  悲观写（``Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE``），为并发读或写操作锁定
   底层数据库行。
-  Pessimistic Read (``Doctrine\DBAL\LockMode::PESSIMISTIC_READ``),
   locks other concurrent requests that attempt to update or lock rows
   in write mode.
-  悲观读（``Doctrine\DBAL\LockMode::PESSIMISTIC_READ``），锁定其他试图更新的并发请求或
   锁定行在写模式。

You can use pessimistic locks in three different scenarios:

在三种不同的情况下，你可以使用悲观锁：

1. Using
   ``EntityManager#find($className, $id, \Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   or
   ``EntityManager#find($className, $id, \Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``
   使用 ``EntityManager#find($className, $id, \Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   或 ``EntityManager#find($className, $id, \Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``
2. Using
   ``EntityManager#lock($entity, \Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   or
   ``EntityManager#lock($entity, \Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``
   使用 ``EntityManager#lock($entity, \Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   或 ``EntityManager#lock($entity, \Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``
3. Using
   ``Query#setLockMode(\Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   or
   ``Query#setLockMode(\Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``
   使用 ``Query#setLockMode(\Doctrine\DBAL\LockMode::PESSIMISTIC_WRITE)``
   或 ``Query#setLockMode(\Doctrine\DBAL\LockMode::PESSIMISTIC_READ)``


