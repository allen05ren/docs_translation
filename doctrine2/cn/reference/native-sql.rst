Native SQL // 原生 SQL
==============================

With ``NativeQuery`` you can execute native SELECT SQL statements
and map the results to Doctrine entities or any other result format
supported by Doctrine.

使用 ``NativeQuery`` 你可以执行原生的 SELECT SQL 语句并映射结果至 Doctrine 实体或
任何其他 Doctrine 所支持的结果格式。

In order to make this mapping possible, you need to describe
to Doctrine what columns in the result map to which entity property.
This description is represented by a ``ResultSetMapping`` object.

为了让该映射成为可能，你需要向 Doctrine 描述结果集中的哪一列映射到实体的哪个属性上。
这个描述通过一个 ``ResultSetMapping`` 对象表示。

With this feature you can map arbitrary SQL code to objects, such as highly
vendor-optimized SQL or stored-procedures.

使用此特性你可以映射任意 SQL 代码到对象中，例如高度供应商（译注：某数据库供应商）优化的 SQL 或
存储过程。

Writing ``ResultSetMapping`` from scratch is complex, but there is a convenience
wrapper around it called a ``ResultSetMappingBuilder``. It can generate
the mappings for you based on Entities and even generates the ``SELECT``
clause based on this information for you.

从头开始写 ``ResultSetMapping`` 是复杂的，但是有一个便利的围绕它的封装器叫 ``ResultSetMappingBuilder``。
它可以基于实体为你生成映射甚至于基于此信息为你生成 ``SELECT`` 子句。

.. note::

    If you want to execute DELETE, UPDATE or INSERT statements
    the Native SQL API cannot be used and will probably throw errors.
    Use ``EntityManager#getConnection()`` to access the native database
    connection and call the ``executeUpdate()`` method for these
    queries.

    如果你希望执行 DELETE、UPDATE 或 INSERT 语句，原生 SQL API 不能被使用并将很可能抛出错误。
    对于这些请求，请使用 ``EntityManager#getConnection()`` 访问原生数据库连接和调用
    ``executeUpdate()`` 方法。

The NativeQuery class // NativeQuery 类
-----------------------------------------------

To create a ``NativeQuery`` you use the method
``EntityManager#createNativeQuery($sql, $resultSetMapping)``. As you can see in
the signature of this method, it expects 2 ingredients: The SQL you want to
execute and the ``ResultSetMapping`` that describes how the results will be
mapped.

使用 ``EntityManager#createNativeQuery($sql, $resultSetMapping)``方法，以创建一个
``NativeQuery``。正如你所见，在此方法的签名（signature）中期望两个参数：你希望执行的 SQL和
描述如何映射结果的 ``ResultSetMapping``。

Once you obtained an instance of a ``NativeQuery``, you can bind parameters to
it with the same API that ``Query`` has and execute it.

一旦你获得了一个 ``NativeQuery`` 实例，你可以用 ``Query`` 具有的相同 API 绑定参数到它并执行它。

.. code-block:: php

    <?php
    use Doctrine\ORM\Query\ResultSetMapping;

    $rsm = new ResultSetMapping();
    // build rsm here

    $query = $entityManager->createNativeQuery('SELECT id, name, discr FROM users WHERE name = ?', $rsm);
    $query->setParameter(1, 'romanb');

    $users = $query->getResult();

ResultSetMappingBuilder // 结果集映射构建器
------------------------------------------------

An easy start into ResultSet mapping is the ``ResultSetMappingBuilder`` object.
This has several benefits:

进入结果集映射的一个简单开局是 ``ResultSetMappingBuilder`` 对象。这有几个好处：


- The builder takes care of automatically updating your ``ResultSetMapping``
  when the fields or associations change on the metadata of an entity.
- 当在实体的元数据的字段或关联改变时，构建器自动地负责更新你的 ``ResultSetMapping``。
- You can generate the required ``SELECT`` expression for a builder
  by converting it to a string.
- 通过将它转换为一个字符串，你可以为构建器生成需要的 ``SELECT`` 表达式。
- The API is much simpler than the usual ``ResultSetMapping`` API.
- 该 API 相比于通常的 ``ResultSetMapping`` API 更加简单。

One downside is that the builder API does not yet support entities
with inheritance hierachies.

一个缺点是，构建器 API 还不支持具有继承层次结构的实体。

.. code-block:: php

    <?php

    use Doctrine\ORM\Query\ResultSetMappingBuilder;

    $sql = "SELECT u.id, u.name, a.id AS address_id, a.street, a.city " . 
           "FROM users u INNER JOIN address a ON u.address_id = a.id";

    $rsm = new ResultSetMappingBuilder($entityManager);
    $rsm->addRootEntityFromClassMetadata('MyProject\User', 'u');
    $rsm->addJoinedEntityFromClassMetadata('MyProject\Address', 'a', 'u', 'address', array('id' => 'address_id'));

The builder extends the ``ResultSetMapping`` class and as such has all the functionality of it as well.

构建器扩展了 ``ResultSetMapping`` 类，并且拥有了它所具有的全部功能。

.. versionadded:: 2.4

