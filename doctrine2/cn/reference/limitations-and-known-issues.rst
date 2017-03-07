Limitations and Known Issues // 局限性与已知问题
=====================================================

We try to make using Doctrine2 a very pleasant experience.
Therefore we think it is very important to be honest about the
current limitations to our users. Much like every other piece of
software Doctrine2 is not perfect and far from feature complete.
This section should give you an overview of current limitations of
Doctrine 2 as well as critical known issues that you should know
about.

我们尝试令使用 Doctrine2 非常舒适的体验。因此我们认为非常重要的是要诚实地告诉
我们的用户目前的局限性。非常类似其他软件 Doctrine2 不是完美的且谈不上功能完整。
这部分给你一个 Doctrine 2当前的局限性以及你应该知道的关键的已知问题的概览。

Current Limitations // 当前局限性
---------------------------------------

There is a set of limitations that exist currently which might be
solved in the future. Any of this limitations now stated has at
least one ticket in the Tracker and is discussed for future
releases.

有一组当前存在的将来可能解决的局限性。现在陈述的任何这种局限性至少在 Tracker 中有“一张票（ticket）”
且在未来版本中讨论。

Join-Columns with non-primary keys // 联结列与非主键
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is not possible to use join columns pointing to non-primary keys. Doctrine will think these are the primary
keys and create lazy-loading proxies with the data, which can lead to unexpected results. Doctrine can for performance
reasons not validate the correctness of this settings at runtime but only through the Validate Schema command.

使用联结的列指向非主键是不可能的。Doctrine 将认为这些都是主键并使用这些数据创建懒加载代理，
这可能导致非期望的结果。Doctrine 可以为了性能原因不在运行时验证这些设置的正确性，但是只能通过
Validate Schema 命令。


Mapping Arrays to a Join Table // 映射数组到一个联结表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Related to the previous limitation with "Foreign Keys as
Identifier" you might be interested in mapping the same table
structure as given above to an array. However this is not yet
possible either. See the following example:

使用“外键作为标识符”与前面的限制性相关，你可能对映射与上面给定的相同的表结构到数组有兴趣。
然而这还是不可能的。请看以下示例：

.. code-block:: sql

    CREATE TABLE product (
        id INTEGER,
        name VARCHAR,
        PRIMARY KEY(id)
    );
    
    CREATE TABLE product_attributes (
        product_id INTEGER,
        attribute_name VARCHAR,
        attribute_value VARCHAR,
        PRIMARY KEY (product_id, attribute_name)
    );

This schema should be mapped to a Product Entity as follows:

这个数据库（schema）应该被映射到 Product 实体，如下：

.. code-block:: php

    class Product
    {
        private $id;
        private $name;
        private $attributes = array();
    }

Where the ``attribute_name`` column contains the key and
``attribute_value`` contains the value of each array element in
``$attributes``.

其中 ``attribute_name`` 列包含键，``attribute_value`` 包含
``$attributes`` 中每个数组元素的值。

The feature request for persistence of primitive value arrays
`is described in the DDC-298 ticket <http://www.doctrine-project.org/jira/browse/DDC-298>`_.

原始值数组的持久化功能请求在 `DDC-298 ticket 中被描述 <http://www.doctrine-project.org/jira/browse/DDC-298>`_。

Cascade Merge with Bi-directional Associations // 级联合并与双向的关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are two bugs now that concern the use of cascade merge in combination with bi-directional associations.
Make sure to study the behavior of cascade merge if you are using it:

现在有两个 bugs，涉及在与双向的关联结合中使用级联合并。如果你正在使用它，请确保研究了级联合并的行为：

-  `DDC-875 <http://www.doctrine-project.org/jira/browse/DDC-875>`_ Merge can sometimes add the same entity twice into a collection
-  `DDC-875 <http://www.doctrine-project.org/jira/browse/DDC-875>`_ 合并可能有时添加同样的实体两次到一个集合中
-  `DDC-763 <http://www.doctrine-project.org/jira/browse/DDC-763>`_ Cascade merge on associated entities can insert too many rows through "Persistence by Reachability"
-  `DDC-763 <http://www.doctrine-project.org/jira/browse/DDC-763>`_ 级联合并在关联的实体上通过“可达性的持久化”可能插入很多行

