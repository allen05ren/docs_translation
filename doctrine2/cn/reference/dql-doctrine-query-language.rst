Doctrine Query Language // Doctrine 查询语言
=================================================

DQL stands for Doctrine Query Language and is an Object
Query Language derivate that is very similar to the Hibernate
Query Language (HQL) or the Java Persistence Query Language (JPQL).

DQL 代表 Doctrine Query Language 且是一个对象查询语言的衍生，它非常类似于
Hibernate 查询语言（HQL）或 Jave 持久化查询语言（JPQL）。

In essence, DQL provides powerful querying capabilities over your
object model. Imagine all your objects lying around in some storage
(like an object database). When writing DQL queries, think about
querying that storage to pick a certain subset of your objects.

本质上，DQL 提供强大的基于你的对象模型上的查询能力。想象所有躺在在某些存储（类似对象数据库）里的对象。
当写 DQL 查询时，思考关于查询那个存储以选择你对象的某些子集。

.. note::

    A common mistake for beginners is to mistake DQL for
    being just some form of SQL and therefore trying to use table names
    and column names or join arbitrary tables together in a query. You
    need to think about DQL as a query language for your object model,
    not for your relational schema.

    对于初学者一个常见的错误是误解 DQL 仅仅是 SQL 的某种形式且因此尝试使用表名和字段名或联结（join）
    任意的表一起在一个查询中。你需要思考关于 DQL 作为一种针对你的对象模型的查询语言，而不是针对你的关系
    数据库（schema）。

DQL is case in-sensitive, except for namespace, class and field
names, which are case sensitive.

DQL 是大小写不敏感的，除了命名空间、类和字段名之外，它们是大小写敏感的。

Types of DQL queries // DQL 查询的类型
------------------------------------------

DQL as a query language has SELECT, UPDATE and DELETE constructs
that map to their corresponding SQL statement types. INSERT
statements are not allowed in DQL, because entities and their
relations have to be introduced into the persistence context
through ``EntityManager#persist()`` to ensure consistency of your
object model.

DQL 作为以中查询语言具有 SELECT、UPDATE 和 DELETE 结构映射到它们相应的 SQL 语句类型。
INSERT 语句在 DQL 中是不被允许的，因为实体和它们的关联必须通过
``EntityManager#persist()`` 引入到持久化上下文中以确保你的对象模型的一致性。

DQL SELECT statements are a very powerful way of retrieving parts
of your domain model that are not accessible via associations.
Additionally they allow to retrieve entities and their associations
in one single SQL select statement which can make a huge difference
in performance in contrast to using several queries.

DQL SELECT 语句是非常强大的取回你的领域模型的部分，它们通过关联是不可访问的。另外，
它们允许在单一 SQL SELECT 语句中取回实体及它们的关联，这与使用多个查询对比能够造成巨大的
性能差异。

DQL UPDATE and DELETE statements offer a way to execute bulk
changes on the entities of your domain model. This is often
necessary when you cannot load all the affected entities of a bulk
update into memory.

DQL UPDATE 和 DELETE 语句提供一种在你的领域模型的实体上执行大量变更的方式。
这是经常需要的，当你不能载入大量 update 的所有的受影响实体到内存中时。

SELECT queries // SELECT 查询
-----------------------------------

DQL SELECT clause // DQL SELECT 子句
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The select clause of a DQL query specifies what appears in the
query result. The composition of all the expressions in the select
clause also influences the nature of the query result.

DQL 查询的 SELECT 子句指定了什么出现在查询结果中。在该 SELECT 子句中的所有的表达式组合也
影响该查询结果的性质。

Here is an example that selects all users with an age > 20:

以下例子选择年龄大于20的所有用户：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u WHERE u.age > 20');
    $users = $query->getResult();

Lets examine the query:

让我们剖析该查询：

-  ``u`` is a so called identification variable or alias that
   refers to the ``MyProject\Model\User`` class. By placing this alias
   in the SELECT clause we specify that we want all instances of the
   User class that are matched by this query to appear in the query
   result.
-  ``u`` 是一个被称为识别变量或别名，它引用至 ``MyProject\Model\User`` 类。
   通过在该 SELECT 子句中放置该别名，我们指定了我们想要通过该查询显示在查询结果中
   的被匹配的 User 类的实例。
-  The FROM keyword is always followed by a fully-qualified class
   name which in turn is followed by an identification variable or
   alias for that class name. This class designates a root of our
   query from which we can navigate further via joins (explained
   later) and path expressions.
-  FROM 关键字后面总是跟着一个完全限定的类名，之后跟着一个识别变量或别名。该类指定了一个
   我们的查询的根，我们可以通过联结（稍后解释）和路径表达式更进一步导航。
-  The expression ``u.age`` in the WHERE clause is a path
   expression. Path expressions in DQL are easily identified by the
   use of the '.' operator that is used for constructing paths. The
   path expression ``u.age`` refers to the ``age`` field on the User
   class.
-  在 WHERE 子句中的表达式 ``u.age`` 是一个路径表达式。路径表达式在 DQL 中是很容易
   通过使用的点（"."）操作符被识别，点操作符被用来构造路径。路径表达式 ``u.age``
   引用 User 类上的 ``age`` 字段。

The result of this query would be a list of User objects where all
users are older than 20.

这个查询的结果会是一个 User 对象的列表，该列表中的所有 Users 的年龄都大于20。

The SELECT clause allows to specify both class identification
variables that signal the hydration of a complete entity class or
just fields of the entity using the syntax ``u.name``. Combinations
of both are also allowed and it is possible to wrap both fields and
identification values into aggregation and DQL functions. Numerical
fields can be part of computations using mathematical operations.
See the sub-section on `Functions, Operators, Aggregates`_ for
more information.

SELECT 子句允许指定两类标识变量，一个完整实体类的水合（hydration）符号或仅是使用
``u.name`` 语法的实体的字段。组合这两者也是被允许的且包装字段与识别值到聚合和 DQL 函数中
是可能的。数字字段可以是使用数学运算的计算的一部分。更多参见 `Functions, Operators, Aggregates`_
小节。

Joins // 联结
~~~~~~~~~~~~~~~~~~

A SELECT query can contain joins. There are 2 types of JOINs:
"Regular" Joins and "Fetch" Joins.

SELECT 查询可以包含联结（join）。有两种类型的联结：
“Regular” Joins 和 “Fetch” Joins

**Regular Joins**: Used to limit the results and/or compute
aggregate values.

**Regular Joins**：用于限定结果和/或计算聚合值。

**Fetch Joins**: In addition to the uses of regular joins: Used to
fetch related entities and include them in the hydrated result of a
query.

**Fetch Joins**：除了 regular joins 的用途之外：用于获取关联的实体和在一个查询的水合的（hydrated）结果中包含它们。

There is no special DQL keyword that distinguishes a regular join
from a fetch join. A join (be it an inner or outer join) becomes a
"fetch join" as soon as fields of the joined entity appear in the
SELECT part of the DQL query outside of an aggregate function.
Otherwise its a "regular join".

没有专门的 DQL 关键字区分 regular join 和 fetch join。
只要已联结的实体的字段出现在聚合函数之外的DQL 查询的 SELECT 部分中，联结（可能是内联结或外联结）就变为 "fetch join"类。
否则，它就是一个 “regular join”。

Example:

例子：

Regular join of the address:

Regular join 的地址：

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
    $users = $query->getResult();

Fetch join of the address:

Fetch join 的地址：

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u, a FROM User u JOIN u.address a WHERE a.city = 'Berlin'");
    $users = $query->getResult();

When Doctrine hydrates a query with fetch-join it returns the class
in the FROM clause on the root level of the result array. In the
previous example an array of User instances is returned and the
address of each user is fetched and hydrated into the
``User#address`` variable. If you access the address Doctrine does
not need to lazy load the association with another query.

当 Doctrine 用 fetch-join 水合（hydrate）一个查询，它返回在结果数组根级上的 FROM 子句中的类。
在面前的例子中返回一个用户实例的数组，每个用户的地址被取回并水合进 ``User#address`` 变量中。
如果你访问该地址 Doctrine 不需要用另一个查询懒加载此关联。

.. note::

    Doctrine allows you to walk all the associations between
    all the objects in your domain model. Objects that were not already
    loaded from the database are replaced with lazy load proxy
    instances. Non-loaded Collections are also replaced by lazy-load
    instances that fetch all the contained objects upon first access.
    However relying on the lazy-load mechanism leads to many small
    queries executed against the database, which can significantly
    affect the performance of your application. **Fetch Joins** are the
    solution to hydrate most or all of the entities that you need in a
    single SELECT query.

    Doctrine 允许你涉足所有在你的领域模型中的对象之间的所有关联。尚未从数据库加载的对象将替换为懒加载代理实例。
    非已加载（Non-loaded）集合也有懒加载实例替换，在第一次访问时获取所有包含的对象。
    然而，依赖懒加载机制会导致针对数据库执行许多小查询，这会显着影响应用程序的性能。
    **Fetch Joins** 是在单个 SELECT 查询中对大多数或所有实体进行水合的解决方案。


Named and Positional Parameters // 命名与位置参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DQL supports both named and positional parameters, however in
contrast to many SQL dialects positional parameters are specified
with numbers, for example "?1", "?2" and so on. Named parameters
are specified with ":name1", ":name2" and so on.

DQL 支持命名和位置参数，然而与大多数 SQL 方言相比它位置参数被指定为数字,如“?1”、“?2”等等，命名参数被指定为
“:name1”、“:name2”等等。

When referencing the parameters in ``Query#setParameter($param, $value)``
both named and positional parameters are used **without** their prefixes.

当在 ``Query#setParameter($param, $value)`` 中引用参数时，命名和位置参数**不带**前缀使用。

DQL SELECT Examples // DQL SELECT 示例
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section contains a large set of DQL queries and some
explanations of what is happening. The actual result also depends
on the hydration mode.
本节包括大量的 DQL 查询及其解释。实际的结果还依赖于水合（hydration）模式。

Hydrate all User entities:

水合（hydrate）所有 User 实体：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u');
    $users = $query->getResult(); // array of User objects

Retrieve the IDs of all CmsUsers:

取回所有 CmsUser 的 IDs：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.id FROM CmsUser u');
    $ids = $query->getResult(); // array of CmsUser ids

Retrieve the IDs of all users that have written an article:

取回所有写过文章的 User 的IDs：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT DISTINCT u.id FROM CmsArticle a JOIN a.user u');
    $ids = $query->getResult(); // array of CmsUser ids

Retrieve all articles and sort them by the name of the articles
users instance:

取回所有文章并用文字的 User 实例的名字（name）排序：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT a FROM CmsArticle a JOIN a.user u ORDER BY u.name ASC');
    $articles = $query->getResult(); // array of CmsArticle objects

Retrieve the Username and Name of a CmsUser:

取回所有用户名（username）和 CmsUser 的名字（name）：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.username, u.name FROM CmsUser u');
    $users = $query->getResult(); // array of CmsUser username and name values
    echo $users[0]['username'];

Retrieve a ForumUser and his single associated entity:

取回一个 ForumUser及其单一关联的实体：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u, a FROM ForumUser u JOIN u.avatar a');
    $users = $query->getResult(); // array of ForumUser objects with the avatar association loaded
    echo get_class($users[0]->getAvatar());

Retrieve a CmsUser and fetch join all the phonenumbers he has:

取回一个 CmsUser 并 fetch join 他拥有的所有的 phonenumbers：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u, p FROM CmsUser u JOIN u.phonenumbers p');
    $users = $query->getResult(); // array of CmsUser objects with the phonenumbers association loaded
    $phonenumbers = $users[0]->getPhonenumbers();

Hydrate a result in Ascending:

水合（hydrate）一个结果升序：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM ForumUser u ORDER BY u.id ASC');
    $users = $query->getResult(); // array of ForumUser objects

Or in Descending Order:

或降序排序：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM ForumUser u ORDER BY u.id DESC');
    $users = $query->getResult(); // array of ForumUser objects

Using Aggregate Functions:

使用聚合函数：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT COUNT(u.id) FROM Entities\User u');
    $count = $query->getSingleScalarResult();

    $query = $em->createQuery('SELECT u, count(g.id) FROM Entities\User u JOIN u.groups g GROUP BY u.id');
    $result = $query->getResult();

With WHERE Clause and Positional Parameter:

使用 WHERE 子句和位置参数：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE u.id = ?1');
    $query->setParameter(1, 321);
    $users = $query->getResult(); // array of ForumUser objects

With WHERE Clause and Named Parameter:

使用 WHERE 子句和命名参数：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE u.username = :name');
    $query->setParameter('name', 'Bob');
    $users = $query->getResult(); // array of ForumUser objects

With Nested Conditions in WHERE Clause:

在 WHERE 子句中使用嵌套条件：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM ForumUser u WHERE (u.username = :name OR u.username = :name2) AND u.id = :id');
    $query->setParameters(array(
        'name' => 'Bob',
        'name2' => 'Alice',
        'id' => 321,
    ));
    $users = $query->getResult(); // array of ForumUser objects

With COUNT DISTINCT:

使用 COUNT DISTINCT：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT COUNT(DISTINCT u.name) FROM CmsUser');
    $users = $query->getResult(); // array of ForumUser objects

With Arithmetic Expression in WHERE clause:

在 WHERE 子句中使用算术表达式：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE ((u.id + 5000) * u.id + 3) < 10000000');
    $users = $query->getResult(); // array of ForumUser objects

Retrieve user entities with Arithmetic Expression in ORDER clause, using the ``HIDDEN`` keyword:

在 ORDER 子句中使用算术表达式取回 User 实体，并使用了 ``HIDDEN`` 关键字：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u, u.posts_count + u.likes_count AS HIDDEN score FROM CmsUser u ORDER BY score');
    $users = $query->getResult(); // array of User objects

Using a LEFT JOIN to hydrate all user-ids and optionally associated
article-ids:

使用左联结（LEFT JOIN）水合（hydrate）所有 User IDs 和可选关联的文章 IDs：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.id, a.id as article_id FROM CmsUser u LEFT JOIN u.articles a');
    $results = $query->getResult(); // array of user ids and every article_id for each user

Restricting a JOIN clause by additional conditions:

通过额外条件约束 JOIN 子句：

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u FROM CmsUser u LEFT JOIN u.articles a WITH a.topic LIKE :foo");
    $query->setParameter('foo', '%foo%');
    $users = $query->getResult();

Using several Fetch JOINs:

使用几个 Fetch JOINs：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u, a, p, c FROM CmsUser u JOIN u.articles a JOIN u.phonenumbers p JOIN a.comments c');
    $users = $query->getResult();

BETWEEN in WHERE clause:

在 WHERE 子句中的 BETWEEN：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.name FROM CmsUser u WHERE u.id BETWEEN ?1 AND ?2');
    $query->setParameter(1, 123);
    $query->setParameter(2, 321);
    $usernames = $query->getResult();

DQL Functions in WHERE clause:

在 WHERE 子句中的 DQL 函数： 

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u.name FROM CmsUser u WHERE TRIM(u.name) = 'someone'");
    $usernames = $query->getResult();

IN() Expression:

IN() 表达式：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.name FROM CmsUser u WHERE u.id IN(46)');
    $usernames = $query->getResult();

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.id IN (1, 2)');
    $users = $query->getResult();

    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.id NOT IN (1)');
    $users = $query->getResult();

CONCAT() DQL Function:

CONCAT() DQL 函数：

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u.id FROM CmsUser u WHERE CONCAT(u.name, 's') = ?1");
    $query->setParameter(1, 'Jess');
    $ids = $query->getResult();

    $query = $em->createQuery('SELECT CONCAT(u.id, u.name) FROM CmsUser u WHERE u.id = ?1');
    $query->setParameter(1, 321);
    $idUsernames = $query->getResult();

EXISTS in WHERE clause with correlated Subquery

在带相关的子查询的 WHERE 子句中的 EXISTS：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.id FROM CmsUser u WHERE EXISTS (SELECT p.phonenumber FROM CmsPhonenumber p WHERE p.user = u.id)');
    $ids = $query->getResult();

Get all users who are members of $group.

取得 $group 中的所有成员用户：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u.id FROM CmsUser u WHERE :groupId MEMBER OF u.groups');
    $query->setParameter('groupId', $group);
    $ids = $query->getResult();

Get all users that have more than 1 phonenumber

取得超过 1 个 phonenumber 的所有用户：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE SIZE(u.phonenumbers) > 1');
    $users = $query->getResult();

Get all users that have no phonenumber

取得没有 phonenumber 的所有用户：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u WHERE u.phonenumbers IS EMPTY');
    $users = $query->getResult();

Get all instances of a specific type, for use with inheritance
hierarchies:

取得所有指定类型的实例，对于使用的继承层次结构：

.. versionadded:: 2.1

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u INSTANCE OF Doctrine\Tests\Models\Company\CompanyEmployee');
    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u INSTANCE OF ?1');
    $query = $em->createQuery('SELECT u FROM Doctrine\Tests\Models\Company\CompanyPerson u WHERE u NOT INSTANCE OF ?1');

Get all users visible on a given website that have chosen certain gender:

取得已挑选某性别在给定站点上可见的所有用户：

.. versionadded:: 2.2

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM User u WHERE u.gender IN (SELECT IDENTITY(agl.gender) FROM Site s JOIN s.activeGenderList agl WHERE s.id = ?1)');

.. versionadded:: 2.4

Starting with 2.4, the IDENTITY() DQL function also works for composite primary keys:

对于复合主键，2.4 版本引入的 IDENTITY() DQL 函数也工作：

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT IDENTITY(c.location, 'latitude') AS latitude, IDENTITY(c.location, 'longitude') AS longitude FROM Checkpoint c WHERE c.user = ?1");

Joins between entities without associations were not possible until version
2.4, where you can generate an arbitrary join with the following syntax:

没有关联的实体之间的联结（join）是不可能的，直到 2.4 版本，你可以使用以下语法产生任意的联结（join）：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM User u JOIN Blacklist b WITH u.email = b.email');

Partial Object Syntax // 部分对象的语法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default when you run a DQL query in Doctrine and select only a
subset of the fields for a given entity, you do not receive objects
back. Instead, you receive only arrays as a flat rectangular result
set, similar to how you would if you were just using SQL directly
and joining some data.

默认地，当你在 Doctrine 中执行一条 DQL 查询并只选择给定实体的字段的子集，你不会接收到对象返回。
相反，你仅接收到一个扁平的矩形结果集数组,类似于你直接地使用 SQL 并联结一些数据。

If you want to select partial objects you can use the ``partial``
DQL keyword:

如果你希望选择部分对象你可以使用 ``partial`` DQL 关键字：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT partial u.{id, username} FROM CmsUser u');
    $users = $query->getResult(); // array of partially loaded CmsUser objects

You use the partial syntax when joining as well:

当联结时使用部分对象的语法也一样：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT partial u.{id, username}, partial a.{id, name} FROM CmsUser u JOIN u.articles a');
    $users = $query->getResult(); // array of partially loaded CmsUser objects

"NEW" Operator Syntax // “NEW” 操作符的语法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. versionadded:: 2.4

Using the ``NEW`` operator you can construct Data Transfer Objects (DTOs) directly from DQL queries.

使用 ``NEW`` 操作符你可以直接地从 DQL 查询构造数据转换对象。

- When using ``SELECT NEW`` you don't need to specify a mapped entity.
- 当使用 ``SELECT NEW`` 你不能指定映射的实体。
- You can specify any PHP class, it only requires that the constructor of this class matches the ``NEW`` statement.
- 你可以指定任何 PHP 类，仅需要此类的构造器匹配 ``NEW`` 语句。
- This approach involves determining exactly which columns you really need,
  and instantiating a data-transfer object that contains a constructor with those arguments.