Starting with Doctrine ORM 2.4 you can generate the ``SELECT`` clause
from a ``ResultSetMappingBuilder``. You can either cast the builder
object to ``(string)`` and the DQL aliases are used as SQL table aliases
or use the ``generateSelectClause($tableAliases)`` method and pass
a mapping from DQL alias (key) to SQL alias (value)

从 Doctrine ORM 2.4 开始你可以从 ``ResultSetMappingBuilder`` 生成 ``SELECT`` 子句。
你可以将构造器对象转换为 ``(string)`` 并将 DQL 别名用于 SQL 表别名或者使用
``generateSelectClause($tableAliases)`` 方法并传递一个从 DQL 别名（键）到 SQL 别名（值）的映射。

.. code-block:: php

    <?php

    $selectClause = $builder->generateSelectClause(array(
        'u' => 't1',
        'g' => 't2'
    ));
    $sql = "SELECT " . $selectClause . " FROM users t1 JOIN groups t2 ON t1.group_id = t2.id";


The ResultSetMapping // 结果集映射
----------------------------------------

Understanding the ``ResultSetMapping`` is the key to using a
``NativeQuery``. A Doctrine result can contain the following
components:

理解 ``ResultSetMapping`` 是使用 ``NativeQuery`` 的关键。Doctrine 结果
可以包含以下组件：

-  Entity results. These represent root result elements.
-  实体结果。这些代表根结果元素。
-  Joined entity results. These represent joined entities in
   associations of root entity results.
-  联结的（joined）实体结果。这些代表根实体结果的关联中的联结实体。
-  Field results. These represent a column in the result set that
   maps to a field of an entity. A field result always belongs to an
   entity result or joined entity result.
-  字段结果。这些代表在映射到实体字段的结果集中的一个列。
-  Scalar results. These represent scalar values in the result set
   that will appear in each result row. Adding scalar results to a
   ResultSetMapping can also cause the overall result to become
   **mixed** (see DQL - Doctrine Query Language) if the same
   ResultSetMapping also contains entity results.
-  标量结果。这些代表在将出现在每个结果行中的结果集中的标量值。添加标量结果到一个
   ResultSetMapping 中也可以导致整个结果变成**混合的（mixed）**（查看 DQL - Doctrine
   查询语言），如果同样的 ResultSetMapping 也包含实体结果的话。
-  Meta results. These represent columns that contain
   meta-information, such as foreign keys and discriminator columns.
   When querying for objects (``getResult()``), all meta columns of
   root entities or joined entities must be present in the SQL query
   and mapped accordingly using ``ResultSetMapping#addMetaResult``.
-  元（meta）结果。这些代表包含元信息（meta-information）的列，比如外键和鉴别器列。
   当对于对象（``getResult()``）查询，所有根实体的元数据列或联结的实体必须出现在 SQL 查询中且
   使用 ``ResultSetMapping#addMetaResult`` 相应地映射。

.. note::

    It might not surprise you that Doctrine uses
    ``ResultSetMapping`` internally when you create DQL queries. As
    the query gets parsed and transformed to SQL, Doctrine fills a
    ``ResultSetMapping`` that describes how the results should be
    processed by the hydration routines.

    当你创建 DQL 查询时 Doctrine 内部使用 ``ResultSetMapping``，可能并不会让你感到惊讶。
    因为查询获得解析并转换为 SQL， Doctrine 填充 ``ResultSetMapping``,它描述了结果应该如何
    通过水合例程被处理。

We will now look at each of the result types that can appear in a
ResultSetMapping in detail.

我现在将看一看每一个可以出现在 ResultSetMapping 中的结果类型的详情。

Entity results // 实体结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An entity result describes an entity type that appears as a root
element in the transformed result. You add an entity result through
``ResultSetMapping#addEntityResult()``. Let's take a look at the
method signature in detail:

实体结果描述了一个作为根元素出现在已转换的结果中的实体类型。你可以通过
``ResultSetMapping#addEntityResult()`` 添加一个实体结果。让我们看一看方法签名的详情：

.. code-block:: php

    <?php
    /**
     * Adds an entity result to this ResultSetMapping.
     * 添加一个实体结果到此 ResultSetMapping
     *
     * @param string $class The class name of the entity.
     *                      实体的类名。
     * @param string $alias The alias for the class. The alias must be unique among all entity
     *                      results or joined entity results within this ResultSetMapping.
     *                      对应类的别名。该别名必须在该 ResultSetMapping 内的所有实体结果或联结实体结果中是唯一的。
     */
    public function addEntityResult($class, $alias)

The first parameter is the fully qualified name of the entity
class. The second parameter is some arbitrary alias for this entity
result that must be unique within a ``ResultSetMapping``. You use
this alias to attach field results to the entity result. It is very
similar to an identification variable that you use in DQL to alias
classes or relationships.

第一个参数是实体类的完全限定类名。第二个参数是该实体结果的一些任意别名，在 ``ResultSetMapping`` 内它必须是唯一的。
你使用此别名附加字段结果到该实体结果。这非常类似于你在 DQL 中用于别名类或关联的标识变量。

An entity result alone is not enough to form a valid
``ResultSetMapping``. An entity result or joined entity result
always needs a set of field results, which we will look at soon.

