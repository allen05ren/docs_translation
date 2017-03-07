The QueryBuilder // 查询构造器
====================================

A ``QueryBuilder`` provides an API that is designed for
conditionally constructing a DQL query in several steps.

``QueryBuilder`` 提供了一个 API，该 API 被设计为在几个步骤中有条件地构建一条 DQL 查询。

It provides a set of classes and methods that is able to
programmatically build queries, and also provides a fluent API.
This means that you can change between one methodology to the other
as you want, and also pick one if you prefer.

它提供了一组类和方法能够以编程的方式构建查询，且也提交了流程的 API。
这意味着你可以按照你希望的在方法之间切换，如果你喜欢也可以选择一个。

Constructing a new QueryBuilder object // 构造一个新的 QueryBuilder 对象
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The same way you build a normal Query, you build a ``QueryBuilder``
object, just providing the correct method name. Here is an example
how to build a ``QueryBuilder`` object:

同样的方式你构建一个常规查询，你构建一个 ``QueryBuilder`` 对象，只要提供一个正确的方法名。
这里有一个如何构建一个 ``QueryBuilder`` 对象的例子：

.. code-block:: php

    <?php
    // $em instanceof EntityManager

    // example1: creating a QueryBuilder instance
    $qb = $em->createQueryBuilder();

Once you have created an instance of QueryBuilder, it provides a
set of useful informative functions that you can use. One good
example is to inspect what type of object the ``QueryBuilder`` is.

一旦你已经创建了 QueryBuilder 的实例，它提供了一组有用的信息函数你可以使用。一个好的检查
``QueryBuilder`` 对象是什么类型的例子。

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // example2: retrieving type of QueryBuilder
    echo $qb->getType(); // Prints: 0

There're currently 3 possible return values for ``getType()``:

``getType()`` 当前有三种可能的返回值：

-  ``QueryBuilder::SELECT``, which returns value 0
-  ``QueryBuilder::SELECT``, 返回值 0
-  ``QueryBuilder::DELETE``, returning value 1
-  ``QueryBuilder::DELETE``, 返回值 1
-  ``QueryBuilder::UPDATE``, which returns value 2
-  ``QueryBuilder::UPDATE``, 返回值 2

It is possible to retrieve the associated ``EntityManager`` of the
current ``QueryBuilder``, its DQL and also a ``Query`` object when
you finish building your DQL.

可以取回当前 ``QueryBuilder`` 关联的 ``EntityManager``，及其 DQL以及当你完成
构建你的 DQL 时的 ``Query`` 对象。

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // example3: retrieve the associated EntityManager
    $em = $qb->getEntityManager();

    // example4: retrieve the DQL string of what was defined in QueryBuilder
    $dql = $qb->getDql();

    // example5: retrieve the associated Query object with the processed DQL
    $q = $qb->getQuery();

Internally, ``QueryBuilder`` works with a DQL cache to increase
performance. Any changes that may affect the generated DQL actually
modifies the state of ``QueryBuilder`` to a stage we call
STATE\_DIRTY. One ``QueryBuilder`` can be in two different states:

在内部，``QueryBuilder`` 使用 DQL 缓存工作以提升性能。任何可能影响生成的 DQL 的变更实际上修改了
``QueryBuilder`` 的状态为暂存（stage），我们称之为 STATE\_DIRTY。一个 ``QueryBuilder``
可以处在两种不同的状态：

-  ``QueryBuilder::STATE_CLEAN``, which means DQL haven't been
   altered since last retrieval or nothing were added since its
   instantiation
-  ``QueryBuilder::STATE_CLEAN`` - 意味着自从最近一次取回 DQL 还未被修改或者
   从实例化之后未添加任何东西。
-  ``QueryBuilder::STATE_DIRTY``, means DQL query must (and will)
   be processed on next retrieval
-  ``QueryBuilder::STATE_DIRTY`` - 意味着 DQL 查询必须（和将）被处理在下一次取回。

Working with QueryBuilder // 使用 QueryBuilder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


High level API methods // 高层 API 方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To simplify even more the way you build a query in Doctrine, we can take
advantage of what we call Helper methods. For all base code, there
is a set of useful methods to simplify a programmer's life. To
illustrate how to work with them, here is the same example 6
re-written using ``QueryBuilder`` helper methods:

