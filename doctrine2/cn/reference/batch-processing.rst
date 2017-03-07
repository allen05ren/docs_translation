Batch Processing // 批量处理
=================================

This chapter shows you how to accomplish bulk inserts, updates and
deletes with Doctrine in an efficient way. The main problem with
bulk operations is usually not to run out of memory and this is
especially what the strategies presented here provide help with.

本章展示给你如何使用 Doctrine 在一个有效率的方式完成大量的 inserts、updates 和 deletes。
大量操作的主要问题通常不是耗尽内存，尤其介绍于此的策略提供帮助。


.. warning::

    An ORM tool is not primarily well-suited for mass
    inserts, updates or deletions. Every RDBMS has its own, most
    effective way of dealing with such operations and if the options
    outlined below are not sufficient for your purposes we recommend
    you use the tools for your particular RDBMS for these bulk
    operations.

    一个 ORM 工具不是主要适用于大量 inserts、updates 或 deletions。每个 RDBMS 都有
    它自己的高效率处理此类操作的方法并且如果以下概括的选项不能满足你的目的，我们推荐你使用你
    的特定 RDBMS 为这些大量操作的提供的工具。


Bulk Inserts // 大量 Inserts
---------------------------------

Bulk inserts in Doctrine are best performed in batches, taking
advantage of the transactional write-behind behavior of an
``EntityManager``. The following code shows an example for
inserting 10000 objects with a batch size of 20. You may need to
experiment with the batch size to find the size that works best for
you. Larger batch sizes mean more prepared statement reuse
internally but also mean more work during ``flush``.

大量 inserts 在 Doctrine 中最好是分批被执行，获得 ``EntityManager`` 的事务后写（write-behind）行为的优势。
下面代码展示了一个用每批20个对象插入10000个对象的例子。你可能需要尝试分批大小以找到一个对你来说
能够工作的最好的分批大小。较大的分批大小意味着更多的预语句在内部重用，但也意味着在 ``flush`` 期间
做更多的工作。

.. code-block:: php

    <?php
    $batchSize = 20;
    for ($i = 1; $i <= 10000; ++$i) {
        $user = new CmsUser;
        $user->setStatus('user');
        $user->setUsername('user' . $i);
        $user->setName('Mr.Smith-' . $i);
        $em->persist($user);
        if (($i % $batchSize) === 0) {
            $em->flush();
            $em->clear(); // Detaches all objects from Doctrine!
        }
    }
    $em->flush(); //Persist objects that did not make up an entire batch
    $em->clear();

Bulk Updates // 大量 Updates
---------------------------------

There are 2 possibilities for bulk updates with Doctrine.

这里有两种使用 Doctrine 大量 uodates 的可能情况。

DQL UPDATE // 数据库查询语言 UPDATE
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The by far most efficient way for bulk updates is to use a DQL
UPDATE query. Example:

目前为止最有效率的大量 updates 的方式是使用 DQL UPDATE 查询。例如：

.. code-block:: php

    <?php
    $q = $em->createQuery('update MyProject\Model\Manager m set m.salary = m.salary * 0.9');
    $numUpdated = $q->execute();

Iterating results // 迭代结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An alternative solution for bulk updates is to use the
``Query#iterate()`` facility to iterate over the query results step
by step instead of loading the whole result into memory at once.
The following example shows how to do this, combining the iteration
with the batching strategy that was already used for bulk inserts:

另一种大量 updates 的解决方案是使用 ``Query#iterate()`` 工具逐一迭代查询结果替代
立刻载入整个结果到内存中。下面的例子展示如何做到这样，组合迭代与已经用于大量 inserts
中的分批策略。

.. code-block:: php

    <?php
    $batchSize = 20;
    $i = 0;
    $q = $em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    foreach ($iterableResult as $row) {
        $user = $row[0];
        $user->increaseCredit();
        $user->calculateNewBonuses();
        if (($i % $batchSize) === 0) {
            $em->flush(); // Executes all updates.
            $em->clear(); // Detaches all objects from Doctrine!
        }
        ++$i;
    }
    $em->flush();