单独的实体结果不足以形成一个有效的 ``ResultSetMapping``。实体结果或联结的实体结果始终需要
一组字段结果，我们很快将会看到。

Joined entity results // 联结的实体结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A joined entity result describes an entity type that appears as a
joined relationship element in the transformed result, attached to
a (root) entity result. You add a joined entity result through
``ResultSetMapping#addJoinedEntityResult()``. Let's take a look at
the method signature in detail:

联结的实体结果描述作为联结的关联元素出现在已转换的结果中的实体类型，附加到一个（根）实体结果。
通过 ``ResultSetMapping#addJoinedEntityResult()`` 你可以添加一个联结的实体结果。
让我们看一看此方法的签名的详情：

.. code-block:: php

    <?php
    /**
     * Adds a joined entity result.
     * 添加一个联结的实体结果。
     *
     * @param string $class The class name of the joined entity.
     *                      联结的实体的类名
     * @param string $alias The unique alias to use for the joined entity.
     *                      用于联结的实体的唯一别名。
     * @param string $parentAlias The alias of the entity result that is the parent of this joined result.
     *                            此联结的结果的父实体结果的别名。
     * @param object $relation The association field that connects the parent entity result with the joined entity result.
     *                         连接拥有此联结的实体结果的父实体结果的关联字段。
     */
    public function addJoinedEntityResult($class, $alias, $parentAlias, $relation)

The first parameter is the class name of the joined entity. The
second parameter is an arbitrary alias for the joined entity that
must be unique within the ``ResultSetMapping``. You use this alias
to attach field results to the entity result. The third parameter
is the alias of the entity result that is the parent type of the
joined relationship. The fourth and last parameter is the name of
the field on the parent entity result that should contain the
joined entity result.

第一个参数是联结的实体的类名。第二个参数是该联结的实体的一个任意的别名，在 ``ResultSetMapping``
内它必须是唯一的。使用此别名附加字段结果到该实体结果。第三个参数是该联结的关联的父类型的实体结果的别名。
第四个即最后一个参数是在应该包含该联结的实体结果的父实体结果上的字段名。

Field results // 字段结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A field result describes the mapping of a single column in a SQL
result set to a field in an entity. As such, field results are
inherently bound to entity results. You add a field result through
``ResultSetMapping#addFieldResult()``. Again, let's examine the
method signature in detail:

字段结果描述在 SQL 结果集中的单一列到实体中的字段的映射。同样地，字段结果本质上是
绑定到实体结果的。你可以通过 ``ResultSetMapping#addFieldResult()`` 添加一个字段
结果。再一次让我们查看该方法签名的详情：

.. code-block:: php

    <?php
    /**
     * Adds a field result that is part of an entity result or joined entity result.
     * 添加一个字段结果，它是实体结果或联结的实体结果的一部分。
     *
     * @param string $alias The alias of the entity result or joined entity result.
     *                      实体结果或联结的实体结果的别名。
     * @param string $columnName The name of the column in the SQL result set.
     *                           在 SQL 结果集中的列名。
     * @param string $fieldName The name of the field on the (joined) entity.
     *                          在（联结的）实体上的字段名。
     */
    public function addFieldResult($alias, $columnName, $fieldName)

The first parameter is the alias of the entity result to which the
field result will belong. The second parameter is the name of the
column in the SQL result set. Note that this name is case
sensitive, i.e. if you use a native query against Oracle it must be
all uppercase. The third parameter is the name of the field on the
entity result identified by ``$alias`` into which the value of the
column should be set.

第一个参数是此字段结果将所属的实体结果的别名。第二个参数是在 SQL 结果集中的列名。
注意这个名字是大小写敏感的，例如如果你使用针对 Oracle 的原生查询它必须是全部大写的。
第三个参数是由 ``$alias`` 所标识的实体结果上应被设置的列为该值的字段名。

Scalar results // 标量结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A scalar result describes the mapping of a single column in a SQL
result set to a scalar value in the Doctrine result. Scalar results
are typically used for aggregate values but any column in the SQL
result set can be mapped as a scalar value. To add a scalar result
use ``ResultSetMapping#addScalarResult()``. The method signature in
detail:

标量结果描述 SQL 结果集中的单一列到 Doctrine 结果中的标量值的映射。标量结果典型地
被用于聚合值，但在 SQL 结果集中的任何列都可以被映射为一个标量值。使用 ``ResultSetMapping#addScalarResult()``
可以添加一个标量结果。此方法的签名详情：

.. code-block:: php

    <?php
    /**
     * Adds a scalar result mapping.
     * 添加一个标量值映射
     *
     * @param string $columnName The name of the column in the SQL result set.
     *                           在 SQL 结果集中的列名。
     * @param string $alias The result alias with which the scalar result should be placed in the result structure.
     *                      应该被放置在结果结构中的标量结果的结果别名。
     */
    public function addScalarResult($columnName, $alias)

The first parameter is the name of the column in the SQL result set
and the second parameter is the result alias under which the value
of the column will be placed in the transformed Doctrine result.

