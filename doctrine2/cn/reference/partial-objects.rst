Partial Objects // 部分对象
=================================

A partial object is an object whose state is not fully initialized
after being reconstituted from the database and that is
disconnected from the rest of its data. The following section will
describe why partial objects are problematic and what the approach
of Doctrine2 to this problem is.

部分对象是在从数据库被重构并且与其余数据断开连接后状态未完全被初始化的对象。
以下部分将描述为何部分对象是有问题的，并且 Doctrine 2对此问题的方法是什么。

.. note::

    The partial object problem in general does not apply to
    methods or queries where you do not retrieve the query result as
    objects. Examples are: ``Query#getArrayResult()``,
    ``Query#getScalarResult()``, ``Query#getSingleScalarResult()``,
    etc.

    部分对象的问题通常不适用于不取回查询结果作为对象的方法或查询。例如：
    ``Query#getArrayResult()``、``Query#getScalarResult()``、
    ``Query#getSingleScalarResult()`` 等等。

.. warning::

    Use of partial objects is tricky. Fields that are not retrieved
    from the database will not be updated by the UnitOfWork even if they
    get changed in your objects. You can only promote a partial object
    to a fully-loaded object by calling ``EntityManager#refresh()``
    or a DQL query with the refresh flag.

    使用部分对象时“狡猾”的。从数据库不被取回的字段将不会通过 UnitOfWork 被更新，即使
    在你的对象中它们得到了改变。你仅可以通过调用 ``EntityManager#refresh()`` 或
    带 refresh 标志的 DQL 查询提升部分对象为完整加载的对象。


What is the problem? // 问题是什么？
--------------------------------------------

In short, partial objects are problematic because they are usually
objects with broken invariants. As such, code that uses these
partial objects tends to be very fragile and either needs to "know"
which fields or methods can be safely accessed or add checks around
every field access or method invocation. The same holds true for
the internals, i.e. the method implementations, of such objects.
You usually simply assume the state you need in the method is
available, after all you properly constructed this object before
you pushed it into the database, right? These blind assumptions can
quickly lead to null reference errors when working with such
partial objects.

简而言之，部分对象是有问题的因为它们通常是具有破坏的不可变对象。因此，使用这些部分对象的代码
往往是非常脆弱的且需要“知道”哪些字段或方法可以安全地访问或围绕每个字段访问或方法调用添加检查。
对于此类对象的内部同样如此，比如方法的实现。你通常简单地假设你需要的状态在方法中是可用的，然后
在你推送它进入数据库之前你全部正确地构造此对象，对吧？这些盲目的假设可以快速地导致空引用错误，当
使用这样的部分对象时。

It gets worse with the scenario of an optional association (0..1 to
1). When the associated field is NULL, you don't know whether this
object does not have an associated object or whether it was simply
not loaded when the owning object was loaded from the database.

对于一个可选关联（0..1 到 1）的情况，它变得更糟糕。当关联得字段是 NULL，你不知道是否
此对象没有关联得对象或者当 owning 对象从数据库加载时是否它简单地不被加载。

These are reasons why many ORMs do not allow partial objects at all
and instead you always have to load an object with all its fields
(associations being proxied). One secure way to allow partial
objects is if the programming language/platform allows the ORM tool
to hook deeply into the object and instrument it in such a way that
individual fields (not only associations) can be loaded lazily on
first access. This is possible in Java, for example, through
bytecode instrumentation. In PHP though this is not possible, so
there is no way to have "secure" partial objects in an ORM with
transparent persistence.

由一些原因为何多数 ORMs 全然不允许部分对象而是你始终必须加载一个对象及其所有字段
（关联正在被代理）。一种安全方式以允许部分对象是如果编程语言/平台允许 ORM 工具
深入地 hook 进对象和在这样得方式中调试它，单独的字段（不仅关联）在第一次访问上可以
懒散地被加载。在 Java 中这是可能的，例如，如果 bytecode 调试。在 PHP 中尽管这是不
可能的，所以在具有透明持久化的 ORM 中没有办法拥有“安全”的部分对象。

Doctrine, by default, does not allow partial objects. That means,
any query that only selects partial object data and wants to
retrieve the result as objects (i.e. ``Query#getResult()``) will
raise an exception telling you that partial objects are dangerous.
If you want to force a query to return you partial objects,
possibly as a performance tweak, you can use the ``partial``
keyword as follows:

默认地，Doctrine 不允许部分对象。这意味着，任何仅选择部分对象数据和希望将此结果
作为对象取回的查询（如，``Query#getResult()``）将引起一个异常告诉你部分对象是危险的。
如果你希望强制查询以返回部分对象，很可能因为性能的调整，你可以使用 ``partial`` 关键字如下：


.. code-block:: php

    <?php
    $q = $em->createQuery("select partial u.{id,name} from MyApp\Domain\User u");

You can also get a partial reference instead of a proxy reference by
calling:

你也可以得到一个部分引用替代代理引用，通过调用：

.. code-block:: php

    <?php
    $reference = $em->getPartialReference('MyApp\Domain\User', 1);

Partial references are objects with only the identifiers set as they
are passed to the second argument of the ``getPartialReference()`` method.
All other fields are null.

部分引用是仅有标识符集的对象，因为它们被传递至 ``getPartialReference()`` 方法
的第二个参数。所有其他字段都是 null。

When should I force partial objects? // 何时我应该强制部分对象？
--------------------------------------------------------------------

Mainly for optimization purposes, but be careful of premature
optimization as partial objects lead to potentially more fragile
code.

主要出于优化目的，但是小心过早优化，因为部分对象导致潜在的更脆弱的代码。