为了简化你在 Doctrine 中构建查询的方式，我们可以利用我们称之为辅助（helper）方法。对于所有
基础代码，有一组有用的方法来简化程序员的生活。为了说明如何使用它们，这里有一个相同的使用 ``QueryBuilder``
的辅助方法重写的例子6：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = ?1')
       ->orderBy('u.name', 'ASC');

``QueryBuilder`` helper methods are considered the standard way to
build DQL queries. Although it is supported, it should be avoided
to use string based queries and greatly encouraged to use
``$qb->expr()->*`` methods. Here is a converted example 8 to
suggested standard way to build queries:

``QueryBuilder`` 辅助方法被认为是构建 DQL 查询的标准方法。尽管支持，也应该被避免
用于基于字符串的查询并且非常鼓励使用 ``$qb->expr()->*`` 方法。这里是一个转换的例子8以
推荐的标准方式来构建查询：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    $qb->select(array('u')) // string 'u' is converted to array internally
       ->from('User', 'u')
       ->where($qb->expr()->orX(
           $qb->expr()->eq('u.id', '?1'),
           $qb->expr()->like('u.nickname', '?2')
       ))
       ->orderBy('u.surname', 'ASC');

Here is a complete list of helper methods available in ``QueryBuilder``:

这里是一个在 ``QueryBuilder`` 中可用的辅助方法的完整列表：