Custom Persisters // 自定义持久器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A Persister in Doctrine is an object that is responsible for the
hydration and write operations of an entity against the database.
Currently there is no way to overwrite the persister implementation
for a given entity, however there are several use-cases that can
benefit from custom persister implementations:

在 Doctrine 中持久器是一个对象，它负责针对数据库的实体的水合和写入操作。当前，对于给定实体没有办法
覆盖持久器实现，但是有几个用例可以从自定义持久器实现中获益：

-  `Add Upsert Support <http://www.doctrine-project.org/jira/browse/DDC-668>`_
-  `添加 Upsert 支持 <http://www.doctrine-project.org/jira/browse/DDC-668>`_
-  `Evaluate possible ways in which stored-procedures can be used <http://www.doctrine-project.org/jira/browse/DDC-445>`_
-  `评估可以使用存储过程的可能方式 <http://www.doctrine-project.org/jira/browse/DDC-445>`_

Persist Keys of Collections // 集合的持久键
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

PHP Arrays are ordered hash-maps and so should be the
``Doctrine\Common\Collections\Collection`` interface. We plan to
evaluate a feature that optionally persists and hydrates the keys
of a Collection instance.

PHP 数组是有序的哈希映射，因此应该是 ``Doctrine\Common\Collections\Collection``
接口。我们计划评估一个功能，可选地持久和水合集合实例的键。

`Ticket DDC-213 <http://www.doctrine-project.org/jira/browse/DDC-213>`_

Mapping many tables to one entity // 映射多表到单实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

It is not possible to map several equally looking tables onto one
entity. For example if you have a production and an archive table
of a certain business concept then you cannot have both tables map
to the same entity.

不可能映射几个看起来同样的表到一个实体上。例如，如果你有 production 和 archive
表的某个业务概念，那么你不能有两张表映射到同样的实体。

Behaviors // 行为
~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine 2 will **never** include a behavior system like Doctrine 1
in the core library. We don't think behaviors add more value than
they cost pain and debugging hell. Please see the many different
blog posts we have written on this topics:

Doctrine 2 将**永不**在核心库中包含类似 Doctrine 1的行为系统。我们不认为行为增加
的价值超过它们痛苦和调试地狱的代价。请参阅我们就此主题撰写的许多不同的博文：

-  `Doctrine2 "Behaviors" in a Nutshell <http://www.doctrine-project.org/2010/02/17/doctrine2-behaviours-nutshell.html>`_
-  `A re-usable Versionable behavior for Doctrine2 <http://www.doctrine-project.org/2010/02/24/doctrine2-versionable.html>`_
-  `Write your own ORM on top of Doctrine2 <http://www.doctrine-project.org/2010/07/19/your-own-orm-doctrine2.html>`_
-  `Doctrine 2 Behavioral Extensions <http://www.doctrine-project.org/2010/11/18/doctrine2-behavioral-extensions.html>`_
-  `Doctrator <https://github.com/pablodip/doctrator`>_

Doctrine 2 has enough hooks and extension points so that **you** can
add whatever you want on top of it. None of this will ever become
core functionality of Doctrine2 however, you will have to rely on
third party extensions for magical behaviors.

Doctrine 2 拥有足够的 hooks 和 扩展点，所以**你**可以在它之上添加任何你想要的。
这将不会变成 Doctrine2 的核心功能，但是你将必须依赖第三方扩展为了奇妙的行为。

Nested Set // 嵌套集
~~~~~~~~~~~~~~~~~~~~~~~~~~~

NestedSet was offered as a behavior in Doctrine 1 and will not be
included in the core of Doctrine 2. However there are already two
extensions out there that offer support for Nested Set with
Doctrine 2:

嵌套集（NestedSet）在 Doctrine 1中作为行为提供且将不会包含在 Doctrine 2的核心中。
但是对于 Doctrine 2 已经有两个扩展提供对嵌套集的支持：

-  `Doctrine2 Hierarchical-Structural Behavior <http://github.com/guilhermeblanco/Doctrine2-Hierarchical-Structural-Behavior>`_
-  `Doctrine2 NestedSet <http://github.com/blt04/doctrine2-nestedset>`_

Known Issues // 已知问题
------------------------------

The Known Issues section describes critical/blocker bugs and other
issues that are either complicated to fix, not fixable due to
backwards compatibility issues or where no simple fix exists (yet).
We don't plan to add every bug in the tracker there, just those
issues that can potentially cause nightmares or pain of any sort.

