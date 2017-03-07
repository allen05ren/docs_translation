Events // 事件
===================

Doctrine 2 features a lightweight event system that is part of the
Common package. Doctrine uses it to dispatch system events, mainly
:ref:`lifecycle events <reference-events-lifecycle-events>`.
You can also use it for your own custom events.

Doctrine 2 特性一个轻量级事件系统，它是 Common 包的一部分。Doctrine 使用它分发系统事件，
大部分 :ref:`生命周期事件 <reference-events-lifecycle-events>`。

The Event System // 事件系统
---------------------------------

The event system is controlled by the ``EventManager``. It is the
central point of Doctrine's event listener system. Listeners are
registered on the manager and events are dispatched through the
manager.

事件系统通过 ``EventManager`` 控制。它是 Doctrine 的事件侦听器系统的核心。
侦听器被注册在该管理器上并且通过该管理器分发事件。

.. code-block:: php

    <?php
    $evm = new EventManager();

Now we can add some event listeners to the ``$evm``. Let's create a
``TestEvent`` class to play around with.

现在我们可以添加一些事件侦听器到 ``$evm``。让我们创建一个 ``TestEvent`` 类来试一试：

.. code-block:: php

    <?php
    class TestEvent
    {
        const preFoo = 'preFoo';
        const postFoo = 'postFoo';

        private $_evm;

        public $preFooInvoked = false;
        public $postFooInvoked = false;

        public function __construct($evm)
        {
            $evm->addEventListener(array(self::preFoo, self::postFoo), $this);
        }

        public function preFoo(EventArgs $e)
        {
            $this->preFooInvoked = true;
        }

        public function postFoo(EventArgs $e)
        {
            $this->postFooInvoked = true;
        }
    }

    // Create a new instance
    $test = new TestEvent($evm);

Events can be dispatched by using the ``dispatchEvent()`` method.

事件可以使用 ``dispatchEvent()`` 方法被分发。

.. code-block:: php

    <?php
    $evm->dispatchEvent(TestEvent::preFoo);
    $evm->dispatchEvent(TestEvent::postFoo);

You can easily remove a listener with the ``removeEventListener()``
method.

你可以使用 ``removeEventListener()`` 方法轻易移除一个侦听器。

.. code-block:: php

    <?php
    $evm->removeEventListener(array(self::preFoo, self::postFoo), $this);

The Doctrine 2 event system also has a simple concept of event
subscribers. We can define a simple ``TestEventSubscriber`` class
which implements the ``\Doctrine\Common\EventSubscriber`` interface
and implements a ``getSubscribedEvents()`` method which returns an
array of events it should be subscribed to.

Doctrine 2 事件系统也有一个简单的事件订阅器概念。我们可以定义一个简单 ``TestEventSubscriber`` 类，
它实现了 ``\Doctrine\Common\EventSubscriber`` 接口和实现了一个 ``getSubscribedEvents()`` 方法，
该方法返回一个事件的数组，它应该是被订阅的事件。

.. code-block:: php

    <?php
    class TestEventSubscriber implements \Doctrine\Common\EventSubscriber
    {
        public $preFooInvoked = false;

        public function preFoo()
        {
            $this->preFooInvoked = true;
        }

        public function getSubscribedEvents()
        {
            return array(TestEvent::preFoo);
        }
    }

    $eventSubscriber = new TestEventSubscriber();
    $evm->addEventSubscriber($eventSubscriber);

.. note::

    The array to return in the ``getSubscribedEvents`` method is a simple array
    with the values being the event names. The subscriber must have a method
    that is named exactly like the event.

    该数组在 ``getSubscribedEvents`` 方法返回，它是一个带有该事件名值的简单数组。该订阅器必须
    有一个被命名为该事件名的方法。

Now when you dispatch an event, any event subscribers will be
notified for that event.

现在当你分发一个事件，任何对于该事件的事件侦听器将收到通知。

.. code-block:: php

    <?php
    $evm->dispatchEvent(TestEvent::preFoo);

Now you can test the ``$eventSubscriber`` instance to see if the
``preFoo()`` method was invoked.

现在你可以测试 ``$eventSubscriber`` 实例，看看如果 ``preFoo()`` 方法被调用。

.. code-block:: php

    <?php
    if ($eventSubscriber->preFooInvoked) {
        echo 'pre foo invoked!';
    }

Naming convention // 命名规范
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Events being used with the Doctrine 2 EventManager are best named
with camelcase and the value of the corresponding constant should
be the name of the constant itself, even with spelling. This has
several reasons:

正在被使用的事件和 Doctrine 2 EventManager 被良好地用驼峰格式命名并且相应常量的值
应该是常量自身的名字，甚至于拼写。这有几个原因：

-  It is easy to read.
-  易于阅读。
-  Simplicity.
-  简单性。
-  Each method within an EventSubscriber is named after the
   corresponding constant's value. If the constant's name and value differ
   it contradicts the intention of using the constant and makes your code
   harder to maintain.
-  EventSubscriber 内部的每个方法以相应常量的值命名。如果该常量的名字与值不同，
   这与使用该常量的目的相矛盾且让你的代码难以维护。

An example for a correct notation can be found in the example
``TestEvent`` above.