第一个参数是在 SQL 结果集中的列名。第二个参数是列的值将被放置在已转换的 Doctrine 结果中
的结果别名。

Meta results // 元（Meta）结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

A meta result describes a single column in a SQL result set that
is either a foreign key or a discriminator column. These columns
are essential for Doctrine to properly construct objects out of SQL
result sets. To add a column as a meta result use
``ResultSetMapping#addMetaResult()``. The method signature in
detail:

元（meta）结果描述在 SQL 结果及中的单一列，它是一个外键或一个鉴别器列。这些列本质上是为了
Doctrine 正确地在 SQL 结果集之外构造对象的。使用 ``ResultSetMapping#addMetaResult()``
可以添加一个列作为元结果。该方法的签名详情：

.. code-block:: php

    <?php
    /**
     * Adds a meta column (foreign key or discriminator column) to the result set.
     * 添加一个元列（外键或鉴别器列）到结果集。
     *
     * @param string  $alias
     * @param string  $columnAlias
     * @param string  $columnName
     * @param boolean $isIdentifierColumn
     */
    public function addMetaResult($alias, $columnAlias, $columnName, $isIdentifierColumn = false)

The first parameter is the alias of the entity result to which the
meta column belongs. A meta result column (foreign key or
discriminator column) always belongs to an entity result. The
second parameter is the column alias/name of the column in the SQL
result set and the third parameter is the column name used in the
mapping.The fourth parameter should be set to true in case the primary key
of the entity is the foreign key you're adding.

第一个参数是元列所属于的实体结果的别名。一个元结果列（外键或鉴别器列）始终属于一个实体结果。
第二个参数是在 SQL 结果集中列的别名/列名。第三个参数是在映射中使用的列名。
第四个参数应该被设置为 true，在实体的主键是你z正在添加的外键的情况。

Discriminator Column // 鉴别器列
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When joining an inheritance tree you have to give Doctrine a hint
which meta-column is the discriminator column of this tree.

当联结一个层次结构树时你必须给 Doctrine 一个提示，哪一个元列（meta-column）是此数的
鉴别器列。

.. code-block:: php

    <?php
    /**
     * Sets a discriminator column for an entity result or joined entity result.
     * 为实体结果或联结的实体结果设置一个鉴别器列。
     * The discriminator column will be used to determine the concrete class name to
     * instantiate.
     * 该鉴别器列将被用于确定哪一个具体的类名以实例化。
     *
     * @param string $alias The alias of the entity result or joined entity result the discriminator
     *                      column should be used for.
     *                      实体结果或联结的实体结果的鉴别器列应该被用于的别名。
     * @param string $discrColumn The name of the discriminator column in the SQL result set.
     *                            在 SQL 结果集中的鉴别器列的名称。
     */
    public function setDiscriminatorColumn($alias, $discrColumn)

Examples // 示例
~~~~~~~~~~~~~~~~~~~~~~~

Understanding a ResultSetMapping is probably easiest through
looking at some examples.

通过查看一些例子，理解 ResultSetMapping 或许是最容易的。

First a basic example that describes the mapping of a single
entity.

首先，一个基础例子描述单一实体的映射。

.. code-block:: php

    <?php
    // Equivalent DQL query: "select u from User u where u.name=?1"
    // 等价的 DQL 查询："select u from User u where u.name=?1"
    // User owns no associations.
    // User 不拥有关联。
    $rsm = new ResultSetMapping;
    $rsm->addEntityResult('User', 'u');
    $rsm->addFieldResult('u', 'id', 'id');
    $rsm->addFieldResult('u', 'name', 'name');
    
    $query = $this->_em->createNativeQuery('SELECT id, name FROM users WHERE name = ?', $rsm);
    $query->setParameter(1, 'romanb');
    
    $users = $query->getResult();

The result would look like this:

结果将看上去这样：

.. code-block:: php

    array(
        [0] => User (Object)
    )

Note that this would be a partial object if the entity has more
fields than just id and name. In the example above the column and
field names are identical but that is not necessary, of course.
Also note that the query string passed to createNativeQuery is
**real native SQL**. Doctrine does not touch this SQL in any way.

注意如果实体拥有不止是 id 和 name 字段，这将是一个部分实体。在上述例子中列和字段的名称是完全一样的，
当然，但那不是必须的。还注意传递给 createNativeQuery 的查询字符串是**真实的原生 SQL**。
Doctrine 不能以任何方式触及此 SQL。

In the previous basic example, a User had no relations and the
table the class is mapped to owns no foreign keys. The next example
assumes User has a unidirectional or bidirectional one-to-one
association to a CmsAddress, where the User is the owning side and
thus owns the foreign key.

在前面的基础例子中，User 没有关联且类被映射的表不拥有外键。下一个例子假设 User 拥有一个单向的或双向的
one-to-one 关联到 CmsAddress，其中 User 是 owning 侧并且因此拥有外键。