已经问题部分描述关键/阻滞 bugs 和其他问题，难以修复、不可固定的由于向后兼容性问题或
没有简单修复存在的问题。我们没有计划添加每个 bug 在 tracker 中，仅仅这些问题可能潜在地导致
噩梦或任何种类的痛苦。

See the Open Bugs on Jira for more details on `bugs, improvement and feature
requests
<http://www.doctrine-project.org/jira/secure/IssueNavigator.jspa?reset=true&mode=hide&pid=10032&resolution=-1&sorter/field=updated&sorter/order=DESC>`_.

在 Jira 上查看打开的 Bugs 以获得更多有关 `bugs, 改善和功能请求 <http://www.doctrine-project.org/jira/secure/IssueNavigator.jspa?reset=true&mode=hide&pid=10032&resolution=-1&sorter/field=updated&sorter/order=DESC>`_ 的详情。

Identifier Quoting and Legacy Databases // 标识符 Quoting 和传统数据库
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For compatibility reasons between all the supported vendors and
edge case problems Doctrine 2 does **NOT** do automatic identifier
quoting. This can lead to problems when trying to get
legacy-databases to work with Doctrine 2.

出于兼容性原因在所有受支持的供应商和边缘情况之间的问题，Doctrine 2 不做自动地标识符 quoting。
这可能导致问题，当尝试传统数据库与 Doctrine 2一起工作时。

-  You can quote column-names as described in the
   :doc:`Basic-Mapping <basic-mapping>` section.
-  你可以 quote 列名如同在 :doc:`基础映射 <basic-mapping>` 中描述的那样。
-  You cannot quote join column names.
-  你不能 quote 联结列名。
-  You cannot use non [a-zA-Z0-9\_]+ characters, they will break
   several SQL statements.
-  你不能使用非 [a-zA-Z0-9\_]+ 字符，它们将中断某些 SQL 语句。

Having problems with these kind of column names? Many databases
support all CRUD operations on views that semantically map to
certain tables. You can create views for all your problematic
tables and column names to avoid the legacy quoting nightmare.

这些类型的列名有问题吗？多数数据库支持在语义化映射到某些表的视图（view）上的所有 CRUD 操作。
你可以为所有你的问题表和列创建视图（view）以避免传统的 quoting 噩梦。

Microsoft SQL Server and Doctrine "datetime" // Microsoft SQL Server 和 Doctrine "datetime"
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine assumes that you use ``DateTime2`` data-types. If your legacy database contains DateTime
datatypes then you have to add your own data-type (see Basic Mapping for an example).

Doctrine 假设你使用 ``DateTime2`` 数据类型。如果你的传统数据库包含 DateTime 数据类型，
那么你必须添加你自己的数据类型（查看基础映射的例子）。

MySQL with MyISAM tables // MySQL 与 MyISAM 表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine cannot provide atomic operations when calling ``EntityManager#flush()`` if one
of the tables involved uses the storage engine MyISAM. You must use InnoDB or
other storage engines that support transactions if you need integrity.

如果一个表涉及使用 MyISAM 存储引擎，当调用 ``EntityManager#flush()`` 时，
Doctrine 不能提供原子（atomic）操作。你必须使用 InnoDB 或其他支持事务的存储引擎，
如果你需要完整性的话。

Entities, Proxies and Reflection // 实体、代理和反射
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using methods for Reflection on entities can be prone to error, when the entity
is actually a proxy the following methods will not work correctly:

对于在实体上的反射使用方法可能易于遭受错误，当实体事实上是代理时，以下方法将不能正常工作：

- ``new ReflectionClass``
- ``new ReflectionObject``
- ``get_class()``
- ``get_parent_class()``

This is why ``Doctrine\Common\Util\ClassUtils`` class exists that has similar
methods, which resolve the proxy problem beforehand.

这是为何 ``Doctrine\Common\Util\ClassUtils`` 类存在类似的方法，事先解决该代理问题。

.. code-block:: php

    <?php
    use Doctrine\Common\Util\ClassUtils;

    $bookProxy = $entityManager->getReference('Acme\Book');

    $reflection = ClassUtils::newReflectionClass($bookProxy);
    $class = ClassUtils::getClass($bookProxy)¸