一个恰当的表示法例子可以在以上的 ``TestEvent`` 例子中找到。

.. _reference-events-lifecycle-events:

Lifecycle Events // 生命周期事件
------------------------------------

The EntityManager and UnitOfWork trigger a bunch of events during
the life-time of their registered entities.

EntityManager 和 UnitOfWork 触发了大量注册在实体生命周期的事件。

-  preRemove - The preRemove event occurs for a given entity before
   the respective EntityManager remove operation for that entity is
   executed.  It is not called for a DQL DELETE statement.
-  preRemove - preRemove 事件发生在一个给定实体相应的 EntityManager 执行 remove 操作之前。
   对于 DQL DELETE 语句它不会被调用。
-  postRemove - The postRemove event occurs for an entity after the
   entity has been deleted. It will be invoked after the database
   delete operations. It is not called for a DQL DELETE statement.
-  postRemove - postRemove 事件发生在一个实体已经被删除之后。它将在数据库删除操作之后被调用。
   对于 DQL DELETE 语句它不会被调用。
-  prePersist - The prePersist event occurs for a given entity
   before the respective EntityManager persist operation for that
   entity is executed. It should be noted that this event is only triggered on
   *initial* persist of an entity (i.e. it does not trigger on future updates).
-  prePersist - prePersist 事件发生在一个给定实体相应的 EntityManager 执行 persist 操作之前。
   应注意该事件仅在实体的*初始* persist 上被触发（比如在将来的 updates 上不触发）。
-  postPersist - The postPersist event occurs for an entity after
   the entity has been made persistent. It will be invoked after the
   database insert operations. Generated primary key values are
   available in the postPersist event.
-  postPersist - postPersist 事件发生在一个实体已经被持久化之后。它将在数据库插入操作之后被调用。
   生成的主键值可以在该 postPersist 事件中使用。
-  preUpdate - The preUpdate event occurs before the database
   update operations to entity data. It is not called for a DQL UPDATE statement
   nor when the computed changeset is empty.
-  preUpdate - preUpdate 事件发生在数据库更新数据至实体数据操作之前。对于 DQL UPDATE 语句或当已计算的变更集
   为空时它不会被调用。
-  postUpdate - The postUpdate event occurs after the database
   update operations to entity data. It is not called for a DQL UPDATE statement.
-  postUpdate - postUpdate 事件发生在数据库更新数据至实体数据操作之后。对于 DQL UPDATE 语句它不会被调用。
-  postLoad - The postLoad event occurs for an entity after the
   entity has been loaded into the current EntityManager from the
   database or after the refresh operation has been applied to it.
-  postLoad - postLoad 事件发生在一个实体已从数据库被加载进当前 EntityManager 之后
   或该实体被应用了 refresh 操作之后。
-  loadClassMetadata - The loadClassMetadata event occurs after the
   mapping metadata for a class has been loaded from a mapping source
   (annotations/xml/yaml). This event is not a lifecycle callback.
-  loadClassMetadata - loadClassMetadata 事件发生在一个类的映射元数据已从一个
   映射源（annotations/xml/yaml）被加载之后。该事件不是一个生命周期回调。
-  onClassMetadataNotFound - Loading class metadata for a particular
   requested class name failed. Manipulating the given event args instance
   allows providing fallback metadata even when no actual metadata exists
   or could be found. This event is not a lifecycle callback.
-  onClassMetadataNotFound - 一个特定请求的类名载入类元数据失败时发生。操纵给定
   EventArgs 实例允许提供回滚的元数据，甚至于无真实元数据存在或无能够被找到的真实元数据。
-  preFlush - The preFlush event occurs at the very beginning of a flush
   operation.
-  preFlush - preFlush 事件发生在一个 flush 操作的最开始。
-  onFlush - The onFlush event occurs after the change-sets of all
   managed entities are computed. This event is not a lifecycle
   callback.
-  onFlush - onFlush 事件发生在所有 managed 实体的变更集被计算之后。该事件不是一个生命周期回调。
-  postFlush - The postFlush event occurs at the end of a flush operation. This
   event is not a lifecycle callback.
-  postFlush - postFlush 事件发生在一个 flush 操作之后。该事件不是一个生命周期回调。
-  onClear - The onClear event occurs when the EntityManager#clear() operation is
   invoked, after all references to entities have been removed from the unit of
   work. This event is not a lifecycle callback.
-  onClear - onClear 事件发生在当 ``EntityManager#clear()`` 操作被调用时，
   之后所有至实体的引用已从 UnitOfWork 被移除。该事件不是一个生命周期回调。

.. warning::

    Note that, when using ``Doctrine\ORM\AbstractQuery#iterate()``, ``postLoad``
    events will be executed immediately after objects are being hydrated, and therefore
    associations are not guaranteed to be initialized. It is not safe to combine
    usage of ``Doctrine\ORM\AbstractQuery#iterate()`` and ``postLoad`` event
    handlers.

    注意，当使用 ``Doctrine\ORM\AbstractQuery#iterate()`` 时，``postLoad`` 事件将立刻被执行，
    在对象再在被 hydrated 时，因此关联不保证被初始化。组合使用 ``Doctrine\ORM\AbstractQuery#iterate()`` 和 ``postLoad``
    事件处理器是不安全的。