.. code-block:: php

    <?php
    class QueryBuilder
    {
        // Example - $qb->select('u')
        // Example - $qb->select(array('u', 'p'))
        // Example - $qb->select($qb->expr()->select('u', 'p'))
        public function select($select = null);
        
        // addSelect does not override previous calls to select
        // addSelect 不会覆盖之前调用的 SELECT
        //
        // Example - $qb->select('u');
        //              ->addSelect('p.area_code');
        public function addSelect($select = null);

        // Example - $qb->delete('User', 'u')
        public function delete($delete = null, $alias = null);

        // Example - $qb->update('Group', 'g')
        public function update($update = null, $alias = null);

        // Example - $qb->set('u.firstName', $qb->expr()->literal('Arnold'))
        // Example - $qb->set('u.numChilds', 'u.numChilds + ?1')
        // Example - $qb->set('u.numChilds', $qb->expr()->sum('u.numChilds', '?1'))
        public function set($key, $value);

        // Example - $qb->from('Phonenumber', 'p')
        // Example - $qb->from('Phonenumber', 'p', 'p.id')
        public function from($from, $alias, $indexBy = null);

        // Example - $qb->join('u.Group', 'g', Expr\Join::WITH, $qb->expr()->eq('u.status_id', '?1'))
        // Example - $qb->join('u.Group', 'g', 'WITH', 'u.status = ?1')
        // Example - $qb->join('u.Group', 'g', 'WITH', 'u.status = ?1', 'g.id')
        public function join($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // Example - $qb->innerJoin('u.Group', 'g', Expr\Join::WITH, $qb->expr()->eq('u.status_id', '?1'))
        // Example - $qb->innerJoin('u.Group', 'g', 'WITH', 'u.status = ?1')
        // Example - $qb->innerJoin('u.Group', 'g', 'WITH', 'u.status = ?1', 'g.id')
        public function innerJoin($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // Example - $qb->leftJoin('u.Phonenumbers', 'p', Expr\Join::WITH, $qb->expr()->eq('p.area_code', 55))
        // Example - $qb->leftJoin('u.Phonenumbers', 'p', 'WITH', 'p.area_code = 55')
        // Example - $qb->leftJoin('u.Phonenumbers', 'p', 'WITH', 'p.area_code = 55', 'p.id')
        public function leftJoin($join, $alias, $conditionType = null, $condition = null, $indexBy = null);

        // NOTE: ->where() overrides all previously set conditions
        // 注意： ->where() 覆盖所有之前设置的条件
        //
        // Example - $qb->where('u.firstName = ?1', $qb->expr()->eq('u.surname', '?2'))
        // Example - $qb->where($qb->expr()->andX($qb->expr()->eq('u.firstName', '?1'), $qb->expr()->eq('u.surname', '?2')))
        // Example - $qb->where('u.firstName = ?1 AND u.surname = ?2')
        public function where($where);

        // NOTE: ->andWhere() can be used directly, without any ->where() before
        // 注意： ->andWhere() 可以前面不带任何的 ->where() 直接地被使用
        //
        // Example - $qb->andWhere($qb->expr()->orX($qb->expr()->lte('u.age', 40), 'u.numChild = 0'))
        public function andWhere($where);

        // Example - $qb->orWhere($qb->expr()->between('u.id', 1, 10));
        public function orWhere($where);

        // NOTE: -> groupBy() overrides all previously set grouping conditions
        // 注意： -> groupBy() 覆盖所有之前设置的 grouping 条件
        //
        // Example - $qb->groupBy('u.id')
        public function groupBy($groupBy);

        // Example - $qb->addGroupBy('g.name')
        public function addGroupBy($groupBy);

        // NOTE: -> having() overrides all previously set having conditions
        // 注意： -> having() 覆盖所有之前设置的 having 条件
        //
        // Example - $qb->having('u.salary >= ?1')
        // Example - $qb->having($qb->expr()->gte('u.salary', '?1'))
        public function having($having);

        // Example - $qb->andHaving($qb->expr()->gt($qb->expr()->count('u.numChild'), 0))
        public function andHaving($having);

        // Example - $qb->orHaving($qb->expr()->lte('g.managerLevel', '100'))
        public function orHaving($having);

        // NOTE: -> orderBy() overrides all previously set ordering conditions
        // 注意： -> orderBy() 覆盖所有之前设置的 ordering 条件
        //
        // Example - $qb->orderBy('u.surname', 'DESC')
        public function orderBy($sort, $order = null);

        // Example - $qb->addOrderBy('u.firstName')
        public function addOrderBy($sort, $order = null); // Default $order = 'ASC'
    }

Binding parameters to your query // 绑定参数至查询
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Doctrine supports dynamic binding of parameters to your query,
similar to preparing queries. You can use both strings and numbers
as placeholders, although both have a slightly different syntax.
Additionally, you must make your choice: Mixing both styles is not
allowed. Binding parameters can simply be achieved as follows:

Doctrine 支持动态绑定参数至查询，类似于预处理查询。你可以使用字符串和数字作为占位符，尽管两者语法有
稍微的不同。另外，你必须做出选择：混合两者风格是不被允许的。绑定参数可以轻易地达成如下：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = ?1')
       ->orderBy('u.name', 'ASC')
       ->setParameter(1, 100); // Sets ?1 to 100, and thus we will fetch a user with u.id = 100

You are not forced to enumerate your placeholders as the
alternative syntax is available:

你不必强制枚举你的占位符，因为备选的语法是可用的：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    $qb->select('u')
       ->from('User', 'u')
       ->where('u.id = :identifier')
       ->orderBy('u.name', 'ASC')
       ->setParameter('identifier', 100); // Sets :identifier to 100, and thus we will fetch a user with u.id = 100

Note that numeric placeholders start with a ? followed by a number
while the named placeholders start with a : followed by a string.

注意数字占位符起始于一个 ? 后跟随一个数字虽然命名占位符起始于一个 : 后跟随一个字符串。

Calling ``setParameter()`` automatically infers which type you are setting as
value. This works for integers, arrays of strings/integers, DateTime instances
and for managed entities. If you want to set a type explicitly you can call
the third argument to ``setParameter()`` explicitly. It accepts either a PDO
type or a DBAL Type name for conversion.

调用 ``setParameter()`` 自动地推断你正在设置作为值的类型。这适用于整型数、字符串/整型数的数组、DateTime 实例和
托管的（managed）实体。如果你希望明确地设置一个类型，你可以明确地传递第三个参数给 ``setParameter()`` 来调用。
它接受一个 PDO 类型或 DBAL 类型名称来进行转换。

If you've got several parameters to bind to your query, you can
also use setParameters() instead of setParameter() with the
following syntax:

如果你必须绑定多个参数至查询，你也可以使用 setParameters() 替代 setParameter()，使用以下语法：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // Query here...
    $qb->setParameters(array(1 => 'value for ?1', 2 => 'value for ?2'));

Getting already bound parameters is easy - simply use the above
mentioned syntax with "getParameter()" or "getParameters()":

获取已经绑定的参数是简单的 - 简单地用上面提到的语法使用 "getParameter()" 或 "getParameters()"：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // See example above
    $params = $qb->getParameters();
    // $params instanceof \Doctrine\Common\Collections\ArrayCollection

    // Equivalent to
    $param = $qb->getParameter(1);
    // $param instanceof \Doctrine\ORM\Query\Parameter

Note: If you try to get a parameter that was not bound yet,
getParameter() simply returns NULL.

注意：如果你尝试获得一个还未绑定的参数，getParameter() 简单地返回 NULL。

The API of a Query Parameter is:

查询参数的 API 是：

.. code-block:: php

    namespace Doctrine\ORM\Query;

    class Parameter
    {
        public function getName();
        public function getValue();
        public function getType();
        public function setValue($value, $type = null);
    }

Limiting the Result // 限制结果
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To limit a result the query builder has some methods in common with
the Query object which can be retrieved from ``EntityManager#createQuery()``.

为了限制结果查询构造器有几个与可以从 ``EntityManager#createQuery()`` 取回的查询对象相同的方法。

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder
    $offset = (int)$_GET['offset'];
    $limit = (int)$_GET['limit'];

    $qb->add('select', 'u')
       ->add('from', 'User u')
       ->add('orderBy', 'u.name ASC')
       ->setFirstResult( $offset )
       ->setMaxResults( $limit );

Executing a Query // 执行查询
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The QueryBuilder is a builder object only, it has no means of actually
executing the Query. Additionally a set of parameters such as query hints
cannot be set on the QueryBuilder itself. This is why you always have to convert
a querybuilder instance into a Query object:

QueryBuilder 仅是一个构建器对象，它并不意味着实际地执行该查询。另外一组参数不能在 QueryBuilder自身上
设置，如查询提示。这就是为何你始终需要转换 QueryBuilder 实例为查询对象：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder
    $query = $qb->getQuery();

    // Set additional Query options
    $query->setQueryHint('foo', 'bar');
    $query->useResultCache('my_cache_id');

    // Execute Query
    $result = $query->getResult();
    $single = $query->getSingleResult();
    $array = $query->getArrayResult();
    $scalar = $query->getScalarResult();
    $singleScalar = $query->getSingleScalarResult();

The Expr class // Expr 类
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To workaround some of the issues that ``add()`` method may cause,
Doctrine created a class that can be considered as a helper for
building expressions. This class is called ``Expr``, which provides a
set of useful methods to help build expressions:

为了暂时避开 ``add()`` 方法可能导致的一些问题，Doctrine 创建了一个类，它可以被视为一个
构建表达式的辅助。此类被称为 ``Expr``，它提供了一组有用的方法来帮助构建表达式：

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // example8: QueryBuilder port of:
    // "SELECT u FROM User u WHERE u.id = ? OR u.nickname LIKE ? ORDER BY u.name ASC" using Expr class
    $qb->add('select', new Expr\Select(array('u')))
       ->add('from', new Expr\From('User', 'u'))
       ->add('where', $qb->expr()->orX(
           $qb->expr()->eq('u.id', '?1'),
           $qb->expr()->like('u.nickname', '?2')
       ))
       ->add('orderBy', new Expr\OrderBy('u.name', 'ASC'));

Although it still sounds complex, the ability to programmatically
create conditions are the main feature of ``Expr``. Here it is a
complete list of supported helper methods available:

虽然它仍然听起来很复杂，但是以编程方式创建条件的能力是 ``Expr`` 的主要特性。
这里是一个已支持的可用的辅助方法的完整列表：

.. code-block:: php

    <?php
    class Expr
    {
        /** Conditional objects **/
        /** 条件对象 **/

        // Example - $qb->expr()->andX($cond1 [, $condN])->add(...)->...
        public function andX($x = null); // Returns Expr\AndX instance

        // Example - $qb->expr()->orX($cond1 [, $condN])->add(...)->...
        public function orX($x = null); // Returns Expr\OrX instance


        /** Comparison objects **/
        /** 比较对象 **/

        // Example - $qb->expr()->eq('u.id', '?1') => u.id = ?1
        public function eq($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->neq('u.id', '?1') => u.id <> ?1
        public function neq($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->lt('u.id', '?1') => u.id < ?1
        public function lt($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->lte('u.id', '?1') => u.id <= ?1
        public function lte($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->gt('u.id', '?1') => u.id > ?1
        public function gt($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->gte('u.id', '?1') => u.id >= ?1
        public function gte($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->isNull('u.id') => u.id IS NULL
        public function isNull($x); // Returns string

        // Example - $qb->expr()->isNotNull('u.id') => u.id IS NOT NULL
        public function isNotNull($x); // Returns string


        /** Arithmetic objects **/
        /** 算术运算对象 **/

        // Example - $qb->expr()->prod('u.id', '2') => u.id * 2
        public function prod($x, $y); // Returns Expr\Math instance

        // Example - $qb->expr()->diff('u.id', '2') => u.id - 2
        public function diff($x, $y); // Returns Expr\Math instance

        // Example - $qb->expr()->sum('u.id', '2') => u.id + 2
        public function sum($x, $y); // Returns Expr\Math instance

        // Example - $qb->expr()->quot('u.id', '2') => u.id / 2
        public function quot($x, $y); // Returns Expr\Math instance


        /** Pseudo-function objects **/
        /** 伪函数对象 **/

        // Example - $qb->expr()->exists($qb2->getDql())
        public function exists($subquery); // Returns Expr\Func instance

        // Example - $qb->expr()->all($qb2->getDql())
        public function all($subquery); // Returns Expr\Func instance

        // Example - $qb->expr()->some($qb2->getDql())
        public function some($subquery); // Returns Expr\Func instance

        // Example - $qb->expr()->any($qb2->getDql())
        public function any($subquery); // Returns Expr\Func instance

        // Example - $qb->expr()->not($qb->expr()->eq('u.id', '?1'))
        public function not($restriction); // Returns Expr\Func instance

        // Example - $qb->expr()->in('u.id', array(1, 2, 3))
        // Make sure that you do NOT use something similar to $qb->expr()->in('value', array('stringvalue')) as this will cause Doctrine to throw an Exception.
        // 确保你没有使用类似于 $qb->expr()->in('value', array('stringvalue'))，因为这将导致 Doctrine 抛出一个异常。
        // Instead, use $qb->expr()->in('value', array('?1')) and bind your parameter to ?1 (see section above)
        // 使用 $qb->expr()->in('value', array('?1')) 替代并绑定你的参数到 ?1 （请看下面部分）
        public function in($x, $y); // Returns Expr\Func instance

        // Example - $qb->expr()->notIn('u.id', '2')
        public function notIn($x, $y); // Returns Expr\Func instance

        // Example - $qb->expr()->like('u.firstname', $qb->expr()->literal('Gui%'))
        public function like($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->notLike('u.firstname', $qb->expr()->literal('Gui%'))
        public function notLike($x, $y); // Returns Expr\Comparison instance

        // Example - $qb->expr()->between('u.id', '1', '10')
        public function between($val, $x, $y); // Returns Expr\Func


        /** Function objects **/
        /** 函数对象 **/

        // Example - $qb->expr()->trim('u.firstname')
        public function trim($x); // Returns Expr\Func

        // Example - $qb->expr()->concat('u.firstname', $qb->expr()->concat($qb->expr()->literal(' '), 'u.lastname'))
        public function concat($x, $y); // Returns Expr\Func

        // Example - $qb->expr()->substring('u.firstname', 0, 1)
        public function substring($x, $from, $len); // Returns Expr\Func

        // Example - $qb->expr()->lower('u.firstname')
        public function lower($x); // Returns Expr\Func

        // Example - $qb->expr()->upper('u.firstname')
        public function upper($x); // Returns Expr\Func

        // Example - $qb->expr()->length('u.firstname')
        public function length($x); // Returns Expr\Func

        // Example - $qb->expr()->avg('u.age')
        public function avg($x); // Returns Expr\Func

        // Example - $qb->expr()->max('u.age')
        public function max($x); // Returns Expr\Func

        // Example - $qb->expr()->min('u.age')
        public function min($x); // Returns Expr\Func

        // Example - $qb->expr()->abs('u.currentBalance')
        public function abs($x); // Returns Expr\Func

        // Example - $qb->expr()->sqrt('u.currentBalance')
        public function sqrt($x); // Returns Expr\Func

        // Example - $qb->expr()->count('u.firstname')
        public function count($x); // Returns Expr\Func

        // Example - $qb->expr()->countDistinct('u.surname')
        public function countDistinct($x); // Returns Expr\Func
    }

Adding a Criteria to a Query // 添加 Criteria 至查询
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can also add a :ref:`Criteria <filtering-collections>` to a QueryBuilder by
using ``addCriteria``:

你也可以通过使用 ``addCriteria`` 添加一个 :ref:`Criteria <filtering-collections>` 到 QueryBuilder：

.. code-block:: php

    <?php
    use Doctrine\Common\Collections\Criteria;
    // ...

    $criteria = Criteria::create()
        ->orderBy(['firstName', 'ASC']);

    // $qb instanceof QueryBuilder
    $qb->addCriteria($criteria);
    // then execute your query like normal

Low Level API // 低层 API 
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we have describe the low level (thought of as the
hardcore method) of creating queries. It may be useful to work at
this level for optimization purposes, but most of the time it is
preferred to work at a higher level of abstraction.

现在我们描述创建查询的底层（被认为是核心方法）。为了优化目的，在此层运行可能是有帮助的，
但是多数时候推荐在更高层的抽象上运行。

All helper methods in ``QueryBuilder`` actually rely on a single
one: ``add()``. This method is responsible of building every piece
of DQL. It takes 3 parameters: ``$dqlPartName``, ``$dqlPart`` and
``$append`` (default=false)

``QueryBuilder`` 中的所有辅助方法实际上依赖于单一的一个：``add()``。此方法负责构建每一个
DQL。它接受三个参数：``$dqlPartName``、 ``$dqlPart`` 和 ``$append`` (默认为 false)。

-  ``$dqlPartName``: Where the ``$dqlPart`` should be placed.
   Possible values: select, from, where, groupBy, having, orderBy
-  ``$dqlPartName``：其中的 ``$dqlPart`` 应该被替换。可能的值：
   select、from、where、groupBy、having、orderBy。
-  ``$dqlPart``: What should be placed in ``$dqlPartName``. Accepts
   a string or any instance of ``Doctrine\ORM\Query\Expr\*``
-  ``$dqlPart``：应该被替换，同 ``$dqlPartName`` 中一样。接受一个字符串或任何
   ``Doctrine\ORM\Query\Expr\*`` 的实例。
-  ``$append``: Optional flag (default=false) if the ``$dqlPart``
   should override all previously defined items in ``$dqlPartName`` or
   not (no effect on the ``where`` and ``having`` DQL query parts,
   which always override all previously defined items)
-  ``$append``：可选标记（默认为 false），是否 ``$dqlPart`` 应该覆盖之前在 ``$dqlPartName``
   中定义的所有条目（在 ``where`` 和 ``having`` DQL 查询部分上不影响，该部分始终覆盖之前定义的所有条目）。

.. code-block:: php

    <?php
    // $qb instanceof QueryBuilder

    // example6: how to define:
    // "SELECT u FROM User u WHERE u.id = ? ORDER BY u.name ASC"
    // using QueryBuilder string support
    $qb->add('select', 'u')
       ->add('from', 'User u')
       ->add('where', 'u.id = ?1')
       ->add('orderBy', 'u.name ASC');

Expr\* classes // Expr\* 类
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When you call ``add()`` with string, it internally evaluates to an
instance of ``Doctrine\ORM\Query\Expr\Expr\*`` class. Here is the
same query of example 6 written using
``Doctrine\ORM\Query\Expr\Expr\*`` classes:

当你带字符串调用 ``add()``，它内部等价于一个 ``Doctrine\ORM\Query\Expr\Expr\*`` 类的实例。
这里是一个使用 ``Doctrine\ORM\Query\Expr\Expr\*`` 类写的和例子6一样的查询：

.. code-block:: php

   <?php
   // $qb instanceof QueryBuilder

   // example7: how to define:
   // "SELECT u FROM User u WHERE u.id = ? ORDER BY u.name ASC"
   // using QueryBuilder using Expr\* instances
   $qb->add('select', new Expr\Select(array('u')))
      ->add('from', new Expr\From('User', 'u'))
      ->add('where', new Expr\Comparison('u.id', '=', '?1'))
      ->add('orderBy', new Expr\OrderBy('u.name', 'ASC'));

Of course this is the hardest way to build a DQL query in Doctrine.
To simplify some of these efforts, we introduce what we call as
``Expr`` helper class.

当然，在 Doctrine 中这是最难的构建 DQL 查询的方式。为了简化这些工作，
我们引入了我们称之为 “Expr” 辅助类。