.. note::

    Iterating results is not possible with queries that
    fetch-join a collection-valued association. The nature of such SQL
    result sets is not suitable for incremental hydration.

    取回联结（fetch-join）一个集合值（collection-valued）关联的查询，迭代结果是不可能的。
    这种 SQL 结果集的性质是不适宜增量 hydration的。

.. note::

    Results may be fully buffered by the database client/ connection allocating
    additional memory not visible to the PHP process. For large sets this
    may easily kill the process for no apparent reason.

    结果可能被完全地由数据库客户端/连接（client/ connection）分配的额外内存缓存，此内存对于 PHP 进程不可见，。
    对于大的集合（sets）这可能很容易地无缘无故杀死一个进程。

Bulk Deletes // 大量 Deletes
---------------------------------

There are two possibilities for bulk deletes with Doctrine. You can
either issue a single DQL DELETE query or you can iterate over
results removing them one at a time.

这里有两种使用 Doctrine 大量 deletes 的可能情况。你可以发布单个 DQL DELETE 查询或
迭代结果一次一个移除它们。

DQL DELETE // 数据库查询语言 DELETE
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The by far most efficient way for bulk deletes is to use a DQL
DELETE query.

目前为止最有效率的大量 deletes 的方式是使用 DQL DELETE 查询。

Example:

例如：

.. code-block:: php

    <?php
    $q = $em->createQuery('delete from MyProject\Model\Manager m where m.salary > 100000');
    $numDeleted = $q->execute();

Iterating results // 迭代结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An alternative solution for bulk deletes is to use the
``Query#iterate()`` facility to iterate over the query results step
by step instead of loading the whole result into memory at once.
The following example shows how to do this:

另一种大量 deletes 的解决方案是使用 ``Query#iterate()`` 工具逐一迭代查询结果替代
立刻载入整个结果到内存中。下面的例子展示如何做到这样：

.. code-block:: php

    <?php
    $batchSize = 20;
    $i = 0;
    $q = $em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    while (($row = $iterableResult->next()) !== false) {
        $em->remove($row[0]);
        if (($i % $batchSize) === 0) {
            $em->flush(); // Executes all deletions.
            $em->clear(); // Detaches all objects from Doctrine!
        }
        ++$i;
    }
    $em->flush();

.. note::

    Iterating results is not possible with queries that
    fetch-join a collection-valued association. The nature of such SQL
    result sets is not suitable for incremental hydration.

    取回联结（fetch-join）一个集合值（collection-valued）关联的查询，迭代结果是不可能的。
    这种 SQL 结果集的性质是不适宜增量 hydration的。


Iterating Large Results for Data-Processing // 迭代大结果以加工数据
---------------------------------------------------------------------

You can use the ``iterate()`` method just to iterate over a large
result and no UPDATE or DELETE intention. The ``IterableResult``
instance returned from ``$query->iterate()`` implements the
Iterator interface so you can process a large result without memory
problems using the following approach:

你可以使用 ``iterate()`` 方法仅迭代一个大结果而无 UPDATE 或 DELETE 意图。
从 ``$query->iterate()`` 返回的 ``IterableResult`` 实例实现了迭代器接口，
所以你可以处理一个大结果而无内存问题，使用以下方法：

.. code-block:: php

    <?php
    $q = $this->_em->createQuery('select u from MyProject\Model\User u');
    $iterableResult = $q->iterate();
    foreach ($iterableResult as $row) {
        // do stuff with the data in the row, $row[0] is always the object
    
        // detach from Doctrine, so that it can be Garbage-Collected immediately
        $this->_em->detach($row[0]);
    }

.. note::

    Iterating results is not possible with queries that
    fetch-join a collection-valued association. The nature of such SQL
    result sets is not suitable for incremental hydration.

    取回联结（fetch-join）一个集合值（collection-valued）关联的查询，迭代结果是不可能的。
    这种 SQL 结果集的性质是不适宜增量 hydration的。