.. warning::

    Note that the postRemove event or any events triggered after an entity removal
    can receive an uninitializable proxy in case you have configured an entity to
    cascade remove relations. In this case, you should load yourself the proxy in
    the associated pre event.

    请注意 postRemove 的事件或实体移除后触发的任何事件，可以收到一个不可初始化代理，
    以防你已经配置了一个实体到级联 remove 的关系。在这种情况下，你应该加载自己在相关的 pre 事件中的代理。

You can access the Event constants from the ``Events`` class in the
ORM package.

在该 ORM 包中，你可以从 ``Events`` 类访问事件常量。

.. code-block:: php

    <?php
    use Doctrine\ORM\Events;
    echo Events::preUpdate;

These can be hooked into by two different types of event
listeners:

它们可以通过两个不同类型的事件侦听器被 hook：

-  Lifecycle Callbacks are methods on the entity classes that are
   called when the event is triggered. As of v2.4 they receive some kind
   of ``EventArgs`` instance.
-  生命周期回调是实体类上当某事件被触发时被调用的方法。自 v2.4 起，它们接收某种 ``EventArgs`` 实例。
-  Lifecycle Event Listeners and Subscribers are classes with specific callback
   methods that receives some kind of ``EventArgs`` instance.
-  生命周期事件侦听器和订阅器是带有特定回调方法的类，此方法接收某种 ``EventArgs`` 实例。

The EventArgs instance received by the listener gives access to the entity,
EntityManager and other relevant data.

通过侦听器取回的 EventArgs 实例可以访问那个实体、EntityManager 和其他相关数据。

.. note::

    All Lifecycle events that happen during the ``flush()`` of
    an EntityManager have very specific constraints on the allowed
    operations that can be executed. Please read the
    :ref:`reference-events-implementing-listeners` section very carefully
    to understand which operations are allowed in which lifecycle event.

    EntityManager 在 ``flush()`` 期间发生的所有生命周期事件都有在允许的操作被执行上的非常特定的限制。
    请阅读 :ref:`实施事件侦听器 <reference-events-implementing-listeners>`
    部分非常仔细解释在那些生命周期事件被允许那些操作。


Lifecycle Callbacks // 生命周期回调
---------------------------------------

Lifecycle Callbacks are defined on an entity class. They allow you to
trigger callbacks whenever an instance of that entity class experiences
a relevant lifecycle event. More than one callback can be defined for each
lifecycle event. Lifecycle Callbacks are best used for simple operations
specific to a particular entity class's lifecycle.

生命周期回调被定义在一个实体类上。它们允许你一旦实体类经历了一个相关的生命周期事件旧触发回调。
每一个生命周期事件都可以定义超过一个回调。生命周期回调被很好地应用在简单地操作特定实体类的生命周期。

.. code-block:: php

    <?php

    /** @Entity @HasLifecycleCallbacks */
    class User
    {
        // ...

        /**
         * @Column(type="string", length=255)
         */
        public $value;

        /** @Column(name="created_at", type="string", length=255) */
        private $createdAt;

        /** @PrePersist */
        public function doStuffOnPrePersist()
        {
            $this->createdAt = date('Y-m-d H:i:s');
        }

        /** @PrePersist */
        public function doOtherStuffOnPrePersist()
        {
            $this->value = 'changed from prePersist callback!';
        }

        /** @PostPersist */
        public function doStuffOnPostPersist()
        {
            $this->value = 'changed from postPersist callback!';
        }

        /** @PostLoad */
        public function doStuffOnPostLoad()
        {
            $this->value = 'changed from postLoad callback!';
        }

        /** @PreUpdate */
        public function doStuffOnPreUpdate()
        {
            $this->value = 'changed from preUpdate callback!';
        }
    }

Note that the methods set as lifecycle callbacks need to be public and,
when using these annotations, you have to apply the
``@HasLifecycleCallbacks`` marker annotation on the entity class.

注意那些设置为生命周期回调的方法集需要是 public 的，并且当使用它们的 annotations时你必须在该实体类上应用
``@HasLifecycleCallbacks`` 标记注释。

If you want to register lifecycle callbacks from YAML or XML you
can do it with the following.

如果你想从 YAML 或 XML 注册生命周期回调，你可以像下面那样做。

.. code-block:: yaml

    User:
      type: entity
      fields:
    # ...
        name:
          type: string(50)
      lifecycleCallbacks:
        prePersist: [ doStuffOnPrePersist, doOtherStuffOnPrePersist ]
        postPersist: [ doStuffOnPostPersist ]

In YAML the ``key`` of the lifecycleCallbacks entry is the event that you
are triggering on and the ``value`` is the method (or methods) to call. The allowed
event types are the ones listed in the previous Lifecycle Events section.

在 YAML 中生命周期回调条目的 ``key`` 和 ``value`` 分别是事件的名称和会被调用的方法（或那些方法）。
被允许的事件种类是在前面生命周期事件部分列出的事件。


XML would look something like this:

XML 看上去像这样：

