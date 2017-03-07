Change Tracking Policies // 变更跟踪策略
============================================

Change tracking is the process of determining what has changed in
managed entities since the last time they were synchronized with
the database.

变更跟踪是确定自最近一次与数据库同步后在 managed 的实体中发生了什么更改的过程。

Doctrine provides 3 different change tracking policies, each having
its particular advantages and disadvantages. The change tracking
policy can be defined on a per-class basis (or more precisely,
per-hierarchy).

Doctrine 提供了三种不同的变更跟踪策略，每一种都有它的特有的优势和劣势。变更跟踪
策略可以被定义在每个类（per-class）基础上（或更准确地，每个层次结构)。

Deferred Implicit // 隐式延迟
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The deferred implicit policy is the default change tracking policy
and the most convenient one. With this policy, Doctrine detects the
changes by a property-by-property comparison at commit time and
also detects changes to entities or new entities that are
referenced by other managed entities ("persistence by
reachability"). Although the most convenient policy, it can have
negative effects on performance if you are dealing with large units
of work (see "Understanding the Unit of Work"). Since Doctrine
can't know what has changed, it needs to check all managed entities
for changes every time you invoke EntityManager#flush(), making
this operation rather costly.

隐式延迟策略是默认的变更跟踪策略并且最实用的一个。使用此策略，Doctrine 在一次提交上
通过逐个的比较属性侦测变更，并且也侦测其他 managed 实体引用的实体或新实体的变更
（“可达性持久化”）。尽管是最实用的策略，它也有负面的性能影响，如果你正在处理大的
UnitOfWork（请查看“理解 UnitOfWork”）。由于 Doctrine 不能知道有什么更新，每次你
调用 EntityManager#flush()，它需要检查所有 managed 实体的变更，使得此操作非常
“昂贵”。

Deferred Explicit // 显式延迟
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The deferred explicit policy is similar to the deferred implicit
policy in that it detects changes through a property-by-property
comparison at commit time. The difference is that Doctrine 2 only
considers entities that have been explicitly marked for change detection
through a call to EntityManager#persist(entity) or through a save
cascade. All other entities are skipped. This policy therefore
gives improved performance for larger units of work while
sacrificing the behavior of "automatic dirty checking".

显式延迟策略类似于隐式延迟策略，它在每次提交时通过逐个的比较属性侦测变更。不同的是，
Doctrine 2 仅考虑通过一个调用 EntityManager#persist(entity) 或通以一个保存级联
已经显式地被标记为变更侦测的实体。所有其他的实体被跳过。因此，该策略为大的 UnitOfWork
提供了改进的性能，但是牺牲了“自动脏检查”的行为。

Therefore, flush() operations are potentially cheaper with this
policy. The negative aspect this has is that if you have a rather
large application and you pass your objects through several layers
for processing purposes and business tasks you may need to track
yourself which entities have changed on the way so you can pass
them to EntityManager#persist().

因此，使用此策略潜在地 flush() 操作更“廉价”。负面的方面是，如果你有一个非常大的应用程序
并且你穿越几个层级传递你的对象以便以处理的目的和业务任务，你可能必须自己跟踪在此过程中那个
实体已经变更，所以你可以将它们传递给 EntityManager#persist()。

This policy can be configured as follows:

此策略可以被配置如下：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @ChangeTrackingPolicy("DEFERRED_EXPLICIT")
     */
    class User
    {
        // ...
    }

Notify // 通知
~~~~~~~~~~~~~~~~~~~

This policy is based on the assumption that the entities notify
interested listeners of changes to their properties. For that
purpose, a class that wants to use this policy needs to implement
the ``NotifyPropertyChanged`` interface from the Doctrine
namespace. As a guideline, such an implementation can look as
follows:

此策略是基于这样的假设，实体向感兴趣的侦听器通知它的属性变更。出于此目的，希望使用此
策略的类需要实现来自 Doctrine 命名空间的 ``NotifyPropertyChanged`` 接口。
作为指导，这样的实现可能看起来像如下：

.. code-block:: php

    <?php
    use Doctrine\Common\NotifyPropertyChanged,
        Doctrine\Common\PropertyChangedListener;
    
    /**
     * @Entity
     * @ChangeTrackingPolicy("NOTIFY")
     */
    class MyEntity implements NotifyPropertyChanged
    {
        // ...
    
        private $_listeners = array();
    
        public function addPropertyChangedListener(PropertyChangedListener $listener)
        {
            $this->_listeners[] = $listener;
        }
    }

Then, in each property setter of this class or derived classes, you
need to notify all the ``PropertyChangedListener`` instances. As an
example we add a convenience method on ``MyEntity`` that shows this
behaviour:

然后，在此类或衍生类的每个属性设置器（seter）中，你需要通知所有的 ``PropertyChangedListener`` 实例。
作为例子，我们在 ``MyEntity`` 上添加了一个方便的方法来展示此行为：

.. code-block:: php

    <?php
    // ...
    
    class MyEntity implements NotifyPropertyChanged
    {
        // ...
    
        protected function _onPropertyChanged($propName, $oldValue, $newValue)
        {
            if ($this->_listeners) {
                foreach ($this->_listeners as $listener) {
                    $listener->propertyChanged($this, $propName, $oldValue, $newValue);
                }
            }
        }
    
        public function setData($data)
        {
            if ($data != $this->data) {
                $this->_onPropertyChanged('data', $this->data, $data);
                $this->data = $data;
            }
        }
    }

You have to invoke ``_onPropertyChanged`` inside every method that
changes the persistent state of ``MyEntity``.

你必须在每个变更 ``MyEntity`` 的持久化的状态的方法内部调用 ``_onPropertyChanged``。

The check whether the new value is different from the old one is
not mandatory but recommended. That way you also have full control
over when you consider a property changed.

检查是否新值不同于旧值不是强制的，但是推荐的。此方式你也拥有完全的控制，当你考虑
一个属性的变更时。

The negative point of this policy is obvious: You need implement an
interface and write some plumbing code. But also note that we tried
hard to keep this notification functionality abstract. Strictly
speaking, it has nothing to do with the persistence layer and the
Doctrine ORM or DBAL. You may find that property notification
events come in handy in many other scenarios as well. As mentioned
earlier, the ``Doctrine\Common`` namespace is not that evil and
consists solely of very small classes and interfaces that have
almost no external dependencies (none to the DBAL and none to the
ORM) and that you can easily take with you should you want to swap
out the persistence layer. This change tracking policy does not
introduce a dependency on the Doctrine DBAL/ORM or the persistence
layer.

此策略的负面点时显而易见的：你需要实现一个接口并写一些管道代码。但是也注意,我们已尝试
努力保持此通知功能的抽象。严格来说，它与持久化层和 Doctrine ORM 或 DBAL 无关。
你可能发现，属性通知事件在许多其他情况也派得上用场。如前所述，``Doctrine\Common``
命名空间不是邪恶，仅由几乎没有外部依赖（没有对 DBAL 和 ORM）的非常小的类和接口组成，
并且你可以轻松地带走你应该希望的置换出持久层。此更改跟踪策略不会对Doctrine DBAL/ORM
或持久化层引入依赖性。

The positive point and main advantage of this policy is its
effectiveness. It has the best performance characteristics of the 3
policies with larger units of work and a flush() operation is very
cheap when nothing has changed.

此策略的积极一面和主要优势是它的效率。对于更大的 UnitOfWord 它拥有三种策略最好的性能特点
且当没有任何变更时 flush() 操作是非常“便宜的”。