- 此方法涉及确定确切地你真实需要那些列，并实例化一个包含带有这些参数的构造器的数据转换对象。

If you want to select data-transfer objects you should create a class:

如果你希望选择数据库转换对象你应该创建一个类：

.. code-block:: php

    <?php
    class CustomerDTO
    {
        public function __construct($name, $email, $city, $value = null)
        {
            // Bind values to the object properties.
        }
    }

And then use the ``NEW`` DQL keyword :

然后，使用 ``NEW`` DQL 关键字：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT NEW CustomerDTO(c.name, e.email, a.city) FROM Customer c JOIN c.email e JOIN c.address a');
    $users = $query->getResult(); // array of CustomerDTO

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT NEW CustomerDTO(c.name, e.email, a.city, SUM(o.value)) FROM Customer c JOIN c.email e JOIN c.address a JOIN c.orders o GROUP BY c');
    $users = $query->getResult(); // array of CustomerDTO

Note that you can only pass scalar expressions to the constructor.

注意你仅可以传递标量表达式至构造器。

Using INDEX BY // 使用 INDEX BY
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The INDEX BY construct is nothing that directly translates into SQL
but that affects object and array hydration. After each FROM and
JOIN clause you specify by which field this class should be indexed
in the result. By default a result is incremented by numerical keys
starting with 0. However with INDEX BY you can specify any other
column to be the key of your result, it really only makes sense
with primary or unique fields though:

INDEX BY 构造没有直接地转换为 SQL，但是它影响对象及数组的水化（hydration）。
每个 FROM 和 JOIN 子句之后你指定通过那个字段索引此类在结果中。默认地，
结果由起始为0的数字键递增。但是，使用 INDEX BY 你可以指定任何其他列作为你的结果的键，
尽管它实际上仅使主键和唯一字段有一样：

.. code-block:: sql

    SELECT u.id, u.status, upper(u.name) nameUpper FROM User u INDEX BY u.id
    JOIN u.phonenumbers p INDEX BY p.phonenumber

Returns an array of the following kind, indexed by both user-id
then phonenumber-id:

返回如下类型的数组，通过 user-id 索引然后 phonenumber-id：

.. code-block:: php

    array
      0 =>
        array
          1 =>
            object(stdClass)[299]
              public '__CLASS__' => string 'Doctrine\Tests\Models\CMS\CmsUser' (length=33)
              public 'id' => int 1
              ..
          'nameUpper' => string 'ROMANB' (length=6)
      1 =>
        array
          2 =>
            object(stdClass)[298]
              public '__CLASS__' => string 'Doctrine\Tests\Models\CMS\CmsUser' (length=33)
              public 'id' => int 2
              ...
          'nameUpper' => string 'JWAGE' (length=5)

UPDATE queries // UPDATE 查询
-----------------------------------

DQL not only allows to select your Entities using field names, you
can also execute bulk updates on a set of entities using an
DQL-UPDATE query. The Syntax of an UPDATE query works as expected,
as the following example shows:

DQL 不仅允许你使用字段名选择实体，你也可以在一组实体上使用一个 DQL-UPDATE 查询执行大量 update。
UPDATE 查询的语法如同预期的那样工作，就像如下例子展示：

.. code-block:: sql

    UPDATE MyProject\Model\User u SET u.password = 'new' WHERE u.id IN (1, 2, 3)

References to related entities are only possible in the WHERE
clause and using sub-selects.

关联的实体的引用仅可能在 WHERE 子句和使用子 SELECT（sub-selects）中。

.. warning::

    DQL UPDATE statements are ported directly into a
    Database UPDATE statement and therefore bypass any locking scheme, events
    and do not increment the version column. Entities that are already
    loaded into the persistence context will *NOT* be synced with the
    updated database state. It is recommended to call
    ``EntityManager#clear()`` and retrieve new instances of any
    affected entity.

    DQL UPDATE 语句直接地被移植到一个数据库 UPDATE 语句中，因此绕过了然后数据库（scheme）锁、
    事件且不能自增版本列。已经被加载进实例化上下文中的实体将**不**与已更新的数据库中的状态同步。
    推荐调用 ``EntityManager#clear()`` 并取回任何受影响实体的新实例。

DELETE queries // DELETE 查询
-----------------------------------

DELETE queries can also be specified using DQL and their syntax is
as simple as the UPDATE syntax:

DELETE 查询也可以使用 DQL 指定，并且它们的语法和 UPDATE 语法一样简单：

.. code-block:: sql

    DELETE MyProject\Model\User u WHERE u.id = 4

The same restrictions apply for the reference of related entities.

对相关的实体的引用的限制同样适用。

.. warning::

    DQL DELETE statements are ported directly into a
    Database DELETE statement and therefore bypass any events and checks for the
    version column if they are not explicitly added to the WHERE clause
    of the query. Additionally Deletes of specifies entities are *NOT*
    cascaded to related entities even if specified in the metadata.

    DQL DELETE 语句直接地被移植到一个数据库 DELETE 语句，因此绕过了任何事件并且检查版本列
    如果它们没有明确地被添加到查询的 WHERE 子句。额外地，指定实体的删除*不*级联到关联的实体，即使在
    元数据（metadata）中指定。

Functions, Operators, Aggregates
---------------------------------

DQL Functions // DQL 函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following functions are supported in SELECT, WHERE and HAVING
clauses:

在 SELECT 、WHERE 和 HAVING 子句中支持以下函数：

-  IDENTITY(single\_association\_path\_expression [, fieldMapping]) - Retrieve the foreign key column of association of the owning side
-  IDENTITY(single\_association\_path\_expression [, fieldMapping]) - 取回关联的 owning 侧的外键
-  ABS(arithmetic\_expression)
-  ABS(arithmetic\_expression) - 算术表达式绝对值
-  CONCAT(str1, str2)
-  CONCAT(str1, str2) - 字符串连接函数
-  CURRENT\_DATE() - Return the current date
-  CURRENT\_DATE() - 返回当前日期
-  CURRENT\_TIME() - Returns the current time
-  CURRENT\_TIME() - 返回当前时间
-  CURRENT\_TIMESTAMP() - Returns a timestamp of the current date
   and time.
-  CURRENT\_TIMESTAMP() - 返回当前日期和时间的时间戳
-  LENGTH(str) - Returns the length of the given string
-  LENGTH(str) - 返回给定字符串的长度
-  LOCATE(needle, haystack [, offset]) - Locate the first
   occurrence of the substring in the string.
-  LOCATE(needle, haystack [, offset]) - 在字符串中定位第一次出现的子字符串。
-  LOWER(str) - returns the string lowercased.
-  LOWER(str) - 返回小写形式的字符串。
-  MOD(a, b) - Return a MOD b.
-  MOD(a, b) - 返回 a 除于 b 的余数（取模）。
-  SIZE(collection) - Return the number of elements in the
   specified collection
-  SIZE(collection) - 返回在指定的集合中的元素数量
-  SQRT(q) - Return the square-root of q.
-  SQRT(q) - 返回 q 的平方根
-  SUBSTRING(str, start [, length]) - Return substring of given
   string.
-  SUBSTRING(str, start [, length]) - 返回给定字符串的子字符串。
-  TRIM([LEADING \| TRAILING \| BOTH] ['trchar' FROM] str) - Trim
   the string by the given trim char, defaults to whitespaces.
-  TRIM([LEADING \| TRAILING \| BOTH] ['trchar' FROM] str) - 通过给定的修剪字符修剪字符串，默认为空白符。
-  UPPER(str) - Return the upper-case of the given string.
-  UPPER(str) - 返回给定字符串的大写形式。
-  DATE_ADD(date, days, unit) - Add the number of days to a given date. (Supported units are DAY, MONTH)
-  DATE_ADD(date, days, unit) - 添加天数至给定日期。（支持单位为天、月份）
-  DATE_SUB(date, days, unit) - Substract the number of days from a given date. (Supported units are DAY, MONTH)
-  DATE_SUB(date, days, unit) - 从给定日期减去天数。（支持单位为天、月份）
-  DATE_DIFF(date1, date2) - Calculate the difference in days between date1-date2.
-  DATE_DIFF(date1, date2) - 计算两个日期相差的天数。

Arithmetic operators // 算术操作符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can do math in DQL using numeric values, for example:

你可以在 DQL 中使用数字值进行数学运算，例如：

.. code-block:: sql

    SELECT person.salary * 1.5 FROM CompanyPerson person WHERE person.salary < 100000

Aggregate Functions // 聚合函数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following aggregate functions are allowed in SELECT and GROUP
BY clauses: AVG, COUNT, MIN, MAX, SUM

在 SELECT 和 GROUP BY 子句中允许以下聚合函数：AVG、COUNT、MIN、MAX、SUM。

Other Expressions // 其他表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

DQL offers a wide-range of additional expressions that are known
from SQL, here is a list of all the supported constructs:

DQL 提供了从 SQL 中知晓的广泛范围的额外表达式，这里有一个支持的概念列表：

-  ``ALL/ANY/SOME`` - Used in a WHERE clause followed by a
   sub-select this works like the equivalent constructs in SQL.
-  ``ALL/ANY/SOME`` - 用在 WHERE 子句之后跟随一个子 SELECT（sub-select），它等价于在 SQL 中的概念。
-  ``BETWEEN a AND b`` and ``NOT BETWEEN a AND b`` can be used to
   match ranges of arithmetic values.
-  ``BETWEEN a AND b`` 和 ``NOT BETWEEN a AND b`` 可被用于匹配算术运算值的范围。
-  ``IN (x1, x2, ...)`` and ``NOT IN (x1, x2, ..)`` can be used to
   match a set of given values.
-  ``IN (x1, x2, ...)`` 和 ``NOT IN (x1, x2, ..)`` 可被用于匹配一组给定的值。
-  ``LIKE ..`` and ``NOT LIKE ..`` match parts of a string or text
   using % as a wildcard.