.. code-block:: xml

    <?xml version="1.0" encoding="UTF-8"?>

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                              /Users/robo/dev/php/Doctrine/doctrine-mapping.xsd">

        <entity name="User">

            <lifecycle-callbacks>
                <lifecycle-callback type="prePersist" method="doStuffOnPrePersist"/>
                <lifecycle-callback type="postPersist" method="doStuffOnPostPersist"/>
            </lifecycle-callbacks>

        </entity>

    </doctrine-mapping>

In XML the ``type`` of the lifecycle-callback entry is the event that you
are triggering on and the ``method`` is the method to call. The allowed event
types are the ones listed in the previous Lifecycle Events section.

在 XML 中生命周期回调条目的 ``type`` 和 ``method`` 分别是事件的名称和会被调用的方法（或那些方法）。
被允许的事件种类是在前面生命周期事件部分列出的事件。

When using YAML or XML you need to remember to create public methods to match the
callback names you defined. E.g. in these examples ``doStuffOnPrePersist()``,
``doOtherStuffOnPrePersist()`` and ``doStuffOnPostPersist()`` methods need to be
defined on your ``User`` model.

当使用 YAML 或 XML 时你需要记得创建与你定义的回调名匹配的 public 方法。例如在这些例子中 ``doStuffOnPrePersist()``、
``doOtherStuffOnPrePersist()`` 和 ``doStuffOnPostPersist()`` 方法需要被定义在你的 ``User`` 模型上。

.. code-block:: php

    <?php
    // ...

    class User
    {
        // ...

        public function doStuffOnPrePersist()
        {
            // ...
        }

        public function doOtherStuffOnPrePersist()
        {
            // ...
        }

        public function doStuffOnPostPersist()
        {
            // ...
        }
    }


Lifecycle Callbacks Event Argument //生命周期回调事件参数
------------------------------------------------------------

.. versionadded:: 2.4

Since 2.4 the triggered event is given to the lifecycle-callback.

自 2.4 版本起，触发的事件会被传给生命周期回调。

With the additional argument you have access to the
``EntityManager`` and ``UnitOfWork`` APIs inside these callback methods.

使用额外的参数，你可以在这些回调方法内部访问 ``EntityManager`` and ``UnitOfWork`` APIs。

.. code-block:: php

    <?php
    // ...

    class User
    {
        public function preUpdate(PreUpdateEventArgs $event)
        {
            if ($event->hasChangedField('username')) {
                // Do something when the username is changed.
            }
        }
    }

Listening and subscribing to Lifecycle Events // 侦听和订阅生命周期事件
-------------------------------------------------------------------------

Lifecycle event listeners are much more powerful than the simple
lifecycle callbacks that are defined on the entity classes. They
sit at a level above the entities and allow you to implement re-usable
behaviors across different entity classes.

生命周期事件侦听器比定义在实体类上的简单生命周期回调更加强大。它们处在实体之上层次，
允许你实现穿越不同实体类的可复用行为。

Note that they require much more detailed knowledge about the inner
workings of the EntityManager and UnitOfWork. Please read the
:ref:`reference-events-implementing-listeners` section carefully if you
are trying to write your own listener.

注意它们需要关于 EntityManager 和 UnitOfWork 内部运行的更多细节知识。请仔细阅读
:ref:`实施事件侦听器 <reference-events-implementing-listeners>` 部分，如果
你正在尝试写一个自己的侦听器的话。

For event subscribers, there are no surprises. They declare the
lifecycle events in their ``getSubscribedEvents`` method and provide
public methods that expect the relevant arguments.

对于事件订阅器，它们没有什么惊奇。它们在 ``getSubscribedEvents`` 方法中声明生命周期事件，
并提供 public 方法期望的相关参数。

A lifecycle event listener looks like the following:

一个生命周期侦听器看上去像下面这样：

.. code-block:: php

    <?php
    use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

    class MyEventListener
    {
        public function preUpdate(LifecycleEventArgs $args)
        {
            $entity = $args->getObject();
            $entityManager = $args->getObjectManager();

            // perhaps you only want to act on some "Product" entity
            if ($entity instanceof Product) {
                // do something with the Product
            }
        }
    }

A lifecycle event subscriber may look like this:

一个生命周期订阅器可能看上去像这样：

