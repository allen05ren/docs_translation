Best Practices // 最佳实践
================================

The best practices mentioned here that affect database
design generally refer to best practices when working with Doctrine
and do not necessarily reflect best practices for database design
in general.

这里提到的影响数据库设计的最佳实践一般是指使用 Doctrine 时的最佳实践，
并且一般来说，不一定反映数据库设计的最佳实践。

Constrain relationships as much as possible // 尽可能约束关联
-----------------------------------------------------------------

It is important to constrain relationships as much as possible.

尽可能约束关联是重要的。

This means:

这意味着：

-  Impose a traversal direction (avoid bidirectional associations
   if possible)
-  推行一个遍历方向（避免双向的关联，如果可能的话）
-  Eliminate nonessential associations
-  消除不重要的关联

This has several benefits:

这有几个优势：

-  Reduced coupling in your domain model
-  在你的领域模型中减少耦合
-  Simpler code in your domain model (no need to maintain
   bidirectionality properly)
-  在你的领域模型中更简洁的代码（不需要维护完全双向性）
-  Less work for Doctrine
-  减少 Doctrine 的工作

Avoid composite keys // 避免复合键
---------------------------------------

Even though Doctrine fully supports composite keys it is best not
to use them if possible. Composite keys require additional work by
Doctrine and thus have a higher probability of errors.

即使 Doctrine 完全地支持复合键，如果可能最好不要使用它们。复合键要求 Doctrine 额外的工作
且因此拥有更高概率的错误。

Use events judiciously // 得当地使用事件
-------------------------------------------

The event system of Doctrine is great and fast. Even though making
heavy use of events, especially lifecycle events, can have a
negative impact on the performance of your application. Thus you
should use events judiciously.

Doctrine 的事件系统是很好且快速的。即使大量使用事件，尤其生命周期事件，在应用程序的性能上
会有负面的影响。因此你应该得当地使用事件。

Use cascades judiciously // 得当地使用级联
---------------------------------------------

Automatic cascades of the persist/remove/merge/etc. operations are
very handy but should be used wisely. Do NOT simply add all
cascades to all associations. Think about which cascades actually
do make sense for you for a particular association, given the
scenarios it is most likely used in.

persist/remove/merge 等等的自动级联操作是非常方便的，但是应该聪明地使用。不要简单
地添加所有级联到所有关联。思考哪些级联对于你对于特定的关联实际上是有意义的，考虑到它
最可能的使用情况。

Don't use special characters // 不使用特殊字符
-------------------------------------------------

Avoid using any non-ASCII characters in class, field, table or
column names. Doctrine itself is not unicode-safe in many places
and will not be until PHP itself is fully unicode-aware.

在类、字段、表或列名中避免使用任何非 ASCII 字符。Doctrine 自身在很多地方不是
unicode-safe 的且将不使用，直到 PHP 自身是完全地 unicode-aware。

Don't use identifier quoting // 不使用标识符 quoting
--------------------------------------------------------

Identifier quoting is a workaround for using reserved words that
often causes problems in edge cases. Do not use identifier quoting
and avoid using reserved words as table or column names.

标识符 quoting 是一个为使用保留字的变通方法，在边缘情况中经常导致问题。不要使用
标识符 quoting 且避免使用保留字作为表或列的名称。

Initialize collections in the constructor // 在构造器中初始化集合
-------------------------------------------------------------------

It is recommended best practice to initialize any business
collections in entities in the constructor. Example:

在构造器中初始化实体中的任何业务集合是推荐的最佳实践。示例：

.. code-block:: php

    <?php
    namespace MyProject\Model;
    use Doctrine\Common\Collections\ArrayCollection;
    
    class User {
        private $addresses;
        private $articles;
    
        public function __construct() {
            $this->addresses = new ArrayCollection;
            $this->articles = new ArrayCollection;
        }
    }

Don't map foreign keys to fields in an entity // 在实体中不要映射外键到字段
----------------------------------------------------------------------------

Foreign keys have no meaning whatsoever in an object model. Foreign
keys are how a relational database establishes relationships. Your
object model establishes relationships through object references.
Thus mapping foreign keys to object fields heavily leaks details of
the relational model into the object model, something you really
should not do.

外键在对象模型中没有任何意义。外键是关系数据如何建立关系。你的对象模型通过对象引用建立关联。
因此映射外键到对象字段严重泄漏关系模型的细节到对象模型中，这是真正不应该做的事情。

Use explicit transaction demarcation // 使用显式事务界限
-----------------------------------------------------------

While Doctrine will automatically wrap all DML operations in a
transaction on flush(), it is considered best practice to
explicitly set the transaction boundaries yourself. Otherwise every
single query is wrapped in a small transaction (Yes, SELECT
queries, too) since you can not talk to your database outside of a
transaction. While such short transactions for read-only (SELECT)
queries generally don't have any noticeable performance impact, it
is still preferable to use fewer, well-defined transactions that
are established through explicit transaction boundaries.

虽然 Doctrine 将自动地在 flush() 上封装所有的 DML 操作在一个事务中，自己显式地设置
事务边界被认为是最佳实践。否则，每一个单一查询被封装在一个小的事务（是的，SELECT 请求也是）中
因为你不能在一个事务之外与你的数据库通讯。虽然这种为只读（SELECT）查询的短事务通常没有任何显著的
性能影响，更少使用，并通过显式事务边界建立的好的定义的事务仍然是更可取的。