-  ``LIKE ..`` 和 ``NOT LIKE ..`` 使用 % 作为通配符匹配字符串或文本的部分。
-  ``IS NULL`` and ``IS NOT NULL`` to check for null values
-  ``IS NULL`` 和 ``IS NOT NULL`` 检查 null 值
-  ``EXISTS`` and ``NOT EXISTS`` in combination with a sub-select
-  ``EXISTS`` 和 ``NOT EXISTS`` 结合子 SELECT（sub-select）

Adding your own functions to the DQL language // 添加自己的函数到 DQL 语言
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default DQL comes with functions that are part of a large basis
of underlying databases. However you will most likely choose a
database platform at the beginning of your project and most likely
never change it. For this cases you can easily extend the DQL
parser with own specialized platform functions.

默认地，DQL 伴随了数据库底层最基础的大部分函数。但是，你很可能在你的项目的开始阶段就选择了数据库平台，
并且很可能永远不会变更。在此情况下，你可以容易地用自己指定平台的函数扩展 DQL 的解析器。

You can register custom DQL functions in your ORM Configuration:

你可以在你的 ORM 配置中注册自定义的 DQL 函数：

.. code-block:: php

    <?php
    $config = new \Doctrine\ORM\Configuration();
    $config->addCustomStringFunction($name, $class);
    $config->addCustomNumericFunction($name, $class);
    $config->addCustomDatetimeFunction($name, $class);

    $em = EntityManager::create($dbParams, $config);

The functions have to return either a string, numeric or datetime
value depending on the registered function type. As an example we
will add a MySQL specific FLOOR() functionality. All the given
classes have to implement the base class :

函数必须返回字符串、数字或日期值，这依赖于所注册的函数类型。作为一个例子，我们将添加一个 MySQL 特定的
FLOOR() 功能。所有给定的类必须实现此基础类：

.. code-block:: php

    <?php
    namespace MyProject\Query\AST;

    use \Doctrine\ORM\Query\AST\Functions\FunctionNode;
    use \Doctrine\ORM\Query\Lexer;

    class MysqlFloor extends FunctionNode
    {
        public $simpleArithmeticExpression;

        public function getSql(\Doctrine\ORM\Query\SqlWalker $sqlWalker)
        {
            return 'FLOOR(' . $sqlWalker->walkSimpleArithmeticExpression(
                $this->simpleArithmeticExpression
            ) . ')';
        }

        public function parse(\Doctrine\ORM\Query\Parser $parser)
        {
            $parser->match(Lexer::T_IDENTIFIER);
            $parser->match(Lexer::T_OPEN_PARENTHESIS);

            $this->simpleArithmeticExpression = $parser->SimpleArithmeticExpression();

            $parser->match(Lexer::T_CLOSE_PARENTHESIS);
        }
    }

We will register the function by calling and can then use it:

我们将通过调用注册该函数并之后可以使用它：

.. code-block:: php

    <?php
    $config = $em->getConfiguration();
    $config->registerNumericFunction('FLOOR', 'MyProject\Query\MysqlFloor');

    $dql = "SELECT FLOOR(person.salary * 1.75) FROM CompanyPerson person";

Querying Inherited Classes // 查询继承的类
----------------------------------------------

This section demonstrates how you can query inherited classes and
what type of results to expect.

本节演示你可以如何查询继承的类和预期的结果的类型

Single Table // 单一表
~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Single Table Inheritance <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
is an inheritance mapping strategy where all classes of a hierarchy
are mapped to a single database table. In order to distinguish
which row represents which type in the hierarchy a so-called
discriminator column is used.

`单一表继承 <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
是一个继承映射策略，其中层次结果的所有类都被映射到一张单一的数据库表。为了区分哪一行代表在层次结构中的哪一类型，将
使用一个被称作鉴别器的列。

First we need to setup an example set of entities to use. In this
scenario it is a generic Person and Employee example:

首先，我们需要配置一组例子实体以便使用。在此情景中，它是一个常规的 Person 和 Employee 例子：

.. code-block:: php

    <?php
    namespace Entities;

    /**
     * @Entity
     * @InheritanceType("SINGLE_TABLE")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        /**
         * @Id @Column(type="integer")
         * @GeneratedValue
         */
        protected $id;

        /**
         * @Column(type="string", length=50)
         */
        protected $name;

        // ...
    }

    /**
     * @Entity
     */
    class Employee extends Person
    {
        /**
         * @Column(type="string", length=50)
         */
        private $department;

        // ...
    }

First notice that the generated SQL to create the tables for these
entities looks like the following:

首先请注意生成的 SQL 为这些实体创建一张表类似如下：

.. code-block:: sql

    CREATE TABLE Person (
        id INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
        name VARCHAR(50) NOT NULL,
        discr VARCHAR(255) NOT NULL,
        department VARCHAR(50) NOT NULL
    )

Now when persist a new ``Employee`` instance it will set the
discriminator value for us automatically:

现在当持久一个新的 ``Employee`` 实例，它将自动地为我们设置鉴别器值：

.. code-block:: php

    <?php
    $employee = new \Entities\Employee();
    $employee->setName('test');
    $employee->setDepartment('testing');
    $em->persist($employee);
    $em->flush();

Now lets run a simple query to retrieve the ``Employee`` we just
created:

现在让我们执行一个简单的查询来取回我们刚刚创建的 ``Employee``：

.. code-block:: sql

    SELECT e FROM Entities\Employee e WHERE e.name = 'test'

If we check the generated SQL you will notice it has some special
conditions added to ensure that we will only get back ``Employee``
entities:

如果我们检查生成的 SQL 你将注意到它有一些特定的条件已添加以确保我们将仅取回 ``Employee`` 实体：

.. code-block:: sql

    SELECT p0_.id AS id0, p0_.name AS name1, p0_.department AS department2,
           p0_.discr AS discr3 FROM Person p0_
    WHERE (p0_.name = ?) AND p0_.discr IN ('employee')

Class Table Inheritance // 类表继承
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

`Class Table Inheritance <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
is an inheritance mapping strategy where each class in a hierarchy
is mapped to several tables: its own table and the tables of all
parent classes. The table of a child class is linked to the table
of a parent class through a foreign key constraint. Doctrine 2
implements this strategy through the use of a discriminator column
in the topmost table of the hierarchy because this is the easiest
way to achieve polymorphic queries with Class Table Inheritance.