.. code-block:: php

    <?php
    // Equivalent DQL query: "select u from User u where u.name=?1"
    // 等价的 DQL 查询："select u from User u where u.name=?1"
    // User owns an association to an Address but the Address is not loaded in the query.
    // User 拥有一个关联到 Address，但 Address 不被加载在此查询中。
    $rsm = new ResultSetMapping;
    $rsm->addEntityResult('User', 'u');
    $rsm->addFieldResult('u', 'id', 'id');
    $rsm->addFieldResult('u', 'name', 'name');
    $rsm->addMetaResult('u', 'address_id', 'address_id');
    
    $query = $this->_em->createNativeQuery('SELECT id, name, address_id FROM users WHERE name = ?', $rsm);
    $query->setParameter(1, 'romanb');
    
    $users = $query->getResult();

Foreign keys are used by Doctrine for lazy-loading purposes when
querying for objects. In the previous example, each user object in
the result will have a proxy (a "ghost") in place of the address
that contains the address\_id. When the ghost proxy is accessed, it
loads itself based on this key.

当查询对象时，外键经 Doctrine 被用于懒加载目的。在前面的例子中，在结果中的每一个 user
对象将拥有一个代理（一个“幽灵”）替代包含 address\_id 的 address。当该“幽灵”代理被访问时，
它基于此键加载自身。

Consequently, associations that are *fetch-joined* do not require
the foreign keys to be present in the SQL result set, only
associations that are lazy.

因此，*fetch-joined* 的关联不需要外键出现在 SQL 结果集中，仅需懒加载的关联。

.. code-block:: php

    <?php
    // Equivalent DQL query: "select u from User u join u.address a WHERE u.name = ?1"
    // 等价的 DQL 查询："select u from User u join u.address a WHERE u.name = ?1"
    // User owns association to an Address and the Address is loaded in the query.
    // User 拥有关联到 Address 且 Address 被加载在此查询中。
    $rsm = new ResultSetMapping;
    $rsm->addEntityResult('User', 'u');
    $rsm->addFieldResult('u', 'id', 'id');
    $rsm->addFieldResult('u', 'name', 'name');
    $rsm->addJoinedEntityResult('Address' , 'a', 'u', 'address');
    $rsm->addFieldResult('a', 'address_id', 'id');
    $rsm->addFieldResult('a', 'street', 'street');
    $rsm->addFieldResult('a', 'city', 'city');
    
    $sql = 'SELECT u.id, u.name, a.id AS address_id, a.street, a.city FROM users u ' .
           'INNER JOIN address a ON u.address_id = a.id WHERE u.name = ?';
    $query = $this->_em->createNativeQuery($sql, $rsm);
    $query->setParameter(1, 'romanb');
    
    $users = $query->getResult();

In this case the nested entity ``Address`` is registered with the
``ResultSetMapping#addJoinedEntityResult`` method, which notifies
Doctrine that this entity is not hydrated at the root level, but as
a joined entity somewhere inside the object graph. In this case we
specify the alias 'u' as third parameter and ``address`` as fourth
parameter, which means the ``Address`` is hydrated into the
``User::$address`` property.

在此例中，使用 ``ResultSetMapping#addJoinedEntityResult`` 方法注册了一个
嵌套的实体 ``Address``，这通知 Doctrine 该实体不被水合在根层级，但是作为
在该对象图内部的某个地方。在此例中我们指定了别名“u”作为第三个参数以及  ``address``
作为第四个参数，这意味着 ``Address`` 被水合进 ``User::$address`` 属性。

If a fetched entity is part of a mapped hierarchy that requires a
discriminator column, this column must be present in the result set
as a meta column so that Doctrine can create the appropriate
concrete type. This is shown in the following example where we
assume that there are one or more subclasses that extend User and
either Class Table Inheritance or Single Table Inheritance is used
to map the hierarchy (both use a discriminator column).

如果一个已取回（fetched）的实体是一个需要鉴别器列的映射层次结构的一部分，此列必须
出现在结果集中作为一个元（meta）列，因此 Doctrine 可以创建一个合适的具体的类型。
这展示在下面的例子中，其中我们假设由一个或多个子类扩展了 User 并且类表继承或单一表继承
被用于映射该层次结构（两者都使用了鉴别器列）。

.. code-block:: php

    <?php
    // Equivalent DQL query: "select u from User u where u.name=?1"
    // 等价的 DQL 查询："select u from User u where u.name=?1"
    // User is a mapped base class for other classes. User owns no associations.
    // User 是一个映射的基础类用于其他类。User 不拥有关联。
    $rsm = new ResultSetMapping;
    $rsm->addEntityResult('User', 'u');
    $rsm->addFieldResult('u', 'id', 'id');
    $rsm->addFieldResult('u', 'name', 'name');
    $rsm->addMetaResult('u', 'discr', 'discr'); // discriminator column
    $rsm->setDiscriminatorColumn('u', 'discr');
    
    $query = $this->_em->createNativeQuery('SELECT id, name, discr FROM users WHERE name = ?', $rsm);
    $query->setParameter(1, 'romanb');
    
    $users = $query->getResult();

Note that in the case of Class Table Inheritance, an example as
above would result in partial objects if any objects in the result
are actually a subtype of User. When using DQL, Doctrine
automatically includes the necessary joins for this mapping
strategy but with native SQL it is your responsibility.