.. code-block:: php

    <?php
    use Doctrine\ORM\Events;
    use Doctrine\Common\EventSubscriber;
    use Doctrine\Common\Persistence\Event\LifecycleEventArgs;

    class MyEventSubscriber implements EventSubscriber
    {
        public function getSubscribedEvents()
        {
            return array(
                Events::postUpdate,
            );
        }

        public function postUpdate(LifecycleEventArgs $args)
        {
            $entity = $args->getObject();
            $entityManager = $args->getObjectManager();

            // perhaps you only want to act on some "Product" entity
            if ($entity instanceof Product) {
                // do something with the Product
            }
        }

.. note::

    Lifecycle events are triggered for all entities. It is the responsibility
    of the listeners and subscribers to check if the entity is of a type
    it wants to handle.

    生命周期事件为所有实体所触发。侦听器和订阅器有责任检查它们想要处理的实体类型。

To register an event listener or subscriber, you have to hook it into the
EventManager that is passed to the EntityManager factory:

注册一个事件侦听器或订阅器，你必须钩（hook）进 EventManager 传递至 EventManager 工厂：

.. code-block:: php

    <?php
    $eventManager = new EventManager();
    $eventManager->addEventListener(array(Events::preUpdate), new MyEventListener());
    $eventManager->addEventSubscriber(new MyEventSubscriber());

    $entityManager = EntityManager::create($dbOpts, $config, $eventManager);

You can also retrieve the event manager instance after the
EntityManager was created:

你也可以取回事件管理器实例在 EntityManager 被创建之后：

.. code-block:: php

    <?php
    $entityManager->getEventManager()->addEventListener(array(Events::preUpdate), new MyEventListener());
    $entityManager->getEventManager()->addEventSubscriber(new MyEventSubscriber());

.. _reference-events-implementing-listeners:

Implementing Event Listeners // 实施事件侦听器
--------------------------------------------------

This section explains what is and what is not allowed during
specific lifecycle events of the UnitOfWork. Although you get
passed the EntityManager in all of these events, you have to follow
these restrictions very carefully since operations in the wrong
event may produce lots of different errors, such as inconsistent
data and lost updates/persists/removes.

本部分解释在特定的 UnitOfWork 生命周期中什么是被允许的什么是不被允许的。
尽管你在所有这些事件中得到传递的 EntityManager，你必须非常小心地遵循这些限制，因为
错误操作事件可能产生许多不同的错误，诸如不一致的数据和丢失  updates/persists/removes。

For the described events that are also lifecycle callback events
the restrictions apply as well, with the additional restriction
that (prior to version 2.4) you do not have access to the
EntityManager or UnitOfWork APIs inside these events.

生命周期回调中描述的事件也同样适用该限制，有一个额外的限制（2.4之前版本），你不能在这些事件内部
访问 EntityManager 或 UnitOfWork APIs。

prePersist
~~~~~~~~~~

There are two ways for the ``prePersist`` event to be triggered.
One is obviously when you call ``EntityManager#persist()``. The
event is also called for all cascaded associations.

有两种触发 ``prePersist`` 事件的方式。很显然的一个是当你调用 ``EntityManager#persist()`` 时。
对于所有级联的关联该事件也被调用。

There is another way for ``prePersist`` to be called, inside the
``flush()`` method when changes to associations are computed and
this association is marked as cascade persist. Any new entity found
during this operation is also persisted and ``prePersist`` called
on it. This is called "persistence by reachability".

有另外一个 ``prePersist`` 被调用的方式，在 ``flush()`` 内部当关联的变更被计算且该关联
被标记为级联 persist 时。在此操作期间找到的任何新实体也被持久化且 ``prePersist`` 在其上被调用。
这被称作“可达性持久化”。

In both cases you get passed a ``LifecycleEventArgs`` instance
which has access to the entity and the entity manager.
两种情况都能得到传递的 ``LifecycleEventArgs``，它可访问该实体和实体管理器。

The following restrictions apply to ``prePersist``:

下列的限制适用于 ``prePersist``：

-  If you are using a PrePersist Identity Generator such as
   sequences the ID value will *NOT* be available within any
   PrePersist events.
-  如果你再在使用一个 PrePersist 身份生成器，如序列 ID 的值将*不*可用在任何 PrePersist 事件内部。
-  Doctrine will not recognize changes made to relations in a prePersist
   event. This includes modifications to
   collections such as additions, removals or replacement.
-  Doctrine 将不会承认在 prePersist 事件中关联所做的更改。这包括修改集合，如添加、 清除或更换。

preRemove
~~~~~~~~~

The ``preRemove`` event is called on every entity when its passed
to the ``EntityManager#remove()`` method. It is cascaded for all
associations that are marked as cascade delete.

当实体被传递至 ``EntityManager#remove()`` 方法时 ``preRemove`` 事件被调用。
对于所有被标记为记录 delete 的关联将被级联。

There are no restrictions to what methods can be called inside the
``preRemove`` event, except when the remove method itself was
called during a flush operation.

在 ``preRemove`` 事件内部没有限制什么方法可以被调用，除了remove 方法自身在 flush 操作期间
被调用。

preFlush
~~~~~~~~

``preFlush`` is called at ``EntityManager#flush()`` before
anything else. ``EntityManager#flush()`` can be called safely
inside its listeners.

``preFlush`` 在任何其他之后 ``EntityManager#flush()``之前被调用。
``EntityManager#flush()`` 可以安全地在其侦听器内部被调用。

.. code-block:: php

    <?php

    use Doctrine\ORM\Event\PreFlushEventArgs;

    class PreFlushExampleListener
    {
        public function preFlush(PreFlushEventArgs $args)
        {
            // ...
        }
    }

onFlush
~~~~~~~

OnFlush is a very powerful event. It is called inside
``EntityManager#flush()`` after the changes to all the managed
entities and their associations have been computed. This means, the
``onFlush`` event has access to the sets of:

``OnFlush`` 是非常强大的事件。它在 ``EntityManager#flush()`` 内部所有至 managed 实体
的变更及它们的关联已经被计算之后被调用。这意味着，``onFlush`` 事件可以访问：

-  Entities scheduled for insert
-  计划 insert 的实体
-  Entities scheduled for update
-  计划 update 的实体
-  Entities scheduled for removal
-  计划 remove 的实体
-  Collections scheduled for update
-  计划 update 的集合
-  Collections scheduled for removal
-  计划 remove 的集合

To make use of the onFlush event you have to be familiar with the
internal UnitOfWork API, which grants you access to the previously
mentioned sets. See this example:

为使用 ``onFlush`` 事件你必须熟悉 UnitOfWork 内部的 API，它授予你访问前面提及的实体。
查看以下例子：

.. code-block:: php

    <?php
    class FlushExampleListener
    {
        public function onFlush(OnFlushEventArgs $eventArgs)
        {
            $em = $eventArgs->getEntityManager();
            $uow = $em->getUnitOfWork();

            foreach ($uow->getScheduledEntityInsertions() as $entity) {

            }

            foreach ($uow->getScheduledEntityUpdates() as $entity) {

            }

            foreach ($uow->getScheduledEntityDeletions() as $entity) {

            }

            foreach ($uow->getScheduledCollectionDeletions() as $col) {

            }

            foreach ($uow->getScheduledCollectionUpdates() as $col) {

            }
        }
    }

The following restrictions apply to the onFlush event:

以下限制适用于 ``onFlush`` 事件：

-  If you create and persist a new entity in ``onFlush``, then
   calling ``EntityManager#persist()`` is not enough.
   You have to execute an additional call to
   ``$unitOfWork->computeChangeSet($classMetadata, $entity)``.
-  如果你创建并 persist 一个新实体在 ``onFlush`` 中，那么调用 ``EntityManager#persist()``
   是不够的。你必须执行一个额外的调用 ``$unitOfWork->computeChangeSet($classMetadata, $entity)``。
-  Changing primitive fields or associations requires you to
   explicitly trigger a re-computation of the changeset of the
   affected entity. This can be done by calling
   ``$unitOfWork->recomputeSingleEntityChangeSet($classMetadata, $entity)``.
-  修改原始字段或关联需要你明确触发一个受影响实体的变更集的重新计算。你可以通过调用
   ``$unitOfWork->recomputeSingleEntityChangeSet($classMetadata, $entity)`` 来完成。

postFlush
~~~~~~~~~

``postFlush`` is called at the end of ``EntityManager#flush()``.
``EntityManager#flush()`` can **NOT** be called safely inside its listeners.

``postFlush`` 在 ``EntityManager#flush()`` 之后被调用。
``EntityManager#flush()`` **不** 能在它的侦听器内部被安全地调用。

.. code-block:: php

    <?php

    use Doctrine\ORM\Event\PostFlushEventArgs;

    class PostFlushExampleListener
    {
        public function postFlush(PostFlushEventArgs $args)
        {
            // ...
        }
    }

preUpdate
~~~~~~~~~

PreUpdate is the most restrictive to use event, since it is called
right before an update statement is called for an entity inside the
``EntityManager#flush()`` method. Note that this event is not
triggered when the computed changeset is empty.

``PreUpdate`` 事件的使用是最受限的，因为它正好在 ``EntityManager#flush()``
方法内部的实体的 update 语句被调用之前被调用。注意该事件在已计算的变更集为空时不会被触发。

Changes to associations of the updated entity are never allowed in
this event, since Doctrine cannot guarantee to correctly handle
referential integrity at this point of the flush operation. This
event has a powerful feature however, it is executed with a
``PreUpdateEventArgs`` instance, which contains a reference to the
computed change-set of this entity.

在该事件中永远不允许修改已更新的实体的关联，因为 Doctrine 不能保证正确地处理引用完整性
在 flush 操作的这个点上。当然，该事件有一个强大的特性，它带一个 ``PreUpdateEventArgs``
实例被执行，该实例包含一个到该实体的已计算变更集的引用。

This means you have access to all the fields that have changed for
this entity with their old and new value. The following methods are
available on the ``PreUpdateEventArgs``:

这意味着你可以访问所有的已变更实体的字段的旧值和新值，以下方法在 ``PreUpdateEventArgs``
上可用：

-  ``getEntity()`` to get access to the actual entity.
-  ``getEntity()`` 获得至那个真实的实体的访问
-  ``getEntityChangeSet()`` to get a copy of the changeset array.
   Changes to this returned array do not affect updating.
-  ``getEntityChangeSet()`` 获得变更集数组得一个拷贝。对该返回数组得修改不会影响更新。
-  ``hasChangedField($fieldName)`` to check if the given field name
   of the current entity changed.
-  ``hasChangedField($fieldName)`` 检查给定的当前实体的字段是否修改了。
-  ``getOldValue($fieldName)`` and ``getNewValue($fieldName)`` to
   access the values of a field.
-  ``getOldValue($fieldName)`` 和 ``getNewValue($fieldName)`` 访问一个字段的值
-  ``setNewValue($fieldName, $value)`` to change the value of a
   field to be updated.
-  ``setNewValue($fieldName, $value)`` 修改一个将被更新字段的值。

A simple example for this event looks like:

该事件一个简单的例子看起来像：

.. code-block:: php

    <?php
    class NeverAliceOnlyBobListener
    {
        public function preUpdate(PreUpdateEventArgs $eventArgs)
        {
            if ($eventArgs->getEntity() instanceof User) {
                if ($eventArgs->hasChangedField('name') && $eventArgs->getNewValue('name') == 'Alice') {
                    $eventArgs->setNewValue('name', 'Bob');
                }
            }
        }
    }

You could also use this listener to implement validation of all the
fields that have changed. This is more efficient than using a
lifecycle callback when there are expensive validations to call:

你也能够使用这个侦听器实现所有已变更字段的验证。这比使用生命周期回调更有效率，当有“昂贵”验证调用时：

.. code-block:: php

    <?php
    class ValidCreditCardListener
    {
        public function preUpdate(PreUpdateEventArgs $eventArgs)
        {
            if ($eventArgs->getEntity() instanceof Account) {
                if ($eventArgs->hasChangedField('creditCard')) {
                    $this->validateCreditCard($eventArgs->getNewValue('creditCard'));
                }
            }
        }

        private function validateCreditCard($no)
        {
            // throw an exception to interrupt flush event. Transaction will be rolled back.
        }
    }

Restrictions for this event:

该事件的限制：

-  Changes to associations of the passed entities are not
   recognized by the flush operation anymore.
-  对已传递实体的关联的变更是不被承认的，通过该 flush 操作。
-  Changes to fields of the passed entities are not recognized by
   the flush operation anymore, use the computed change-set passed to
   the event to modify primitive field values, e.g. use
   ``$eventArgs->setNewValue($field, $value);`` as in the Alice to Bob example above.
   对已传递实体的字段的变更是不被承认的，通过该 flush 操作。使用已计算变更集传递至该事件以修改原始字段的值，
   如使用 ``$eventArgs->setNewValue($field, $value);`` 像上述例子中的 Alice 修改为 Bob 一样。
-  Any calls to ``EntityManager#persist()`` or
   ``EntityManager#remove()``, even in combination with the UnitOfWork
   API are strongly discouraged and don't work as expected outside the
   flush operation.
-  任何``EntityManager#persist()`` 或 ``EntityManager#remove()`` 的调用，
   即便是在与 UnitOfWork API 组合都是极力不鼓励的，且在 flush 操作之外不能如预期一样工作。

postUpdate, postRemove, postPersist
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The three post events are called inside ``EntityManager#flush()``.
Changes in here are not relevant to the persistence in the
database, but you can use these events to alter non-persistable items,
like non-mapped fields, logging or even associated classes that are
not directly mapped by Doctrine.

这三个 post 事件在 ``EntityManager#flush()`` 内部被调用。在这里变更是无关数据库持久化，
但是你可以使用这些事件修改非可持久的条目，像非映射字段、日志或甚至不是通过 Doctrine 被直接地映射的相关的类。

postLoad
~~~~~~~~

This event is called after an entity is constructed by the
EntityManager.

该事件在实体通过 EntityManager 被构造后被调用。

Entity listeners // 实体侦听器
----------------------------------

.. versionadded:: 2.4

An entity listener is a lifecycle listener class used for an entity.

一个实体侦听器是一个用于实体的生命周期侦听器类

- The entity listener's mapping may be applied to an entity class or mapped superclass.
- 该实体侦听器的映射可能被应用于一个实体类或映射的子类。
- An entity listener is defined by mapping the entity class with the corresponding mapping.
- 一个实体侦听器被定义，通过映射实体类与相应的映射。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Entity;

        /** @Entity @EntityListeners({"UserListener"}) */
        class User
        {
            // ....
        }
    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Entity\User">
                <entity-listeners>
                    <entity-listener class="UserListener"/>
                </entity-listeners>
                <!-- .... -->
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Entity\User:
          type: entity
          entityListeners:
            UserListener:
          # ....

.. _reference-entity-listeners:

Entity listeners class // 实体侦听器类
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An ``Entity Listener`` could be any class, by default it should be a class with a no-arg constructor.

一个 ``Entity Listener`` 可以是任何类，默认地它应该是带一个无参构造器的类。

- Different from :ref:`reference-events-implementing-listeners` an ``Entity Listener`` is invoked just to the specified entity
- 不同于 :ref:`实施事件侦听器 <reference-events-implementing-listeners>`，一个 ``Entity Listener`` 仅为特定的实体所调用。
- An entity listener method receives two arguments, the entity instance and the lifecycle event.
- 一个实体侦听器方法接收两个参数，该实体实例和该生命周期事件。
- The callback method can be defined by naming convention or specifying a method mapping.
- 回调方法可以通过命名规范或指定方法映射被定义。
- When a listener mapping is not given the parser will use the naming convention to look for a matching method,
  e.g. it will look for a public ``preUpdate()`` method if you are listening to the ``preUpdate`` event.
- 当侦听器映射无给定解析器将使用命名规范来查找匹配的方法，如它将查找一个 public 的 ``preUpdate()`` 方法，如果你正在侦听 ``preUpdate`` 事件的话。
- When a listener mapping is given the parser will not look for any methods using the naming convention.
- 当侦听器映射给定了解析器将不会使用命名规范查找任何方法。

.. code-block:: php

    <?php
    class UserListener
    {
        public function preUpdate(User $user, PreUpdateEventArgs $event)
        {
            // Do something on pre update.
        }
    }

To define a specific event listener method (one that does not follow the naming convention)
you need to map the listener method using the event type mapping:

为定义一个特定事件的侦听器方法（不遵循命名规范）你需要使用该事件类型的映射映射侦听器方法：

.. configuration-block::

    .. code-block:: php

        <?php
        class UserListener
        {
            /** @PrePersist */
            public function prePersistHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PostPersist */
            public function postPersistHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreUpdate */
            public function preUpdateHandler(User $user, PreUpdateEventArgs $event) { // ... }

            /** @PostUpdate */
            public function postUpdateHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PostRemove */
            public function postRemoveHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreRemove */
            public function preRemoveHandler(User $user, LifecycleEventArgs $event) { // ... }

            /** @PreFlush */
            public function preFlushHandler(User $user, PreFlushEventArgs $event) { // ... }

            /** @PostLoad */
            public function postLoadHandler(User $user, LifecycleEventArgs $event) { // ... }
        }
    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Entity\User">
                 <entity-listeners>
                    <entity-listener class="UserListener">
                        <lifecycle-callback type="preFlush"      method="preFlushHandler"/>
                        <lifecycle-callback type="postLoad"      method="postLoadHandler"/>

                        <lifecycle-callback type="postPersist"   method="postPersistHandler"/>
                        <lifecycle-callback type="prePersist"    method="prePersistHandler"/>

                        <lifecycle-callback type="postUpdate"    method="postUpdateHandler"/>
                        <lifecycle-callback type="preUpdate"     method="preUpdateHandler"/>

                        <lifecycle-callback type="postRemove"    method="postRemoveHandler"/>
                        <lifecycle-callback type="preRemove"     method="preRemoveHandler"/>
                    </entity-listener>
                </entity-listeners>
                <!-- .... -->
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Entity\User:
          type: entity
          entityListeners:
            UserListener:
              preFlush: [preFlushHandler]
              postLoad: [postLoadHandler]

              postPersist: [postPersistHandler]
              prePersist: [prePersistHandler]

              postUpdate: [postUpdateHandler]
              preUpdate: [preUpdateHandler]

              postRemove: [postRemoveHandler]
              preRemove: [preRemoveHandler]
          # ....

.. note::

    The order of execution of multiple methods for the same event (e.g. multiple @PrePersist) is not guaranteed.
    同一事件的多个方法（如多个 @PrePersist）的执行顺序不被保证。


Entity listeners resolver // 实体侦听器解析器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine invokes the listener resolver to get the listener instance.

Doctrine 调用侦听器解析器来得到一个侦听器实例。

- A resolver allows you register a specific entity listener instance.
- 解析器允许你注册一个特定实体侦听器实例。
- You can also implement your own resolver by extending ``Doctrine\ORM\Mapping\DefaultEntityListenerResolver`` or implementing ``Doctrine\ORM\Mapping\EntityListenerResolver``
- 你也可以实现自己的解析器，通过扩展 ``Doctrine\ORM\Mapping\DefaultEntityListenerResolver``
  或实现 ``Doctrine\ORM\Mapping\EntityListenerResolver``。

Specifying an entity listener instance :

指定一个实体侦听器实例：

.. code-block:: php

    <?php
    // User.php

    /** @Entity @EntityListeners({"UserListener"}) */
    class User
    {
        // ....
    }

    // UserListener.php
    class UserListener
    {
        public function __construct(MyService $service)
        {
            $this->service = $service;
        }

        public function preUpdate(User $user, PreUpdateEventArgs $event)
        {
            $this->service->doSomething($user);
        }
    }

    // register a entity listener.
    $listener = $container->get('user_listener');
    $em->getConfiguration()->getEntityListenerResolver()->register($listener);

Implementing your own resolver :

实现自己的解析器：

.. code-block:: php

    <?php
    class MyEntityListenerResolver extends \Doctrine\ORM\Mapping\DefaultEntityListenerResolver
    {
        public function __construct($container)
        {
            $this->container = $container;
        }

        public function resolve($className)
        {
            // resolve the service id by the given class name;
            $id = 'user_listener';

            return $this->container->get($id);
        }
    }

    // Configure the listener resolver only before instantiating the EntityManager
    $configurations->setEntityListenerResolver(new MyEntityListenerResolver);
    EntityManager::create(.., $configurations, ..);

Load ClassMetadata Event // loadClassMetadata 事件
-------------------------------------------------------

When the mapping information for an entity is read, it is populated
in to a ``ClassMetadataInfo`` instance. You can hook in to this
process and manipulate the instance.

当一个实体的映射信息被读取时，它被填充进一个 ``ClassMetadataInfo`` 实例。你可以 hook 该进程并操作该实例。

.. code-block:: php

    <?php
    $test = new TestEvent();
    $metadataFactory = $em->getMetadataFactory();
    $evm = $em->getEventManager();
    $evm->addEventListener(Events::loadClassMetadata, $test);

    class TestEvent
    {
        public function loadClassMetadata(\Doctrine\ORM\Event\LoadClassMetadataEventArgs $eventArgs)
        {
            $classMetadata = $eventArgs->getClassMetadata();
            $fieldMapping = array(
                'fieldName' => 'about',
                'type' => 'string',
                'length' => 255
            );
            $classMetadata->mapField($fieldMapping);
        }
    }