`类表继承 <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
是一个继承映射策略，其中在层次结构中的每一个类被映射到几个表：自身表和所有父类的表。
子类的表通过外键约束被链接到父类的表。Doctrine 2通过在层次结构中最顶层表中使用一个鉴别器列
实现了此策略，因为这是最简单的使用类表继承达到多态查询的方式。

The example for class table inheritance is the same as single
table, you just need to change the inheritance type from
``SINGLE_TABLE`` to ``JOINED``:

类表继承的示例于单一表一样，你仅需要修改继承类型 ``SINGLE_TABLE`` 为 ``JOINED``：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @InheritanceType("JOINED")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        // ...
    }

Now take a look at the SQL which is generated to create the table,
you'll notice some differences:

现在看看生成的 SQL 以创建表，你将注意到一些不同：

.. code-block:: sql

    CREATE TABLE Person (
        id INT AUTO_INCREMENT NOT NULL,
        name VARCHAR(50) NOT NULL,
        discr VARCHAR(255) NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Employee (
        id INT NOT NULL,
        department VARCHAR(50) NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Employee ADD FOREIGN KEY (id) REFERENCES Person(id) ON DELETE CASCADE


-  The data is split between two tables
-  数据在两张表之间被分离
-  A foreign key exists between the two tables
-  一个外键在两张表之间存在

Now if were to insert the same ``Employee`` as we did in the
``SINGLE_TABLE`` example and run the same example query it will
generate different SQL joining the ``Person`` information
automatically for you:

现在如果像我们在 ``SINGLE_TABLE`` 示例中那样插入一个 ``Employee`` 并执行同样的示例的查询，它将自动地为你
生成不同的 SQL 联结  ``Person`` 的信息。

.. code-block:: sql

    SELECT p0_.id AS id0, p0_.name AS name1, e1_.department AS department2,
           p0_.discr AS discr3
    FROM Employee e1_ INNER JOIN Person p0_ ON e1_.id = p0_.id
    WHERE p0_.name = ?


The Query class // 查询类
------------------------------

An instance of the ``Doctrine\ORM\Query`` class represents a DQL
query. You create a Query instance be calling
``EntityManager#createQuery($dql)``, passing the DQL query string.
Alternatively you can create an empty ``Query`` instance and invoke
``Query#setDQL($dql)`` afterwards. Here are some examples:

一个 ``Doctrine\ORM\Query`` 类的实例代表一条 DQL 查询。你可以调用
``EntityManager#createQuery($dql)``，并传递 DQL 查询字符串来创建一个查询实例。
此外，你可以创建一个空的 ``Query`` 实例并之后调用 ``Query#setDQL($dql)``。
这里有一些例子：

.. code-block:: php

    <?php
    // $em instanceof EntityManager

    // example1: passing a DQL string
    $q = $em->createQuery('select u from MyProject\Model\User u');

    // example2: using setDQL
    $q = $em->createQuery();
    $q->setDQL('select u from MyProject\Model\User u');

Query Result Formats // 查询结果格式化
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The format in which the result of a DQL SELECT query is returned
can be influenced by a so-called ``hydration mode``. A hydration
mode specifies a particular way in which a SQL result set is
transformed. Each hydration mode has its own dedicated method on
the Query class. Here they are:

一个 DQL SELECT 查询中返回结果的格式可以通过一个被称为 ``hydration mode``
所影响。水合模式（hydration mode）指定了一个 SQL 结果集被转换的一种特定方式。
每一种水合模式在查询类上都有它自己的专门的方法。它们是：

-  ``Query#getResult()``: Retrieves a collection of objects. The
   result is either a plain collection of objects (pure) or an array
   where the objects are nested in the result rows (mixed).
-  ``Query#getResult()``：取回一个对象的集合。结果是一个普通的对象的集合（纯的）或是一个数组，
   其中对象被嵌套在结果行中（混合的）。
-  ``Query#getSingleResult()``: Retrieves a single object. If the
   result contains more than one object, an ``NonUniqueResultException``
   is thrown. If the result contains no objects, an ``NoResultException``
   is thrown. The pure/mixed distinction does not apply.
-  ``Query#getSingleResult()``：取回一个单一对象。如果结果包含超过一个对象，一个
   ``NonUniqueResultException`` 异常被抛出。如果结果不包含任何对象，一个
   ``NoResultException`` 异常被抛出。纯的或混合的区分并不适用。
-  ``Query#getOneOrNullResult()``: Retrieve a single object. If no
   object is found null will be returned.
-  ``Query#getOneOrNullResult()``：取回一个单一对象。如果没有对象被找到将返回 null。
-  ``Query#getArrayResult()``: Retrieves an array graph (a nested
   array) that is largely interchangeable with the object graph
   generated by ``Query#getResult()`` for read-only purposes.
-  ``Query#getArrayResult()``：取回一个数组图（一个嵌套的数组），它很大程度上可以与
   由 ``Query#getResult()`` 出于只读意图生成的对象图互换。

    .. note::

        An array graph can differ from the corresponding object
        graph in certain scenarios due to the difference of the identity
        semantics between arrays and objects.

        一个数组图可以不同于相应的对象图，在某些情景下由于数组和对象之间标示语义的差异。

-  ``Query#getScalarResult()``: Retrieves a flat/rectangular result
   set of scalar values that can contain duplicate data. The
   pure/mixed distinction does not apply.
-  ``Query#getScalarResult()``：取回一个扁平的/矩形的标量（scalar）值，它可以包含重复数据。
   纯的/混合的区别并不适用。
-  ``Query#getSingleScalarResult()``: Retrieves a single scalar
   value from the result returned by the dbms. If the result contains
   more than a single scalar value, an exception is thrown. The
   pure/mixed distinction does not apply.
-  ``Query#getSingleScalarResult()``：取回一个从 DBMS 返回的结果中的单一标量（scalar）值。
   如果此结果包含超过一个单一标量（scalar）值，一个异常被抛出。纯的/混合的区别并不适用。

Instead of using these methods, you can alternatively use the
general-purpose method
``Query#execute(array $params = array(), $hydrationMode = Query::HYDRATE_OBJECT)``.
Using this method you can directly supply the hydration mode as the
second parameter via one of the Query constants. In fact, the
methods mentioned earlier are just convenient shortcuts for the
execute method. For example, the method ``Query#getResult()``
internally invokes execute, passing in ``Query::HYDRATE_OBJECT`` as
the hydration mode.

作为替代使用这些方法，你可以另外使用一般用途的方法 ``Query#execute(array $params = array(), $hydrationMode = Query::HYDRATE_OBJECT)``。
使用此方法通过一个查询常量你可以直接地提供水合模式作为第二个参数。事实上，前面提及的方法都仅仅是此方法便利的快捷方式。
例如，``Query#getResult()`` 方法内部调用此方法，传递 ``Query::HYDRATE_OBJECT`` 作为水合模式。

The use of the methods mentioned earlier is generally preferred as
it leads to more concise code.

使用前面提及的方法通常是被推荐的，因为它产生更简略的代码。

Pure and Mixed Results // 纯的和混合的结果
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The nature of a result returned by a DQL SELECT query retrieved
through ``Query#getResult()`` or ``Query#getArrayResult()`` can be
of 2 forms: **pure** and **mixed**. In the previous simple
examples, you already saw a "pure" query result, with only objects.
By default, the result type is **pure** but
**as soon as scalar values, such as aggregate values or other scalar values that do not belong to an entity, appear in the SELECT part of the DQL query, the result becomes mixed**.
A mixed result has a different structure than a pure result in
order to accommodate for the scalar values.

通过 ``Query#getResult()`` 或 ``Query#getArrayResult()`` 取回的由一条 DQL SELECT 查询返回的结果的性质（nature）可以是2种形态：
**纯的（pure）** and **混合的（mixed）**。在前面的简单例子中，你已经见过一个“纯的（pure）”查询结果，仅带有对象。
默认地，结果类型是**纯的（pure）**，但是**只要标量值，如不属于一个实体的聚合值或其他标量值，出现在 DQL 查询的 SELECT 部分，
就变成混合的（mixed）**。

A pure result usually looks like this:

一个纯的结果通常看上去像这样：

.. code-block:: php

    $dql = "SELECT u FROM User u";

    array
        [0] => Object
        [1] => Object
        [2] => Object
        ...

A mixed result on the other hand has the following general
structure:

另一方面，一个混合的结果具有以下常规结构：

.. code-block:: php

    $dql = "SELECT u, 'some scalar string', count(g.id) AS num FROM User u JOIN u.groups g GROUP BY u.id";

    array
        [0]
            [0] => Object
            [1] => "some scalar string"
            ['num'] => 42
            // ... more scalar values, either indexed numerically or with a name
        [1]
            [0] => Object
            [1] => "some scalar string"
            ['num'] => 42
            // ... more scalar values, either indexed numerically or with a name

To better understand mixed results, consider the following DQL
query:

更好的理解混合的结果，思考以下 DQL 查询：

.. code-block:: sql

    SELECT u, UPPER(u.name) nameUpper FROM MyProject\Model\User u

This query makes use of the ``UPPER`` DQL function that returns a
scalar value and because there is now a scalar value in the SELECT
clause, we get a mixed result.

此查询使用 ``UPPER`` DQL 函数返回一个标量（scalar）值且因为现在有一个标量值在 SELECT 子句中，
我们得到了一个混合的结果。

Conventions for mixed results are as follows:

对于混合结果，有如下惯例：

-  The object fetched in the FROM clause is always positioned with the key '0'.
-  在 FROM 子句中取回的对象始终被放置在键“0”的位置。
-  Every scalar without a name is numbered in the order given in the query, starting with 1.
-  每个没有名字的标量以查询中给定的顺序被编号，从1开始。
-  Every aliased scalar is given with its alias-name as the key. The case of the name is kept.
-  每个别名的标量使用它的别名作为键。名称的大小写被保留。
-  If several objects are fetched from the FROM clause they alternate every row.
-  如果从 FROM 子句取回了一些对象，它们轮流每一行


Here is how the result could look like:

结果可能看上去像：

.. code-block:: php

    array
        array
            [0] => User (Object)
            ['nameUpper'] => "ROMAN"
        array
            [0] => User (Object)
            ['nameUpper'] => "JONATHAN"
        ...

And here is how you would access it in PHP code:

在 PHP 代码中你大概这样访问它们：

.. code-block:: php

    <?php
    foreach ($results as $row) {
        echo "Name: " . $row[0]->getName();
        echo "Name UPPER: " . $row['nameUpper'];
    }

Fetching Multiple FROM Entities // 取回多样 FROM 的实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you fetch multiple entities that are listed in the FROM clause then the hydration
will return the rows iterating the different top-level entities.

如果你取回被列举在 FROM 子句中的多样的实体，那么水合（hydration）将返回迭代不同顶层实体的行。

.. code-block:: php

    $dql = "SELECT u, g FROM User u, Group g";

    array
        [0] => Object (User)
        [1] => Object (Group)
        [2] => Object (User)
        [3] => Object (Group)


Hydration Modes // 水合（hydration）模式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each of the Hydration Modes makes assumptions about how the result
is returned to user land. You should know about all the details to
make best use of the different result formats:

每一种水合模式假设有关结果如何被返回给用户侧。你应当知晓有关充分使用不同的结果格式的所有的细节：

The constants for the different hydration modes are:

不同水合模式有以下常量：

-  Query::HYDRATE\_OBJECT
-  Query::HYDRATE\_ARRAY
-  Query::HYDRATE\_SCALAR
-  Query::HYDRATE\_SINGLE\_SCALAR

Object Hydration // 对象水合（hydration）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Object hydration hydrates the result set into the object graph:

对象水合（hydration）将结果集水合（hydrate）为一个对象图：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_OBJECT);

Array Hydration // 数组水合（hydration）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can run the same query with array hydration and the result set
is hydrated into an array that represents the object graph:

你可以用数组水合执行同样的查询并将结果集水合为代表对象图的数组：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_ARRAY);

You can use the ``getArrayResult()`` shortcut as well:

你也可以使用 ``getArrayResult()`` 快捷方式：

.. code-block:: php

    <?php
    $users = $query->getArrayResult();

Scalar Hydration // 标量水合（hydration）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you want to return a flat rectangular result set instead of an
object graph you can use scalar hydration:

如果你希望返回一个扁平的矩形结果集替代一个对象图，你可以使用标量水合：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $users = $query->getResult(Query::HYDRATE_SCALAR);
    echo $users[0]['u_id'];

The following assumptions are made about selected fields using
Scalar Hydration:

以下假设使涉及被选字段使用标量水合：

1. Fields from classes are prefixed by the DQL alias in the result.
   A query of the kind 'SELECT u.name ..' returns a key 'u\_name' in
   the result rows.

   来自类的字段在结果中被加上了 DQL 别名作前缀。'SELECT u.name ..' 类型的查询
   ，在查询结果行中，返回一个键 'u\_name'。

Single Scalar Hydration // 单一标量水合（hydration）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you have a query which returns just a single scalar value you can use
single scalar hydration:

如果你有一条查询只返回单一标量值你可以使用单一标量水合：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT COUNT(a.id) FROM CmsUser u LEFT JOIN u.articles a WHERE u.username = ?1 GROUP BY u.id');
    $query->setParameter(1, 'jwage');
    $numArticles = $query->getResult(Query::HYDRATE_SINGLE_SCALAR);

You can use the ``getSingleScalarResult()`` shortcut as well:

你也可以使用 ``getSingleScalarResult()`` 快捷方式：

.. code-block:: php

    <?php
    $numArticles = $query->getSingleScalarResult();

Custom Hydration Modes // 自定义水合（hydtation）模式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can easily add your own custom hydration modes by first
creating a class which extends ``AbstractHydrator``:

你可以简单地添加自定义水合模式，首先创建一个扩展了 ``AbstractHydrator`` 的类：

.. code-block:: php

    <?php
    namespace MyProject\Hydrators;

    use Doctrine\ORM\Internal\Hydration\AbstractHydrator;

    class CustomHydrator extends AbstractHydrator
    {
        protected function _hydrateAll()
        {
            return $this->_stmt->fetchAll(PDO::FETCH_ASSOC);
        }
    }

Next you just need to add the class to the ORM configuration:

然后，你只需要添加此类到 ORM 配置：

.. code-block:: php

    <?php
    $em->getConfiguration()->addCustomHydrationMode('CustomHydrator', 'MyProject\Hydrators\CustomHydrator');

Now the hydrator is ready to be used in your queries:

现在该水合器（hydrator）已经可以使用在你的查询中：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM CmsUser u');
    $results = $query->getResult('CustomHydrator');

Iterating Large Result Sets // 迭代大结果集
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are situations when a query you want to execute returns a
very large result-set that needs to be processed. All the
previously described hydration modes completely load a result-set
into memory which might not be feasible with large result sets. See
the `Batch Processing <batch-processing.html>`_ section on details how
to iterate large result sets.

有些情况，你希望执行一个查询返回一个需要被处理的非常大的结果集。所有之前描述过的水合模式
都完整地加载结果集到内存中，这对于大的结果集可能并不可行。请查看 `批量处理 <batch-processing.html>`_
章节的如何迭代大结果集的细节。

Functions // 函数
~~~~~~~~~~~~~~~~~~~~~~

The following methods exist on the ``AbstractQuery`` which both
``Query`` and ``NativeQuery`` extend from.

以下方法存在于 ``AbstractQuery``,``Query`` 和 ``NativeQuery`` 都扩展于该抽象类。

Parameters // 参数
^^^^^^^^^^^^^^^^^^^^^^^^

Prepared Statements that use numerical or named wildcards require
additional parameters to be executable against the database. To
pass parameters to the query the following methods can be used:

使用数字或命名的通配符的预处理语句需要针对数据库的额外参数以变得可执行。可以使用以下方法传递参数到查询：

-  ``AbstractQuery::setParameter($param, $value)`` - Set the
   numerical or named wildcard to the given value.
-  ``AbstractQuery::setParameter($param, $value)`` - 设置数字或命名的通配符为给定值。
-  ``AbstractQuery::setParameters(array $params)`` - Set an array
   of parameter key-value pairs.
-  ``AbstractQuery::setParameters(array $params)`` - 设置一个键值对的数组参数。
-  ``AbstractQuery::getParameter($param)``
-  ``AbstractQuery::getParameter($param)`` - 获得给定参数的值。
-  ``AbstractQuery::getParameters()``
-  ``AbstractQuery::getParameters()`` - 获得多有参数的键值对数组。

Both named and positional parameters are passed to these methods without their ? or : prefix.

不带 ? 或 : 前缀的命名的和位置参数被传递到这些方法.

Cache related API // 缓存相关的 API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can cache query results based either on all variables that
define the result (SQL, Hydration Mode, Parameters and Hints) or on
user-defined cache keys. However by default query results are not
cached at all. You have to enable the result cache on a per query
basis. The following example shows a complete workflow using the
Result Cache API:

你可以基于结果集定义的所有变量（SQL 、水合模式、参数和提示）或用户定义的缓存键来缓存查询结果。
但是，默认查询结果完全不被缓存。你必须启用结果缓存在每一个查询基础上。以下示例展示使用结果缓存 API
的完整工作流：

.. code-block:: php

    <?php
    $query = $em->createQuery('SELECT u FROM MyProject\Model\User u WHERE u.id = ?1');
    $query->setParameter(1, 12);

    $query->setResultCacheDriver(new ApcCache());

    $query->useResultCache(true)
          ->setResultCacheLifeTime($seconds = 3600);

    $result = $query->getResult(); // cache miss

    $query->expireResultCache(true);
    $result = $query->getResult(); // forced expire, cache miss

    $query->setResultCacheId('my_query_result');
    $result = $query->getResult(); // saved in given result cache id.

    // or call useResultCache() with all parameters:
    $query->useResultCache(true, $seconds = 3600, 'my_query_result');
    $result = $query->getResult(); // cache hit!

    // Introspection
    $queryCacheProfile = $query->getQueryCacheProfile();
    $cacheDriver = $query->getResultCacheDriver();
    $lifetime = $query->getLifetime();
    $key = $query->getCacheKey();

.. note::

    You can set the Result Cache Driver globally on the
    ``Doctrine\ORM\Configuration`` instance so that it is passed to
    every ``Query`` and ``NativeQuery`` instance.

    你可以在 ``Doctrine\ORM\Configuration`` 实例上设置全局地结果缓存驱动。
    所以它将被传递给每一个 ``Query`` 和 ``NativeQuery`` 实例。

Query Hints // 查询提示
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can pass hints to the query parser and hydrators by using the
``AbstractQuery::setHint($name, $value)`` method. Currently there
exist mostly internal query hints that are not be consumed in
userland. However the following few hints are to be used in
userland:

你可以通过使用 ``AbstractQuery::setHint($name, $value)`` 方法传递提示（hints）到
查询解析器和水合器。当前存在的大部分内部查询提示没有在用户空间被消费。但是以下几个提示可以
被用于用户空间：

-  Query::HINT\_FORCE\_PARTIAL\_LOAD - Allows to hydrate objects
   although not all their columns are fetched. This query hint can be
   used to handle memory consumption problems with large result-sets
   that contain char or binary data. Doctrine has no way of implicitly
   reloading this data. Partially loaded objects have to be passed to
   ``EntityManager::refresh()`` if they are to be reloaded fully from
   the database.
-  Query::HINT\_FORCE\_PARTIAL\_LOAD - 允许水合对象，尽管不是它们所有的列被取回。
   此查询提示可以被用于处理使用包含字符或二进制数据的大结果集的内存消耗问题。Doctrine
   没有隐式重新加载此数据的办法。如果要从数据库完全地重新加载部分加载的对象必须将它们传递给
   ``EntityManager::refresh()``。
-  Query::HINT\_REFRESH - This query is used internally by
   ``EntityManager::refresh()`` and can be used in userland as well.
   If you specify this hint and a query returns the data for an entity
   that is already managed by the UnitOfWork, the fields of the
   existing entity will be refreshed. In normal operation a result-set
   that loads data of an already existing entity is discarded in favor
   of the already existing entity.
-  Query::HINT\_REFRESH - 此查询通过 ``EntityManager::refresh()`` 被使用在内部
   且也可以被用在用户空间。如果你指定该提示并查询返回已经由 UnitOfWork 托管的实体的数据，
   现有实体的字段将被更新（refresh）。在常规操作中，加载现有实体的数据的结果集将被丢弃有利于
   此现存实体。
-  Query::HINT\_CUSTOM\_TREE\_WALKERS - An array of additional
   ``Doctrine\ORM\Query\TreeWalker`` instances that are attached to
   the DQL query parsing process.
-  Query::HINT\_CUSTOM\_TREE\_WALKERS - 额外的 ``Doctrine\ORM\Query\TreeWalker`` 实例
   的数组被附上该 DQL 查询的解析过程。

Query Cache (DQL Query Only) // 查询缓存（仅 DQL 查询）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Parsing a DQL query and converting it into a SQL query against the
underlying database platform obviously has some overhead in
contrast to directly executing Native SQL queries. That is why
there is a dedicated Query Cache for caching the DQL parser
results. In combination with the use of wildcards you can reduce
the number of parsed queries in production to zero.

解析 DQL 查询并将它转换为针对底层数据库平台的 SQL 查询，很显然和直接执行原生 SQL
查询比较有一些开销。这就是为何有一个专门的查询缓存来缓存 DQL 解析器结果。在与使用的
通配符结合你可以在生成环境中减少解析查询的次数到0。

The Query Cache Driver is passed from the
``Doctrine\ORM\Configuration`` instance to each
``Doctrine\ORM\Query`` instance by default and is also enabled by
default. This also means you don't regularly need to fiddle with
the parameters of the Query Cache, however if you do there are
several methods to interact with it:

默认地，由 ``Doctrine\ORM\Configuration`` 实例传递查询缓存驱动到每个
``Doctrine\ORM\Query`` 实例并且默认也被启用。这也意味着你不必经常地调整查询缓存的这些参数，
但是如果你需要有几个方法可以与它交互：

-  ``Query::setQueryCacheDriver($driver)`` - Allows to set a Cache
   instance
-  ``Query::setQueryCacheDriver($driver)`` - 允许设置缓存实例。
-  ``Query::setQueryCacheLifeTime($seconds = 3600)`` - Set lifetime
   of the query caching.
-  ``Query::setQueryCacheLifeTime($seconds = 3600)`` - 设置查询缓存的生命周期。
-  ``Query::expireQueryCache($bool)`` - Enforce the expiring of the
   query cache if set to true.
-  ``Query::expireQueryCache($bool)`` - 如果设置为 true，强制查询缓存过期。
-  ``Query::getExpireQueryCache()``
-  ``Query::getExpireQueryCache()`` - 获得查询缓存过期时间。
-  ``Query::getQueryCacheDriver()``
-  ``Query::getQueryCacheDriver()`` - 获得查询缓存驱动。
-  ``Query::getQueryCacheLifeTime()``
-  ``Query::getQueryCacheLifeTime()`` - 获得查询缓存的生命周期。

First and Max Result Items (DQL Query Only) // 起始和最大的结果条目（仅 DQL 查询）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can limit the number of results returned from a DQL query as
well as specify the starting offset, Doctrine then uses a strategy
of manipulating the select query to return only the requested
number of results:

你可以限制从 DQL 查询返回的结果的数量以及指定起始偏移，Doctrine 之后使用一个操作 SELECT 查询的策略以
只返回请求的结果数量：

-  ``Query::setMaxResults($maxResults)``
-  ``Query::setMaxResults($maxResults)`` - 设置最大的结果数。
-  ``Query::setFirstResult($offset)``
-  ``Query::setFirstResult($offset)`` - 设置查询结果的起始偏移。

.. note::

    If your query contains a fetch-joined collection
    specifying the result limit methods are not working as you would
    expect. Set Max Results restricts the number of database result
    rows, however in the case of fetch-joined collections one root
    entity might appear in many rows, effectively hydrating less than
    the specified number of results.

    如果你的查询包含一个 fetch-joined 集合，指定结果限制方法不会如你预期那样工作。
    设置最大的结果数限制数据库结果行的数量，但是在 fetch-joined 的集合情况下，一个根实体
    可能出现在多行中，事实上，水化的少于指定的结果的数量。

.. _dql-temporarily-change-fetch-mode:

Temporarily change fetch mode in DQL // 在 DQL 中临时变更取回（fetch）模式
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

While normally all your associations are marked as lazy or extra lazy you will have cases where you are using DQL and don't want to
fetch join a second, third or fourth level of entities into your result, because of the increased cost of the SQL JOIN. You
can mark a many-to-one or one-to-one association as fetched temporarily to batch fetch these entities using a WHERE .. IN query.

尽管通常所有关联被标记为懒的或特别懒，你将有使用 DQL 的情况且不希望 fetch join 实体的第二、三、四层到你的结果中，因为增加的 SQL JOIN 的开销。
你可以标记 many-to-one 或 one-to-one 关联为临时地取回以使用 WHERE .. IN 查询批量取回这些实体。

.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT u FROM MyProject\User u");
    $query->setFetchMode("MyProject\User", "address", \Doctrine\ORM\Mapping\ClassMetadata::FETCH_EAGER);
    $query->execute();

Given that there are 10 users and corresponding addresses in the database the executed queries will look something like:

假定在数据库中有10个用户及其相应地地址，执行的查询将看起来像这样：

.. code-block:: sql

    SELECT * FROM users;
    SELECT * FROM address WHERE id IN (1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

.. note::
    Changing the fetch mode during a query is only possible for one-to-one and many-to-one relations.

    在查询期间修改取回模式仅对 one-to-one 和 many-to-one 关联是可能。

EBNF
----

The following context-free grammar, written in an EBNF variant,
describes the Doctrine Query Language. You can consult this grammar
whenever you are unsure about what is possible with DQL or what the
correct syntax for a particular query should be.

以下上下文无关（context-free）的语法，使用 EBNF 变体编写，描述 Doctrine 查询语言。
你可以查阅此语法，每当你不确定关于使用 DQL 什么是可能的或部分查询正确的语法应该是什么时。

Document syntax: // 文档语法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  non-terminals begin with an upper case character
-  非终端使用大写字符开头
-  terminals begin with a lower case character
-  终端使用小写字符开头
-  parentheses (...) are used for grouping
-  圆括号 (...) 被用于分组
-  square brackets [...] are used for defining an optional part,
   e.g. zero or one time
-  方括号 [...] 被用于定义一个可选的部分，例如0或1次
-  curly brackets {...} are used for repetition, e.g. zero or more
   times
-  大括号 {...} 被用于重复，例如0或多次
-  double quotation marks "..." define a terminal string
-  双引号标记 "..." 定义一个终端字符串
-  a vertical bar \| represents an alternative
-  一个竖线代表另一个替换

Terminals // 终端
~~~~~~~~~~~~~~~~~~~~~~


-  identifier (name, email, ...) must match ``[a-z_][a-z0-9_]*``
-  标识符 (name, email, ...) 必须匹配 ``[a-z_][a-z0-9_]*``
-  fully_qualified_name (Doctrine\Tests\Models\CMS\CmsUser) matches PHP's fully qualified class names
-  完全限定名 (Doctrine\Tests\Models\CMS\CmsUser) 匹配 PHP 的完全限定类名
-  aliased_name (CMS:CmsUser) uses two identifiers, one for the namespace alias and one for the class inside it
-  别名 (CMS:CmsUser) 使用两个标识符，一个用于命名空间别名，一个用于其中的类
-  string ('foo', 'bar''s house', '%ninja%', ...)
-  字符串('foo', 'bar''s house', '%ninja%', ...)
-  char ('/', '\\', ' ', ...)
-  字符 ('/', '\\', ' ', ...)
-  integer (-1, 0, 1, 34, ...)
-  整型数 (-1, 0, 1, 34, ...)
-  float (-0.23, 0.007, 1.245342E+8, ...)
-  浮点型数 (-0.23, 0.007, 1.245342E+8, ...)
-  boolean (false, true)
-  布尔型 (false, true)

Query Language // 查询语言
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    QueryLanguage ::= SelectStatement | UpdateStatement | DeleteStatement

Statements // 语句
~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    SelectStatement ::= SelectClause FromClause [WhereClause] [GroupByClause] [HavingClause] [OrderByClause]
    UpdateStatement ::= UpdateClause [WhereClause]
    DeleteStatement ::= DeleteClause [WhereClause]

Identifiers // 标识符
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    /* Alias Identification usage (the "u" of "u.name") */
    /* 别名标识用法（"u.name" 的 "u"） */
    IdentificationVariable ::= identifier

    /* Alias Identification declaration (the "u" of "FROM User u") */
    /* 别名标识声明（"FROM User u" 的 "u" */
    AliasIdentificationVariable :: = identifier

    /* identifier that must be a class name (the "User" of "FROM User u"), possibly as a fully qualified class name or namespace-aliased */
    /* 标识符必须是一个类名（"FROM User u" 的 "User"），或许作为一个完全限定类名或命名空间别名 */
    AbstractSchemaName ::= fully_qualified_name | aliased_name | identifier

    /* Alias ResultVariable declaration (the "total" of "COUNT(*) AS total") */
    /* 别名结果变量声明（"COUNT(*) AS total" 的 "total"） */
    AliasResultVariable = identifier

    /* ResultVariable identifier usage of mapped field aliases (the "total" of "COUNT(*) AS total") */
    /* 结果变量标识符映射字段别名用法（"COUNT(*) AS total" 的 "total"） */
    ResultVariable = identifier

    /* identifier that must be a field (the "name" of "u.name") */
    /* 标识符必须是一个字段（ "u.name" 的 "name"） */
    /* This is responsible to know if the field exists in Object, no matter if it's a relation or a simple field */
    /* 有负责知道该字段是否存在于对象中，无论它是一个关联还是一个简单的字段 */
    FieldIdentificationVariable ::= identifier

    /* identifier that must be a collection-valued association field (to-many) (the "Phonenumbers" of "u.Phonenumbers") */
    /* 字段必须是一个集合值关联字段（to-many）（"u.Phonenumbers" 的 "Phonenumbers"） */
    CollectionValuedAssociationField ::= FieldIdentificationVariable

    /* identifier that must be a single-valued association field (to-one) (the "Group" of "u.Group") */
    /* 标识符必须是一个单一值关联字段（to-one）（"u.Group" 的 "Group"） */
    SingleValuedAssociationField ::= FieldIdentificationVariable

    /* identifier that must be an embedded class state field */
    /* 标识符必须是一个嵌入的类状态字段*/

    EmbeddedClassStateField ::= FieldIdentificationVariable

    /* identifier that must be a simple state field (name, email, ...) (the "name" of "u.name") */
    /* 标识符必须是一个简单的状态字段（name、email、 ...）（"u.name" 的 "name"） */
    /* The difference between this and FieldIdentificationVariable is only semantical, because it points to a single field (not mapping to a relation) */
    /* 这和 FieldIdentificationVariable 之间的不同仅是语义上的，因为它指向一个单一字段（没有映射至一个关联） */
    SimpleStateField ::= FieldIdentificationVariable

Path Expressions //路径匹配
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    /* "u.Group" or "u.Phonenumbers" declarations */
    /* "u.Group" 或 "u.Phonenumbers" 声明 */
    JoinAssociationPathExpression             ::= IdentificationVariable "." (CollectionValuedAssociationField | SingleValuedAssociationField)

    /* "u.Group" or "u.Phonenumbers" usages */
    /* "u.Group" 或 "u.Phonenumbers" 用法 */
    AssociationPathExpression                 ::= CollectionValuedPathExpression | SingleValuedAssociationPathExpression

    /* "u.name" or "u.Group" */
    /* "u.name" 或 "u.Group" */
    SingleValuedPathExpression                ::= StateFieldPathExpression | SingleValuedAssociationPathExpression

    /* "u.name" or "u.Group.name" */
    /* "u.name" 或 "u.Group.name" */
    StateFieldPathExpression                  ::= IdentificationVariable "." StateField

    /* "u.Group" */
    SingleValuedAssociationPathExpression     ::= IdentificationVariable "." SingleValuedAssociationField

    /* "u.Group.Permissions" */
    CollectionValuedPathExpression            ::= IdentificationVariable "." CollectionValuedAssociationField

    /* "name" */
    StateField                                ::= {EmbeddedClassStateField "."}* SimpleStateField

Clauses // 子句
~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    SelectClause        ::= "SELECT" ["DISTINCT"] SelectExpression {"," SelectExpression}*
    SimpleSelectClause  ::= "SELECT" ["DISTINCT"] SimpleSelectExpression
    UpdateClause        ::= "UPDATE" AbstractSchemaName ["AS"] AliasIdentificationVariable "SET" UpdateItem {"," UpdateItem}*
    DeleteClause        ::= "DELETE" ["FROM"] AbstractSchemaName ["AS"] AliasIdentificationVariable
    FromClause          ::= "FROM" IdentificationVariableDeclaration {"," IdentificationVariableDeclaration}*
    SubselectFromClause ::= "FROM" SubselectIdentificationVariableDeclaration {"," SubselectIdentificationVariableDeclaration}*
    WhereClause         ::= "WHERE" ConditionalExpression
    HavingClause        ::= "HAVING" ConditionalExpression
    GroupByClause       ::= "GROUP" "BY" GroupByItem {"," GroupByItem}*
    OrderByClause       ::= "ORDER" "BY" OrderByItem {"," OrderByItem}*
    Subselect           ::= SimpleSelectClause SubselectFromClause [WhereClause] [GroupByClause] [HavingClause] [OrderByClause]

Items // 条目
~~~~~~~~~~~~~~~~~~

.. code-block:: php

    UpdateItem  ::= SingleValuedPathExpression "=" NewValue
    OrderByItem ::= (SimpleArithmeticExpression | SingleValuedPathExpression | ScalarExpression | ResultVariable | FunctionDeclaration) ["ASC" | "DESC"]
    GroupByItem ::= IdentificationVariable | ResultVariable | SingleValuedPathExpression
    NewValue    ::= SimpleArithmeticExpression | "NULL"

From, Join and Index by
~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    IdentificationVariableDeclaration          ::= RangeVariableDeclaration [IndexBy] {Join}*
    SubselectIdentificationVariableDeclaration ::= IdentificationVariableDeclaration
    RangeVariableDeclaration                   ::= AbstractSchemaName ["AS"] AliasIdentificationVariable
    JoinAssociationDeclaration                 ::= JoinAssociationPathExpression ["AS"] AliasIdentificationVariable [IndexBy]
    Join                                       ::= ["LEFT" ["OUTER"] | "INNER"] "JOIN" (JoinAssociationDeclaration | RangeVariableDeclaration) ["WITH" ConditionalExpression]
    IndexBy                                    ::= "INDEX" "BY" StateFieldPathExpression

Select Expressions // SELECT 表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    SelectExpression        ::= (IdentificationVariable | ScalarExpression | AggregateExpression | FunctionDeclaration | PartialObjectExpression | "(" Subselect ")" | CaseExpression | NewObjectExpression) [["AS"] ["HIDDEN"] AliasResultVariable]
    SimpleSelectExpression  ::= (StateFieldPathExpression | IdentificationVariable | FunctionDeclaration | AggregateExpression | "(" Subselect ")" | ScalarExpression) [["AS"] AliasResultVariable]
    PartialObjectExpression ::= "PARTIAL" IdentificationVariable "." PartialFieldSet
    PartialFieldSet         ::= "{" SimpleStateField {"," SimpleStateField}* "}"
    NewObjectExpression     ::= "NEW" AbstractSchemaName "(" NewObjectArg {"," NewObjectArg}* ")"
    NewObjectArg            ::= ScalarExpression | "(" Subselect ")"

Conditional Expressions // 条件表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ConditionalExpression       ::= ConditionalTerm {"OR" ConditionalTerm}*
    ConditionalTerm             ::= ConditionalFactor {"AND" ConditionalFactor}*
    ConditionalFactor           ::= ["NOT"] ConditionalPrimary
    ConditionalPrimary          ::= SimpleConditionalExpression | "(" ConditionalExpression ")"
    SimpleConditionalExpression ::= ComparisonExpression | BetweenExpression | LikeExpression |
                                    InExpression | NullComparisonExpression | ExistsExpression |
                                    EmptyCollectionComparisonExpression | CollectionMemberExpression |
                                    InstanceOfExpression


Collection Expressions // 集合表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    EmptyCollectionComparisonExpression ::= CollectionValuedPathExpression "IS" ["NOT"] "EMPTY"
    CollectionMemberExpression          ::= EntityExpression ["NOT"] "MEMBER" ["OF"] CollectionValuedPathExpression

Literal Values // 字面量值
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    Literal     ::= string | char | integer | float | boolean
    InParameter ::= Literal | InputParameter

Input Parameter // 输入参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    InputParameter      ::= PositionalParameter | NamedParameter
    PositionalParameter ::= "?" integer
    NamedParameter      ::= ":" string

Arithmetic Expressions // 算术运算表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ArithmeticExpression       ::= SimpleArithmeticExpression | "(" Subselect ")"
    SimpleArithmeticExpression ::= ArithmeticTerm {("+" | "-") ArithmeticTerm}*
    ArithmeticTerm             ::= ArithmeticFactor {("*" | "/") ArithmeticFactor}*
    ArithmeticFactor           ::= [("+" | "-")] ArithmeticPrimary
    ArithmeticPrimary          ::= SingleValuedPathExpression | Literal | "(" SimpleArithmeticExpression ")"
                                   | FunctionsReturningNumerics | AggregateExpression | FunctionsReturningStrings
                                   | FunctionsReturningDatetime | IdentificationVariable | ResultVariable
                                   | InputParameter | CaseExpression

Scalar and Type Expressions // 标量（scalar）和类型表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    ScalarExpression       ::= SimpleArithmeticExpression | StringPrimary | DateTimePrimary | StateFieldPathExpression | BooleanPrimary | CaseExpression | InstanceOfExpression
    StringExpression       ::= StringPrimary | ResultVariable | "(" Subselect ")"
    StringPrimary          ::= StateFieldPathExpression | string | InputParameter | FunctionsReturningStrings | AggregateExpression | CaseExpression
    BooleanExpression      ::= BooleanPrimary | "(" Subselect ")"
    BooleanPrimary         ::= StateFieldPathExpression | boolean | InputParameter
    EntityExpression       ::= SingleValuedAssociationPathExpression | SimpleEntityExpression
    SimpleEntityExpression ::= IdentificationVariable | InputParameter
    DatetimeExpression     ::= DatetimePrimary | "(" Subselect ")"
    DatetimePrimary        ::= StateFieldPathExpression | InputParameter | FunctionsReturningDatetime | AggregateExpression

.. note::

    Parts of CASE expressions are not yet implemented.

    部分 CASE 表达式还未被实现。

Aggregate Expressions // 聚合表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    AggregateExpression ::= ("AVG" | "MAX" | "MIN" | "SUM" | "COUNT") "(" ["DISTINCT"] SimpleArithmeticExpression ")"

Case Expressions // CASE 表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    CaseExpression        ::= GeneralCaseExpression | SimpleCaseExpression | CoalesceExpression | NullifExpression
    GeneralCaseExpression ::= "CASE" WhenClause {WhenClause}* "ELSE" ScalarExpression "END"
    WhenClause            ::= "WHEN" ConditionalExpression "THEN" ScalarExpression
    SimpleCaseExpression  ::= "CASE" CaseOperand SimpleWhenClause {SimpleWhenClause}* "ELSE" ScalarExpression "END"
    CaseOperand           ::= StateFieldPathExpression | TypeDiscriminator
    SimpleWhenClause      ::= "WHEN" ScalarExpression "THEN" ScalarExpression
    CoalesceExpression    ::= "COALESCE" "(" ScalarExpression {"," ScalarExpression}* ")"
    NullifExpression      ::= "NULLIF" "(" ScalarExpression "," ScalarExpression ")"

Other Expressions // 其他表达式
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

QUANTIFIED/BETWEEN/COMPARISON/LIKE/NULL/EXISTS

.. code-block:: php

    QuantifiedExpression     ::= ("ALL" | "ANY" | "SOME") "(" Subselect ")"
    BetweenExpression        ::= ArithmeticExpression ["NOT"] "BETWEEN" ArithmeticExpression "AND" ArithmeticExpression
    ComparisonExpression     ::= ArithmeticExpression ComparisonOperator ( QuantifiedExpression | ArithmeticExpression )
    InExpression             ::= SingleValuedPathExpression ["NOT"] "IN" "(" (InParameter {"," InParameter}* | Subselect) ")"
    InstanceOfExpression     ::= IdentificationVariable ["NOT"] "INSTANCE" ["OF"] (InstanceOfParameter | "(" InstanceOfParameter {"," InstanceOfParameter}* ")")
    InstanceOfParameter      ::= AbstractSchemaName | InputParameter
    LikeExpression           ::= StringExpression ["NOT"] "LIKE" StringPrimary ["ESCAPE" char]
    NullComparisonExpression ::= (InputParameter | NullIfExpression | CoalesceExpression | AggregateExpression | FunctionDeclaration | IdentificationVariable | SingleValuedPathExpression | ResultVariable) "IS" ["NOT"] "NULL"
    ExistsExpression         ::= ["NOT"] "EXISTS" "(" Subselect ")"
    ComparisonOperator       ::= "=" | "<" | "<=" | "<>" | ">" | ">=" | "!="

Functions // 函数
~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: php

    FunctionDeclaration ::= FunctionsReturningStrings | FunctionsReturningNumerics | FunctionsReturningDateTime

    FunctionsReturningNumerics ::=
            "LENGTH" "(" StringPrimary ")" |
            "LOCATE" "(" StringPrimary "," StringPrimary ["," SimpleArithmeticExpression]")" |
            "ABS" "(" SimpleArithmeticExpression ")" |
            "SQRT" "(" SimpleArithmeticExpression ")" |
            "MOD" "(" SimpleArithmeticExpression "," SimpleArithmeticExpression ")" |
            "SIZE" "(" CollectionValuedPathExpression ")" |
            "DATE_DIFF" "(" ArithmeticPrimary "," ArithmeticPrimary ")" |
            "BIT_AND" "(" ArithmeticPrimary "," ArithmeticPrimary ")" |
            "BIT_OR" "(" ArithmeticPrimary "," ArithmeticPrimary ")"

    FunctionsReturningDateTime ::=
            "CURRENT_DATE" |
            "CURRENT_TIME" |
            "CURRENT_TIMESTAMP" |
            "DATE_ADD" "(" ArithmeticPrimary "," ArithmeticPrimary "," StringPrimary ")" |
            "DATE_SUB" "(" ArithmeticPrimary "," ArithmeticPrimary "," StringPrimary ")"

    FunctionsReturningStrings ::=
            "CONCAT" "(" StringPrimary "," StringPrimary ")" |
            "SUBSTRING" "(" StringPrimary "," SimpleArithmeticExpression "," SimpleArithmeticExpression ")" |
            "TRIM" "(" [["LEADING" | "TRAILING" | "BOTH"] [char] "FROM"] StringPrimary ")" |
            "LOWER" "(" StringPrimary ")" |
            "UPPER" "(" StringPrimary ")" |
            "IDENTITY" "(" SingleValuedAssociationPathExpression {"," string} ")"