注意在类表继承的情况中，如果在结果中的任何对象事实上都是 User 的子类型，上述例子将导致部分对象。
当使用 DQL 时，Doctrine 自动地为此映射策略包括需要的联结，但是使用原生的 SQL 它是你的职责。

Named Native Query // 命名的原生查询
-----------------------------------------

You can also map a native query using a named native query mapping.

你也可以使用命名原生查询映射映射一个原生查询。

To achieve that, you must describe the SQL resultset structure
using named native query (and sql resultset mappings if is a several resultset mappings).

为了实现这一点，你必须使用命名的原生查询（以及 sql 结果集映射，如果是数个结果映射的话）描述此 SQL 结果集的结构。

Like named query, a named native query can be defined at class level or in a XML or YAML file.

类似命名查询，一个命名原生查询可以在类级别上或 XML 或 YAML 文件中定义。

A resultSetMapping parameter is defined in @NamedNativeQuery,
it represents the name of a defined @SqlResultSetMapping.

resultSetMapping 参数被定义在 @NamedNativeQuery 中，它代表了已定义的
@SqlResultSetMapping的名字。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;
        /**
         * @NamedNativeQueries({
         *      @NamedNativeQuery(
         *          name            = "fetchMultipleJoinsEntityResults",
         *          resultSetMapping= "mappingMultipleJoinsEntityResults",
         *          query           = "SELECT u.id AS u_id, u.name AS u_name, u.status AS u_status, a.id AS a_id, a.zip AS a_zip, a.country AS a_country, COUNT(p.phonenumber) AS numphones FROM users u INNER JOIN addresses a ON u.id = a.user_id INNER JOIN phonenumbers p ON u.id = p.user_id GROUP BY u.id, u.name, u.status, u.username, a.id, a.zip, a.country ORDER BY u.username"
         *      ),
         * })
         * @SqlResultSetMappings({
         *      @SqlResultSetMapping(
         *          name    = "mappingMultipleJoinsEntityResults",
         *          entities= {
         *              @EntityResult(
         *                  entityClass = "__CLASS__",
         *                  fields      = {
         *                      @FieldResult(name = "id",       column="u_id"),
         *                      @FieldResult(name = "name",     column="u_name"),
         *                      @FieldResult(name = "status",   column="u_status"),
         *                  }
         *              ),
         *              @EntityResult(
         *                  entityClass = "Address",
         *                  fields      = {
         *                      @FieldResult(name = "id",       column="a_id"),
         *                      @FieldResult(name = "zip",      column="a_zip"),
         *                      @FieldResult(name = "country",  column="a_country"),
         *                  }
         *              )
         *          },
         *          columns = {
         *              @ColumnResult("numphones")
         *          }
         *      )
         *})
         */
         class User
        {
            /** @Id @Column(type="integer") @GeneratedValue */
            public $id;

            /** @Column(type="string", length=50, nullable=true) */
            public $status;

            /** @Column(type="string", length=255, unique=true) */
            public $username;

            /** @Column(type="string", length=255) */
            public $name;

            /** @OneToMany(targetEntity="Phonenumber") */
            public $phonenumbers;

            /** @OneToOne(targetEntity="Address") */
            public $address;

            // ....
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Model\User">
                <named-native-queries>
                    <named-native-query name="fetchMultipleJoinsEntityResults" result-set-mapping="mappingMultipleJoinsEntityResults">
                        <query>SELECT u.id AS u_id, u.name AS u_name, u.status AS u_status, a.id AS a_id, a.zip AS a_zip, a.country AS a_country, COUNT(p.phonenumber) AS numphones FROM users u INNER JOIN addresses a ON u.id = a.user_id INNER JOIN phonenumbers p ON u.id = p.user_id GROUP BY u.id, u.name, u.status, u.username, a.id, a.zip, a.country ORDER BY u.username</query>
                    </named-native-query>
                </named-native-queries>
                <sql-result-set-mappings>
                    <sql-result-set-mapping name="mappingMultipleJoinsEntityResults">
                        <entity-result entity-class="__CLASS__">
                            <field-result name="id" column="u_id"/>
                            <field-result name="name" column="u_name"/>
                            <field-result name="status" column="u_status"/>
                        </entity-result>
                        <entity-result entity-class="Address">
                            <field-result name="id" column="a_id"/>
                            <field-result name="zip" column="a_zip"/>
                            <field-result name="country" column="a_country"/>
                        </entity-result>
                        <column-result name="numphones"/>
                    </sql-result-set-mapping>
                </sql-result-set-mappings>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Model\User:
          type: entity
          namedNativeQueries:
            fetchMultipleJoinsEntityResults:
              name: fetchMultipleJoinsEntityResults
              resultSetMapping: mappingMultipleJoinsEntityResults
              query: SELECT u.id AS u_id, u.name AS u_name, u.status AS u_status, a.id AS a_id, a.zip AS a_zip, a.country AS a_country, COUNT(p.phonenumber) AS numphones FROM users u INNER JOIN addresses a ON u.id = a.user_id INNER JOIN phonenumbers p ON u.id = p.user_id GROUP BY u.id, u.name, u.status, u.username, a.id, a.zip, a.country ORDER BY u.username
          sqlResultSetMappings:
            mappingMultipleJoinsEntityResults:
              name: mappingMultipleJoinsEntityResults
              columnResult:
                0:
                  name: numphones
              entityResult:
                0:
                  entityClass: __CLASS__
                  fieldResult:
                    0:
                      name: id
                      column: u_id
                    1:
                      name: name
                      column: u_name
                    2:
                      name: status
                      column: u_status
                1:
                  entityClass: Address
                  fieldResult:
                    0:
                      name: id
                      column: a_id
                    1:
                      name: zip
                      column: a_zip
                    2:
                      name: country
                      column: a_country


Things to note:

注意事项：

    - The resultset mapping declares the entities retrieved by this native query.
    - 结果集映射声明该实体由此原生查询取回。
    - Each field of the entity is bound to a SQL alias (or column name).
    - 该实体的每个字段被绑定到一个 SQL 别名（或列名）。
    - All fields of the entity including the ones of subclasses
      and the foreign key columns of related entities have to be present in the SQL query.
    - 该实体的所有字段包括子类的所有字段以及关联的实体的外键列必须出现在 SQL 查询中。
    - Field definitions are optional provided that they map to the same
      column name as the one declared on the class property.
    - 字段定义是可选的，它们映射到相同列名作为在此类属性上的一个声明。
    - ``__CLASS__`` is an alias for the mapped class
    - ``__CLASS__`` 对于此映射类的一个别名。


In the above example,
the ``fetchJoinedAddress`` named query use the joinMapping result set mapping.
This mapping returns 2 entities, User and Address, each property is declared and associated to a column name,
actually the column name retrieved by the query.

在上述例子中，``fetchJoinedAddress`` 命名查询使用 joinMapping 结果集映射。
此映射返回两个实体，User 和 Address，每个属性被声明和关联至一个列名，实际上列名
由该查询取回。

Let's now see an implicit declaration of the property / column.

现在让我们看一个隐式声明的属性/列。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;
            /**
             * @NamedNativeQueries({
             *      @NamedNativeQuery(
             *          name                = "findAll",
             *          resultSetMapping    = "mappingFindAll",
             *          query               = "SELECT * FROM addresses"
             *      ),
             * })
             * @SqlResultSetMappings({
             *      @SqlResultSetMapping(
             *          name    = "mappingFindAll",
             *          entities= {
             *              @EntityResult(
             *                  entityClass = "Address"
             *              )
             *          }
             *      )
             * })
             */
           class Address
           {
                /**  @Id @Column(type="integer") @GeneratedValue */
                public $id;

                /** @Column() */
                public $country;

                /** @Column() */
                public $zip;

                /** @Column()*/
                public $city;

                // ....
            }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Model\Address">
                <named-native-queries>
                    <named-native-query name="findAll" result-set-mapping="mappingFindAll">
                        <query>SELECT * FROM addresses</query>
                    </named-native-query>
                </named-native-queries>
                <sql-result-set-mappings>
                    <sql-result-set-mapping name="mappingFindAll">
                        <entity-result entity-class="Address"/>
                    </sql-result-set-mapping>
                </sql-result-set-mappings>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Model\Address:
          type: entity
          namedNativeQueries:
            findAll:
              resultSetMapping: mappingFindAll
              query: SELECT * FROM addresses
          sqlResultSetMappings:
            mappingFindAll:
              name: mappingFindAll
              entityResult:
                address:
                  entityClass: Address


In this example, we only describe the entity member of the result set mapping.
The property / column mappings is done using the entity mapping values.
In this case the model property is bound to the model_txt column.
If the association to a related entity involve a composite primary key,
a @FieldResult element should be used for each foreign key column.
The @FieldResult name is composed of the property name for the relationship,
followed by a dot ("."), followed by the name or the field or property of the primary key.

在这个例子中，我们仅描述结果集映射的实体成员。属性/列映射使用实体映射值完成。
在此情况下，模型的属性被绑定到 model_txt 列。如果相关的实体的关联涉及复合主键，
@FieldResult 元素应该被用于每个外键列。@FieldResult 名由关联的属性名组成，
后跟随一个点（"."），跟随主键的名字、字段或属性。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;
            /**
             * @NamedNativeQueries({
             *      @NamedNativeQuery(
             *          name            = "fetchJoinedAddress",
             *          resultSetMapping= "mappingJoinedAddress",
             *          query           = "SELECT u.id, u.name, u.status, a.id AS a_id, a.country AS a_country, a.zip AS a_zip, a.city AS a_city FROM users u INNER JOIN addresses a ON u.id = a.user_id WHERE u.username = ?"
             *      ),
             * })
             * @SqlResultSetMappings({
             *      @SqlResultSetMapping(
             *          name    = "mappingJoinedAddress",
             *          entities= {
             *              @EntityResult(
             *                  entityClass = "__CLASS__",
             *                  fields      = {
             *                      @FieldResult(name = "id"),
             *                      @FieldResult(name = "name"),
             *                      @FieldResult(name = "status"),
             *                      @FieldResult(name = "address.id", column = "a_id"),
             *                      @FieldResult(name = "address.zip", column = "a_zip"),
             *                      @FieldResult(name = "address.city", column = "a_city"),
             *                      @FieldResult(name = "address.country", column = "a_country"),
             *                  }
             *              )
             *          }
             *      )
             * })
             */
            class User
            {
                /** @Id @Column(type="integer") @GeneratedValue */
                public $id;

                /** @Column(type="string", length=50, nullable=true) */
                public $status;

                /** @Column(type="string", length=255, unique=true) */
                public $username;

                /** @Column(type="string", length=255) */
                public $name;

                /** @OneToOne(targetEntity="Address") */
                public $address;

                // ....
            }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Model\User">
                <named-native-queries>
                    <named-native-query name="fetchJoinedAddress" result-set-mapping="mappingJoinedAddress">
                        <query>SELECT u.id, u.name, u.status, a.id AS a_id, a.country AS a_country, a.zip AS a_zip, a.city AS a_city FROM users u INNER JOIN addresses a ON u.id = a.user_id WHERE u.username = ?</query>
                    </named-native-query>
                </named-native-queries>
                <sql-result-set-mappings>
                    <sql-result-set-mapping name="mappingJoinedAddress">
                        <entity-result entity-class="__CLASS__">
                            <field-result name="id"/>
                            <field-result name="name"/>
                            <field-result name="status"/>
                            <field-result name="address.id" column="a_id"/>
                            <field-result name="address.zip"  column="a_zip"/>
                            <field-result name="address.city"  column="a_city"/>
                            <field-result name="address.country" column="a_country"/>
                        </entity-result>
                    </sql-result-set-mapping>
                </sql-result-set-mappings>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Model\User:
          type: entity
          namedNativeQueries:
            fetchJoinedAddress:
              name: fetchJoinedAddress
              resultSetMapping: mappingJoinedAddress
              query: SELECT u.id, u.name, u.status, a.id AS a_id, a.country AS a_country, a.zip AS a_zip, a.city AS a_city FROM users u INNER JOIN addresses a ON u.id = a.user_id WHERE u.username = ?
          sqlResultSetMappings:
            mappingJoinedAddress:
              entityResult:
                0:
                  entityClass: __CLASS__
                  fieldResult:
                    0:
                      name: id
                    1:
                      name: name
                    2:
                      name: status
                    3:
                      name: address.id
                      column: a_id
                    4:
                      name: address.zip
                      column: a_zip
                    5:
                      name: address.city
                      column: a_city
                    6:
                      name: address.country
                      column: a_country
                    


If you retrieve a single entity and if you use the default mapping,
you can use the resultClass attribute instead of resultSetMapping:

如果你取回一个单一实体且如果你使用默认映射，你可以使用 resultClass 属性替换
resultSetMapping：

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;
            /**
             * @NamedNativeQueries({
             *      @NamedNativeQuery(
             *          name           = "find-by-id",
             *          resultClass    = "Address",
             *          query          = "SELECT * FROM addresses"
             *      ),
             * })
             */
           class Address
           {
                // ....
           }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Model\Address">
                <named-native-queries>
                    <named-native-query name="find-by-id" result-class="Address">
                        <query>SELECT * FROM addresses WHERE id = ?</query>
                    </named-native-query>
                </named-native-queries>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Model\Address:
          type: entity
          namedNativeQueries:
            findAll:
              name: findAll
              resultClass: Address
              query: SELECT * FROM addresses


In some of your native queries, you'll have to return scalar values,
for example when building report queries.
You can map them in the @SqlResultsetMapping through @ColumnResult.
You actually can even mix, entities and scalar returns in the same native query (this is probably not that common though).

在你的一些原生查询中，你将必须返回标量值，比如当构建报告查询时。你可以通过 @ColumnResult
在 @SqlResultsetMapping 中映射它们。事实上，你甚至可以混合实体和标量值在同一个原生查询中返回
（尽管这并不常见）。

.. configuration-block::

    .. code-block:: php

        <?php
        namespace MyProject\Model;
            /**
             * @NamedNativeQueries({
             *      @NamedNativeQuery(
             *          name            = "count",
             *          resultSetMapping= "mappingCount",
             *          query           = "SELECT COUNT(*) AS count FROM addresses"
             *      )
             * })
             * @SqlResultSetMappings({
             *      @SqlResultSetMapping(
             *          name    = "mappingCount",
             *          columns = {
             *              @ColumnResult(
             *                  name = "count"
             *              )
             *          }
             *      )
             * })
             */
           class Address
           {
                // ....
           }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="MyProject\Model\Address">
                <named-native-query name="count" result-set-mapping="mappingCount">
                    <query>SELECT COUNT(*) AS count FROM addresses</query>
                </named-native-query>
                <sql-result-set-mappings>
                    <sql-result-set-mapping name="mappingCount">
                        <column-result name="count"/>
                    </sql-result-set-mapping>
                </sql-result-set-mappings>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        MyProject\Model\Address:
          type: entity
          namedNativeQueries:
            count:
              name: count
              resultSetMapping: mappingCount
              query: SELECT COUNT(*) AS count FROM addresses
          sqlResultSetMappings:
            mappingCount:
              name: mappingCount
              columnResult:
                count:
                  name: count
