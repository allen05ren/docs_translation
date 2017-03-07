Annotations Reference // 注释参考
======================================

You've probably used docblock annotations in some form already,
most likely to provide documentation metadata for a tool like
``PHPDocumentor`` (@author, @link, ...). Docblock annotations are a
tool to embed metadata inside the documentation section which can
then be processed by some tool. Doctrine 2 generalizes the concept
of docblock annotations so that they can be used for any kind of
metadata and so that it is easy to define new docblock annotations.
In order to allow more involved annotation values and to reduce the
chances of clashes with other docblock annotations, the Doctrine 2
docblock annotations feature an alternative syntax that is heavily
inspired by the Annotation syntax introduced in Java 5.

你很可能已经使用了某种形式的文档块注释，很可能为像 ``PHPDocumentor`` (@author, @link, ...)
这样的工具提供文档元数据。文档块注释是嵌入元数据到文档部分的工具，之后可以通过一些工具处理。
Doctrine 2 概括了文档块注释的概念，所以它容易定义新的文档块注释。为了允许更多涉及注释值和
减少与其他文档块注释冲突的机会，Doctrine 2 文档块注释功能替换语法很大程度上受到了在 Java 5
中引入的注释语法的启发。

The implementation of these enhanced docblock annotations is
located in the ``Doctrine\Common\Annotations`` namespace and
therefore part of the Common package. Doctrine 2 docblock
annotations support namespaces and nested annotations among other
things. The Doctrine 2 ORM defines its own set of docblock
annotations for supplying object-relational mapping metadata.

这些增强的文档块注释的实现被定位在 ``Doctrine\Common\Annotations`` 命名空间，并因此是
Common 包的一部分。Doctrine 2 文档块注释支持命名空间和嵌套注释等等。Doctrine 2 ORM 定义
了它自己的一组文档块注释以提供对象关系映射元数据。

.. note::

    If you're not comfortable with the concept of docblock
    annotations, don't worry, as mentioned earlier Doctrine 2 provides
    XML and YAML alternatives and you could easily implement your own
    favourite mechanism for defining ORM metadata.

    如果你对文档块注释的概念并不感到舒适，没有关系，如前所述 Doctrine 2 提供了更容易的
    XML 和 YAML 替代而且你可以轻易地实现自己喜欢的机制来定义 ORM 元数据。

In this chapter a reference of every Doctrine 2 Annotation is given
with short explanations on their context and usage.

在本章中给出了每一个 Doctrine 2 文档块注释的参考，对它们的上下文和用法作简短的解释。

Index // 目录
-------------------

-  :ref:`@Column <annref_column>`
-  :ref:`@ColumnResult <annref_column_result>`
-  :ref:`@Cache <annref_cache>`
-  :ref:`@ChangeTrackingPolicy <annref_changetrackingpolicy>`
-  :ref:`@CustomIdGenerator <annref_customidgenerator>`
-  :ref:`@DiscriminatorColumn <annref_discriminatorcolumn>`
-  :ref:`@DiscriminatorMap <annref_discriminatormap>`
-  :ref:`@Embeddable <annref_embeddable>`
-  :ref:`@Embedded <annref_embedded>`
-  :ref:`@Entity <annref_entity>`
-  :ref:`@EntityResult <annref_entity_result>`
-  :ref:`@FieldResult <annref_field_result>`
-  :ref:`@GeneratedValue <annref_generatedvalue>`
-  :ref:`@HasLifecycleCallbacks <annref_haslifecyclecallbacks>`
-  :ref:`@Index <annref_index>`
-  :ref:`@Id <annref_id>`
-  :ref:`@InheritanceType <annref_inheritancetype>`
-  :ref:`@JoinColumn <annref_joincolumn>`
-  :ref:`@JoinColumns <annref_joincolumns>`
-  :ref:`@JoinTable <annref_jointable>`
-  :ref:`@ManyToOne <annref_manytoone>`
-  :ref:`@ManyToMany <annref_manytomany>`
-  :ref:`@MappedSuperclass <annref_mappedsuperclass>`
-  :ref:`@NamedNativeQuery <annref_named_native_query>`
-  :ref:`@OneToOne <annref_onetoone>`
-  :ref:`@OneToMany <annref_onetomany>`
-  :ref:`@OrderBy <annref_orderby>`
-  :ref:`@PostLoad <annref_postload>`
-  :ref:`@PostPersist <annref_postpersist>`
-  :ref:`@PostRemove <annref_postremove>`
-  :ref:`@PostUpdate <annref_postupdate>`
-  :ref:`@PrePersist <annref_prepersist>`
-  :ref:`@PreRemove <annref_preremove>`
-  :ref:`@PreUpdate <annref_preupdate>`
-  :ref:`@SequenceGenerator <annref_sequencegenerator>`
-  :ref:`@SqlResultSetMapping <annref_sql_resultset_mapping>`
-  :ref:`@Table <annref_table>`
-  :ref:`@UniqueConstraint <annref_uniqueconstraint>`
-  :ref:`@Version <annref_version>`

Reference // 参考
-----------------------

.. _annref_column:

@Column
~~~~~~~

Marks an annotated instance variable as "persistent". It has to be
inside the instance variables PHP DocBlock comment. Any value hold
inside this variable will be saved to and loaded from the database
as part of the lifecycle of the instance variables entity-class.

标记一个注释的实例变量作为“持久化的”。它必须是在该实例变量内部的 PHP 文档块注解。
任何保留在此变量内部的将被保存到数据库并从数据库加载作为实例变量实体类的生命周期的一部分。

Required attributes:

必须的属性：

-  **type**: Name of the Doctrine Type which is converted between PHP
   and Database representation.
-  **type**：在 PHP 和 数据库表示之间转换的 Doctrine 类型名称。

Optional attributes:

可选的属性：

-  **name**: By default the property name is used for the database
   column name also, however the 'name' attribute allows you to
   determine the column name.
-  **name**：默认属性名也被用于数据库列名，但是“name”属性允许你确定列名。
-  **length**: Used by the "string" type to determine its maximum
   length in the database. Doctrine does not validate the length of a
   string values for you.
-  **length**：用于“string”类型以确定它在数据库中的最大长度。Doctrine 不能为你
   验证字符串的长度。
-  **precision**: The precision for a decimal (exact numeric) column
   (applies only for decimal column), which is the maximum number of
   digits that are stored for the values.
-  **precision**：decimal（精确的数字）列的精度（仅适用于 decimal 列），
   值被存储的最大位数。
-  **scale**: The scale for a decimal (exact numeric) column (applies
   only for decimal column), which represents the number of digits
   to the right of the decimal point and must not be greater than
   *precision*.
-  **scale**：decimal（精确的数字）列的刻度（仅适用于 decimal 列），小数点右边的位数，
   且不能大于 *precision*。
-  **unique**: Boolean value to determine if the value of the column
   should be unique across all rows of the underlying entities table.
-  **unique**：布尔值，用于确定列的值是否应该在底层实例表的所有行中是唯一的。
-  **nullable**: Determines if NULL values allowed for this column.
-  **nullable**：确定是否此列允许 NULL 值。
-  **options**: Array of additional options:
-  **options**：其他的选项数组：

   -  ``default``: The default value to set for the column if no value
      is supplied.

   -  ``default``：如果没有值提供，为此列设置该默认值。
   -  ``unsigned``: Boolean value to determine if the column should
      be capable of representing only non-negative integers
      (applies only for integer column and might not be supported by
      all vendors).
   -  ``unsigned``：布尔值，用于确定是否此列应该仅能代表非负整数（仅适用于整型列且
      可能不被所有提供商所支持）
   -  ``fixed``: Boolean value to determine if the specified length of
      a string column should be fixed or varying (applies only for
      string/binary column and might not be supported by all vendors).
   -  ``fixed``：布尔值，用于确定是否字符串列的指定长度应该是固定的还是变化的（仅适用
      于字符串/二进制列并且可能不被所有的提供商所支持）。
   -  ``comment``: The comment of the column in the schema (might not
      be supported by all vendors).
   -  ``comment``：在数据库（schema）中此列的注释（comment）（可能不被所有提供商所支持）。
   -  ``collation``: The collation of the column (only supported by Drizzle, Mysql, PostgreSQL>=9.1, Sqlite and SQLServer).
   -  ``collation``： 此列的排序规则（仅 Drizzle、Mysql、PostgreSQL>=9.1、Sqlite 和 SQLServer支持）。

-  **columnDefinition**: DDL SQL snippet that starts after the column
   name and specifies the complete (non-portable!) column definition.
   This attribute allows to make use of advanced RMDBS features.
   However you should make careful use of this feature and the
   consequences. SchemaTool will not detect changes on the column correctly
   anymore if you use "columnDefinition".
-  **columnDefinition**：在此列名之后起始并指定完整的（非便携的）列定义的 DDL SQL 片断。
   此属性允许使用高级的 RMDBS 特性。但是你应该小心地使用该特性和后果。如果你使用“columnDefinition”
   SchemaTool 将不再正确地侦测在此列的变更。

   Additionally you should remember that the "type"
   attribute still handles the conversion between PHP and Database
   values. If you use this attribute on a column that is used for
   joins between tables you should also take a look at
   :ref:`@JoinColumn <annref_joincolumn>`.

   另外，你应该记住“type”属性仍然处理 PHP 和数据库值之间的转换。如果你在一个列上使用此属性，
   且该列被用于联结，你也应当看一下 :ref:`@JoinColumn <annref_joincolumn>`。

.. note::

    For more detailed information on each attribute, please refer to
    the DBAL ``Schema-Representation`` documentation.

    在每个属性上的更多详细信息，请参考 DBAL ``Schema-Representation`` 文档。

Examples:

示例：

.. code-block:: php

    <?php
    /**
     * @Column(type="string", length=32, unique=true, nullable=false)
     */
    protected $username;

    /**
     * @Column(type="string", columnDefinition="CHAR(2) NOT NULL")
     */
    protected $country;

    /**
     * @Column(type="decimal", precision=2, scale=1)
     */
    protected $height;

    /**
     * @Column(type="string", length=2, options={"fixed":true, "comment":"Initial letters of first and last name"})
     */
    protected $initials;

    /**
     * @Column(type="integer", name="login_count" nullable=false, options={"unsigned":true, "default":0})
     */
    protected $loginCount;

.. _annref_column_result:

@ColumnResult
~~~~~~~~~~~~~~
References name of a column in the SELECT clause of a SQL query.
Scalar result types can be included in the query result by specifying this annotation in the metadata.

引用在SQL 查询的 SELECT 子句中列的名。标量结果类型可以通过在元数据中指定此注释来包括在查询结果中。

Required attributes:

必须的属性：

-  **name**: The name of a column in the SELECT clause of a SQL query
-  **name**：在SQL 查询的 SELECT 子句中列的名字。

.. _annref_cache:

@Cache
~~~~~~~~~~~~~~
Add caching strategy to a root entity or a collection.

添加缓存策略到根实体或集合。

Optional attributes:

可选的属性：

-  **usage**: One of ``READ_ONLY``, ``READ_WRITE`` or ``NONSTRICT_READ_WRITE``, By default this is ``READ_ONLY``.
-  **usage**：``READ_ONLY``、``READ_WRITE``或``NONSTRICT_READ_WRITE`` 之一，默认为 ``READ_ONLY``。
-  **region**: An specific region name
-  **region**： 一个特定的区域（region）名。

.. _annref_changetrackingpolicy:

@ChangeTrackingPolicy
~~~~~~~~~~~~~~~~~~~~~

The Change Tracking Policy annotation allows to specify how the
Doctrine 2 UnitOfWork should detect changes in properties of
entities during flush. By default each entity is checked according
to a deferred implicit strategy, which means upon flush UnitOfWork
compares all the properties of an entity to a previously stored
snapshot. This works out of the box, however you might want to
tweak the flush performance where using another change tracking
policy is an interesting option.

变更跟踪策略注释允许指定 Doctrine 2 UnitOfWork 在 flush 期间应该如何侦测在实体的属性中的变更。
默认每个实体根据隐式延迟策略检查，这意味着在刷新时 UnitOfWork 将实体上所有的属性与之前存储的快照进行
比较。这是开箱即用的，但是你可能希望调整 flush 的性能，使用另一个变更跟踪策略是一个有趣的选项。

The :doc:`details on all the available change tracking policies <change-tracking-policies>`
can be found in the configuration section.

:doc:`所有可用的变更跟踪策略详情 <change-tracking-policies>` 可以在配置部分中找到。

Example:

.. code-block:: php

    <?php
    /**
     * @Entity
     * @ChangeTrackingPolicy("DEFERRED_IMPLICIT")
     * @ChangeTrackingPolicy("DEFERRED_EXPLICIT")
     * @ChangeTrackingPolicy("NOTIFY")
     */
    class User {}

.. _annref_customidgenerator:

@CustomIdGenerator
~~~~~~~~~~~~~~~~~~~~~

This annotations allows you to specify a user-provided class to generate identifiers. This annotation only works when both :ref:`@Id <annref_id>` and :ref:`@GeneratedValue(strategy="CUSTOM") <annref_generatedvalue>` are specified.

此注释允许你指定用户提供的类来生成标识符。此注释仅当:ref:`@Id <annref_id>` 和 :ref:`@GeneratedValue(strategy="CUSTOM") <annref_generatedvalue>`
被指定时才可使用。

Required attributes:

必须的属性：

-  **class**: name of the class which should extend Doctrine\ORM\Id\AbstractIdGenerator
-  **class**:应当是扩展了 Doctrine\ORM\Id\AbstractIdGenerator 的类的名字。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Id 
     * @Column(type="integer")
     * @GeneratedValue(strategy="CUSTOM")
     * @CustomIdGenerator(class="My\Namespace\MyIdGenerator")
     */
    public $id;

.. _annref_discriminatorcolumn:

@DiscriminatorColumn
~~~~~~~~~~~~~~~~~~~~~

This annotation is an optional annotation for the topmost/super
class of an inheritance hierarchy. It specifies the details of the
column which saves the name of the class, which the entity is
actually instantiated as.

此注释是一个可选的注释，用于继承层次结构的最顶端/超级类。它指定了保存类名称的列的详情，
该类名称指示哪一实体是实际上要实例化的。

If this annotation is not specified, the discriminator column defaults
to a string column of length 255 called ``dtype``.

如果此注释没有被指定，鉴别器列默认为长度为 255 称为 ``dtype`` 的字符串列。

Required attributes:

必须的属性：

-  **name**: The column name of the discriminator. This name is also
   used during Array hydration as key to specify the class-name.
-  **name**：鉴别器列的名称。此名称仅被用在数组水合时作为键所指定的类名。

Optional attributes:

可选的属性：

-  **type**: By default this is string.
-  **type**：默认为 string。
-  **length**: By default this is 255.
-  **length**：默认为 255。

.. _annref_discriminatormap:

@DiscriminatorMap
~~~~~~~~~~~~~~~~~~~~~

The discriminator map is a required annotation on the
topmost/super class in an inheritance hierarchy. Its only argument is an
array which defines which class should be saved under
which name in the database. Keys are the database value and values
are the classes, either as fully- or as unqualified class names
depending on whether the classes are in the namespace or not.

鉴别器映射是一个在继承层次结构中最顶端/超级类上必须的注释。它仅有一个数组参数，该数组定义
哪个类应该保存在数据库中的哪个名称。键是数据库的值，值是类的完全或非限定类名称，这取决于是否
类是在命名空间中还是不在。

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


.. _annref_embeddable:

@Embeddable
~~~~~~~~~~~~~~~~~~~~~

The embeddable is required on an entity targeted to be embeddable inside
another. It works together with the :ref:`@Embedded <annref_embedded>`
annotation to establish the relationship between two entities.

在一个实体目标至可嵌入另一个实体内部时需要 @Embeddable 注释。它与 :ref:`@Embedded <annref_embedded>`
注释一起工作，以建立两个实体之间的关联。

译注：希望被嵌入其他类中的类指定此注释。

.. code-block:: php

    <?php

    /**
     * @Embeddable
     */
    class Address
    {
    // ...
    class User
    {
        /**
         * @Embedded(class = "Address")
         */
        private $address;


.. _annref_embedded:

@Embedded
~~~~~~~~~~~~~~~~~~~~~

The embedded annotation is required on a member class varible targeted to
embed it's class argument inside it's own class.

在一个类成员变量目标至嵌入它的 class 参数在自身类内部时需要 @Embedded 注释。

译注：希望嵌入其他类作为类成员的类指定此注释。

Required attributes:

必须的参数：

-  **class**: The embeddable class


.. code-block:: php

    <?php

    // ...
    class User
    {
        /**
         * @Embedded(class = "Address")
         */
        private $address;

    /**
     * @Embeddable
     */
    class Address
    {
    // ...


.. _annref_entity:

@Entity
~~~~~~~

Required annotation to mark a PHP class as an entity. Doctrine manages
the persistence of all classes marked as entities.

标记一个 PHP 类作为一个实体必须的注释。Doctrine 管理所有已标记作为实体的类的持久化。

Optional attributes:

可选的参数：

-  **repositoryClass**: Specifies the FQCN of a subclass of the
   EntityRepository. Use of repositories for entities is encouraged to keep
   specialized DQL and SQL operations separated from the Model/Domain
   Layer.
-  **repositoryClass**：指定 EntityRepository 的子类的 FQCN。鼓励实体使用仓库来保持
   特定的 DQL 和 SQL 操作与模型/领域的分离。
-  **readOnly**: (>= 2.1) Specifies that this entity is marked as read only and not
   considered for change-tracking. Entities of this type can be persisted
   and removed though.
-  **readOnly**：（>= 2.1）指定此实体被标记为只读且不考虑变更跟踪。尽管此类型的实体可以被持久化和移除。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Entity(repositoryClass="MyProject\UserRepository")
     */
    class User
    {
        //...
    }

.. _annref_entity_result:

@EntityResult
~~~~~~~~~~~~~~
References an entity in the SELECT clause of a SQL query.
If this annotation is used, the SQL statement should select all of the columns that are mapped to the entity object.
This should include foreign key columns to related entities.
The results obtained when insufficient data is available are undefined.

引用在 SQL 查询的 SELECT 子句中的实体。如果此注释被使用，SQL 语句应该选择被映射到实体对象的所有列。
这应该包括到相关实体的外键列。当可用数据不足时此结果是未被定义的。

Required attributes:

必须的属性：

-  **entityClass**: The class of the result.
-  **entityClass**：该类的结果。

Optional attributes:

可选的属性：

-  **fields**: Array of @FieldResult, Maps the columns specified in the SELECT list of the query to the properties or fields of the entity class.
-  **fields**：数组的 @FieldResult，映射在查询的 SELECT 列表中指定的列到实体类的属性或字段。
-  **discriminatorColumn**: Specifies the column name of the column in the SELECT list that is used to determine the type of the entity instance.
-  **discriminatorColumn**：指定在 SELECT 列表中的被用于确定实体实例类型的列的列名。

.. _annref_field_result:

@FieldResult
~~~~~~~~~~~~~
Is used to map the columns specified in the SELECT list of the query to the properties or fields of the entity class.

被用于映射在查询的 SELECT 列表中指定的列到实体类属性或字段。

Required attributes:

必须的属性：

-  **name**: Name of the persistent field or property of the class.
-  **name**：类的持久化的字段或属性的名称。

Optional attributes:

可选的属性：

-  **column**: Name of the column in the SELECT clause.
-  **column**：在 SELECT 子句中的列名称。

.. _annref_generatedvalue:

@GeneratedValue
~~~~~~~~~~~~~~~~~~~~~

Specifies which strategy is used for identifier generation for an
instance variable which is annotated by :ref:`@Id <annref_id>`. This
annotation is optional and only has meaning when used in
conjunction with @Id.

为由 :ref:`@Id <annref_id>` 注释的实例变量指定被用于标识符生成的策略。此注释是
可选的并且仅当与 @Id 结合使用时才有意义。

If this annotation is not specified with @Id the NONE strategy is
used as default.

如果未使用 @Id 指定此注释，NONE 策略将被用作默认值。

Optional attributes:

可选的属性：

-  **strategy**: Set the name of the identifier generation strategy.
   Valid values are AUTO, SEQUENCE, TABLE, IDENTITY, UUID, CUSTOM and NONE.
   If not specified, default value is AUTO.
-  **strategy**：设置标识符生成策略的名字。有效值为 AUTO、SEQUENCE、TABLE、IDENTITY、
   UUID、CUSTOM 和 NONE。未指定时，默认值为 AUTO。

Example:

示例

.. code-block:: php

    <?php
    /**
     * @Id
     * @Column(type="integer")
     * @GeneratedValue(strategy="IDENTITY")
     */
    protected $id = null;

.. _annref_haslifecyclecallbacks:

@HasLifecycleCallbacks
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Annotation which has to be set on the entity-class PHP DocBlock to
notify Doctrine that this entity has entity lifecycle callback
annotations set on at least one of its methods. Using @PostLoad,
@PrePersist, @PostPersist, @PreRemove, @PostRemove, @PreUpdate or
@PostUpdate without this marker annotation will make Doctrine
ignore the callbacks.

此注释必须被设置在实体类 PHP 文档块以通知 Doctrine 该实体在至少它的方法中的一个上
拥有实体生命周期回调注释设置。不带此标记注释使用 @PostLoad、@PrePersist、@PostPersist、
@PreRemove、@PostRemove、@PreUpdate 或 @PostUpdate 将使 Doctrine 忽略
其回调。


Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @HasLifecycleCallbacks
     */
    class User
    {
        /**
         * @PostPersist
         */
        public function sendOptinMail() {}
    }

.. _annref_index:

@Index
~~~~~~~

Annotation is used inside the :ref:`@Table <annref_table>` annotation on
the entity-class level. It provides a hint to the SchemaTool to
generate a database index on the specified table columns. It only
has meaning in the SchemaTool schema generation context.

此注释被用于在实体类级别上的 :ref:`@Table <annref_table>` 注释内部。它提供一个暗示使
SchemaTool 在指定的表列上生成数据库索引。它仅在 SchemaTool 数据库生成上下文中有意义。

Required attributes:

必须的属性：

-  **name**: Name of the Index
-  **name**：索引的名称。
-  **columns**: Array of columns.
-  **columns**：列的数组。

Optional attributes:

可选的属性：

-  **options**: Array of platform specific options:
-  **options**：平台特定的选项数组：

   -  ``where``: SQL WHERE condition to be used for partial indexes. It will
      only have effect on supported platforms.

   -  ``where``：SQL WHERE 条件被用于部分索引。它将仅在支持的平台上具有影响。

Basic example:

基础的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",indexes={@Index(name="search_idx", columns={"name", "email"})})
     */
    class ECommerceProduct
    {
    }

Example with partial indexes:

带部分索引的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",indexes={@Index(name="search_idx", columns={"name", "email"}, options={"where": "(((id IS NOT NULL) AND (name IS NULL)) AND (email IS NULL))"})})
     */
    class ECommerceProduct
    {
    }

.. _annref_id:

@Id
~~~~~~~

The annotated instance variable will be marked as entity
identifier, the primary key in the database. This annotation is a
marker only and has no required or optional attributes. For
entities that have multiple identifier columns each column has to
be marked with @Id.

注释的示例变量将被标记为实体的标识符、数据库中的主键。此注释仅是一个标记器（marker）且
没有必须的或可选的属性。对于拥有多标识符列的实体的每个列必须使用 @Id 标记。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Id
     * @Column(type="integer")
     */
    protected $id = null;

.. _annref_inheritancetype:

@InheritanceType
~~~~~~~~~~~~~~~~~~~~~

In an inheritance hierarchy you have to use this annotation on the
topmost/super class to define which strategy should be used for
inheritance. Currently Single Table and Class Table Inheritance are
supported.

在一个继承层次结构中你必须使用此注释在最顶端/超级类上定义哪一个策略应该被用于继承。
当前支持单一表继承和类表继承。

This annotation has always been used in conjunction with the
:ref:`@DiscriminatorMap <annref_discriminatormap>` and
:ref:`@DiscriminatorColumn <annref_discriminatorcolumn>` annotations.

此注释始终与 :ref:`@DiscriminatorMap <annref_discriminatormap>` 和
:ref:`@DiscriminatorColumn <annref_discriminatorcolumn>` 注释结合使用。

Examples:

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @InheritanceType("SINGLE_TABLE")
     * @DiscriminatorColumn(name="discr", type="string")
     * @DiscriminatorMap({"person" = "Person", "employee" = "Employee"})
     */
    class Person
    {
        // ...
    }

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

.. _annref_joincolumn:

@JoinColumn
~~~~~~~~~~~~~~

This annotation is used in the context of relations in
:ref:`@ManyToOne <annref_manytoone>`, :ref:`@OneToOne <annref_onetoone>` fields
and in the Context of :ref:`@JoinTable <annref_jointable>` nested inside
a @ManyToMany. This annotation is not required. If it is not
specified the attributes *name* and *referencedColumnName* are
inferred from the table and primary key names.

此注释被用在 :ref:`@ManyToOne <annref_manytoone>`、:ref:`@OneToOne <annref_onetoone>` 字段中关联
的上下文中或在嵌套在一个 @ManyToMany 中的 :ref:`@JoinTable <annref_jointable>` 的上下文中。
此注释不是必须的。如果它未被指定，则从表和主键名推断属性 *name* 和 *referencedColumnName*。

Required attributes:

必须的属性：

-  **name**: Column name that holds the foreign key identifier for
   this relation. In the context of @JoinTable it specifies the column
   name in the join table.
-  **name**：保存此关联的外键标识符的列名称。在 @JoinTable 上下文中它指定在联结表中的列名称。
-  **referencedColumnName**: Name of the primary key identifier that
   is used for joining of this relation.
-  **referencedColumnName**：被用于联结此关联的主键标识符的名称。

Optional attributes:

可选的属性：

-  **unique**: Determines whether this relation is exclusive between the
   affected entities and should be enforced as such on the database
   constraint level. Defaults to false.
-  **unique**：确定是否此关联在受影响的实体之间是排外的且同样地在数据库约束级别上应该被强制。
   默认为 false。
-  **nullable**: Determine whether the related entity is required, or if
   null is an allowed state for the relation. Defaults to true.
-  **nullable**：确定是否该关联实体是必须的，或如果允许关联的状态为 null。默认为 true。
-  **onDelete**: Cascade Action (Database-level)
-  **onDelete**：级联操作（数据库级别）
-  **columnDefinition**: DDL SQL snippet that starts after the column
   name and specifies the complete (non-portable!) column definition.
   This attribute enables the use of advanced RMDBS features. Using
   this attribute on @JoinColumn is necessary if you need slightly
   different column definitions for joining columns, for example
   regarding NULL/NOT NULL defaults. However by default a
   "columnDefinition" attribute on :ref:`@Column <annref_column>` also sets
   the related @JoinColumn's columnDefinition. This is necessary to
   make foreign keys work.
-  **columnDefinition**：在此列名之后起始并指定完整的（非便携的）列定义的 DDL SQL 片断。
   此属性使能够使用高级的 RMDBS 特性。如果你需要为联结列定义稍微不同的列，在 @JoinColumn 上
   使用此属性是必须的，例如关于 NULL/NOT NULL 默认值。然而，默认 "columnDefinition"
   属性也在 :ref:`@Column <annref_column>` 上设置相关的 @JoinColumn 的 columnDefinition。
   这是使外键工作所必需的。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @OneToOne(targetEntity="Customer")
     * @JoinColumn(name="customer_id", referencedColumnName="id")
     */
    private $customer;

.. _annref_joincolumns:

@JoinColumns
~~~~~~~~~~~~~~

An array of @JoinColumn annotations for a
:ref:`@ManyToOne <annref_manytoone>` or :ref:`@OneToOne <annref_onetoone>`
relation with an entity that has multiple identifiers.

@JoinColumn 注释的一个数组，用于为一个拥有多标识符的实体与 :ref:`@ManyToOne <annref_manytoone>`
或 :ref:`@OneToOne <annref_onetoone>` 关联。

.. _annref_jointable:

@JoinTable
~~~~~~~~~~~~~~

Using :ref:`@OneToMany <annref_onetomany>` or
:ref:`@ManyToMany <annref_manytomany>` on the owning side of the relation
requires to specify the @JoinTable annotation which describes the
details of the database join table. If you do not specify
@JoinTable on these relations reasonable mapping defaults apply
using the affected table and the column names.

在关联的 owning 侧使用 :ref:`@OneToMany <annref_onetomany>` 或
:ref:`@ManyToMany <annref_manytomany>` 需要指定 @JoinTable 注释来
描述数据库联结表的详情。如果你不在这些关联上指定 @JoinTable 将应用合理的
受影响的表和列名映射默认值。

Optional attributes:

可选的属性：

-  **name**: Database name of the join-table
-  **name**：联结表的数据库名称。
-  **joinColumns**: An array of @JoinColumn annotations describing the
   join-relation between the owning entities table and the join table.
-  **joinColumns**：@JoinColumn 注释的数组，描述 owning 实体表和联结表之间的联结关系。
-  **inverseJoinColumns**: An array of @JoinColumn annotations
   describing the join-relation between the inverse entities table and
   the join table.
-  **inverseJoinColumns**：@JoinColumn 注释的数组，描述 inverse 实体表和联结表之间
   的联结关系。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @ManyToMany(targetEntity="Phonenumber")
     * @JoinTable(name="users_phonenumbers",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="phonenumber_id", referencedColumnName="id", unique=true)}
     * )
     */
    public $phonenumbers;

.. _annref_manytoone:

@ManyToOne
~~~~~~~~~~~~~~

Defines that the annotated instance variable holds a reference that
describes a many-to-one relationship between two entities.

定义注释的实例变量保存的一个引用，该引用描述了一个两个实体之间的 many-to-one 关联。

Required attributes:

必须的属性：

-  **targetEntity**: FQCN of the referenced target entity. Can be the
   unqualified class name if both classes are in the same namespace.
   *IMPORTANT:* No leading backslash!
-  **targetEntity**：引用的目标实体的FQCN。可以是非限定类名，如果两个类在同一命名空间中的话。
   *重要*：没有前导反斜杠！

Optional attributes:

可选的属性：

-  **cascade**: Cascade Option
-  **cascade**：级联选项
-  **fetch**: One of LAZY or EAGER
-  **fetch**：LAZY 或 EAGER 之一
-  inversedBy - The inversedBy attribute designates the field in
   the entity that is the inverse side of the relationship.
-  inversedBy - inversedBy 属性指派在关联的 inverse 侧的实体中的字段。

Example:

实例：

.. code-block:: php

    <?php
    /**
     * @ManyToOne(targetEntity="Cart", cascade={"all"}, fetch="EAGER")
     */
    private $cart;

.. _annref_manytomany:

@ManyToMany
~~~~~~~~~~~~~~

Defines that the annotated instance variable holds a many-to-many relationship
between two entities. :ref:`@JoinTable <annref_jointable>` is an
additional, optional annotation that has reasonable default
configuration values using the table and names of the two related
entities.

定义注释的实例变量保存的两个实体之间的 many-to-many 关联。:ref:`@JoinTable <annref_jointable>`
是一个额外的、可选的注释，拥有合理的默认值，使用表和两个相关的实体的名字配置值。

Required attributes:

必须的属性：

-  **targetEntity**: FQCN of the referenced target entity. Can be the
   unqualified class name if both classes are in the same namespace.
   *IMPORTANT:* No leading backslash!
-  **targetEntity**:引用的目标实体的 FQCN。可以是非限定类名，如果两个类在同一命名空间中的话。
   *重要*：没有前导反斜杠！

Optional attributes:

可选的属性：

-  **mappedBy**: This option specifies the property name on the
   targetEntity that is the owning side of this relation. It is a
   required attribute for the inverse side of a relationship.
-  **mappedBy**：此选项指定在此关联的 owning 侧的 targetEntity 上的属性名称。
   对于关联的 inverse 侧它是必须的属性。
-  **inversedBy**: The inversedBy attribute designates the field in the
   entity that is the inverse side of the relationship.
-  **inversedBy**：inversedBy 属性指派在关联的 inverse 侧的实体中的字段。
-  **cascade**: Cascade Option
-  **cascade**：级联选项
-  **fetch**: One of LAZY, EXTRA_LAZY or EAGER
-  **fetch**：LAZY、EXTRA_LAZY 或 EAGER 之一
-  **indexBy**: Index the collection by a field on the target entity.
-  **indexBy**：通过目标实体上的一个字段索引的集合。

.. note::

    For ManyToMany bidirectional relationships either side may
    be the owning side (the side that defines the @JoinTable and/or
    does not make use of the mappedBy attribute, thus using a default
    join table).

    对于 ManyToMany 的双向的关联任何一侧都可能是 owning 侧（定义 @JoinTable 的一侧
    和/或不能使用 mappedBy 属性的一侧，因此使用一个默认的联结表）。

Example:

例如：

.. code-block:: php

    <?php
    /**
     * Owning Side
     *
     * @ManyToMany(targetEntity="Group", inversedBy="features")
     * @JoinTable(name="user_groups",
     *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
     *      inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
     *      )
     */
    private $groups;

    /**
     * Inverse Side
     *
     * @ManyToMany(targetEntity="User", mappedBy="groups")
     */
    private $features;

.. _annref_mappedsuperclass:

@MappedSuperclass
~~~~~~~~~~~~~~~~~~~~~

A mapped superclass is an abstract or concrete class that provides
persistent entity state and mapping information for its subclasses,
but which is not itself an entity. This annotation is specified on
the Class docblock and has no additional attributes.

映射超类是一个抽象的或具体的类，为它的子类提供了持久化的实体的状态和映射信息，但自身不是
一个实例。此注释在类文档块上指定并没有额外的属性。

The @MappedSuperclass annotation cannot be used in conjunction with
@Entity. See the Inheritance Mapping section for
:doc:`more details on the restrictions of mapped superclasses <inheritance-mapping>`.

@MappedSuperclass 注释不能与 @Entity 组合使用。查看继承映射部分以 :doc:`了解更多映射超类的局限性 <inheritance-mapping>`

Optional attributes:

可选的属性：

-  **repositoryClass**: (>= 2.2) Specifies the FQCN of a subclass of the EntityRepository.
   That will be inherited for all subclasses of that Mapped Superclass.
-  **repositoryClass**：（>= 2.2）指定 EntityRepository 的子类的 FQCN。它将被该映射超类的所有子集继承。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @MappedSuperclass
     */
    class MappedSuperclassBase
    {
        // ... fields and methods
    }

    /**
     * @Entity
     */
    class EntitySubClassFoo extends MappedSuperclassBase
    {
        // ... fields and methods
    }

.. _annref_named_native_query:

@NamedNativeQuery
~~~~~~~~~~~~~~~~~
Is used to specify a native SQL named query.
The NamedNativeQuery annotation can be applied to an entity or mapped superclass.

被用于指定一个原生的 SQL 命名的查询。

Required attributes:

必须的属性：

-  **name**: The name used to refer to the query with the EntityManager methods that create query objects.
-  **name**：用于使用创建查询对象的 EntityManager 方法引用的查询的名称。
-  **query**: The SQL query string.
-  **query**：SQL 查询字符串。

Optional attributes:

可选的属性：

-  **resultClass**: The class of the result.
-  **resultClass**： 结果的类。
-  **resultSetMapping**: The name of a SqlResultSetMapping, as defined in metadata.
-  **resultSetMapping**：SqlResultSetMapping 的名称，如元数据中定义的。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @NamedNativeQueries({
     *      @NamedNativeQuery(
     *          name            = "fetchJoinedAddress",
     *          resultSetMapping= "mappingJoinedAddress",
     *          query           = "SELECT u.id, u.name, u.status, a.id AS a_id, a.country, a.zip, a.city FROM cms_users u INNER JOIN cms_addresses a ON u.id = a.user_id WHERE u.username = ?"
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
     *                      @FieldResult(name = "address.zip"),
     *                      @FieldResult(name = "address.city"),
     *                      @FieldResult(name = "address.country"),
     *                      @FieldResult(name = "address.id", column = "a_id"),
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
.. _annref_onetoone:

@OneToOne
~~~~~~~~~~~~~~

The @OneToOne annotation works almost exactly as the
:ref:`@ManyToOne <annref_manytoone>` with one additional option which can
be specified. The configuration defaults for
:ref:`@JoinColumn <annref_joincolumn>` using the target entity table and
primary key column names apply here too.

@OneToOne 注释的工作几乎完全与 :ref:`@ManyToOne <annref_manytoone>` 一样，带有一个可以
被指定的额外选项。使用目标实体表和主键列名为 :ref:`@JoinColumn <annref_joincolumn>` 配置默认
值也适用于此处。

Required attributes:

必须的属性：

-  **targetEntity**: FQCN of the referenced target entity. Can be the
   unqualified class name if both classes are in the same namespace.
   *IMPORTANT:* No leading backslash!
-  **targetEntity**：引用的目标实体的 FQCN。可以是非限定类名，如果两个类在同一命名空间中的话。
   *重要*：没有前导反斜杠！

Optional attributes:

可选的属性：

-  **cascade**: Cascade Option
-  **cascade**：级联选项
-  **fetch**: One of LAZY or EAGER
-  **fetch**：LAZY 或 EAGER 之一
-  **orphanRemoval**: Boolean that specifies if orphans, inverse
   OneToOne entities that are not connected to any owning instance,
   should be removed by Doctrine. Defaults to false.
-  **orphanRemoval**：布尔值，指定如果 orphans、inverse 的 OneToOne 实体不与任何
   owning 实例连接，应该由 Doctrine 移除。默认为 false。
-  **inversedBy**: The inversedBy attribute designates the field in the
   entity that is the inverse side of the relationship.
-  **inversedBy**：inversedBy 属性指派在关联的 inverse 侧的实体的字段。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @OneToOne(targetEntity="Customer")
     * @JoinColumn(name="customer_id", referencedColumnName="id")
     */
    private $customer;

.. _annref_onetomany:

@OneToMany
~~~~~~~~~~~~~~

Required attributes:

必须的属性：

-  **targetEntity**: FQCN of the referenced target entity. Can be the
   unqualified class name if both classes are in the same namespace.
   *IMPORTANT:* No leading backslash!
-  **targetEntity**：引用的目标实体的 FQCN。可以是非限定类名，如果两个类在同一命名空间中的话。
   *重要*：没有前导反斜杠！

Optional attributes:

可选的属性：

-  **cascade**: Cascade Option
-  **cascade**：级联选项
-  **orphanRemoval**: Boolean that specifies if orphans, inverse
   OneToOne entities that are not connected to any owning instance,
   should be removed by Doctrine. Defaults to false.
-  **orphanRemoval**：布尔值，指定如果 orphans、inverse 的 OneToOne 实体不与任何
   owning 实例连接，应该由 Doctrine 移除。默认为 false。
-  **mappedBy**: This option specifies the property name on the
   targetEntity that is the owning side of this relation. Its a
   required attribute for the inverse side of a relationship.
-  **mappedBy**：此选项指定在此关联的 owning 侧的 targetEntity 上的属性名称。
   对于关联的 inverse 侧它是必须的属性。
-  **fetch**: One of LAZY, EXTRA_LAZY or EAGER.
-  **fetch**：LAZY、EXTRA_LAZY 或 EAGER 之一。
-  **indexBy**: Index the collection by a field on the target entity.
-  **indexBy**：通过目标实体上的一个字段索引的集合。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @OneToMany(targetEntity="Phonenumber", mappedBy="user", cascade={"persist", "remove", "merge"}, orphanRemoval=true)
     */
    public $phonenumbers;

.. _annref_orderby:

@OrderBy
~~~~~~~~~~~~~~

Optional annotation that can be specified with a
:ref:`@ManyToMany <annref_manytomany>` or :ref:`@OneToMany <annref_onetomany>`
annotation to specify by which criteria the collection should be
retrieved from the database by using an ORDER BY clause.

可以使用 :ref:`@ManyToMany <annref_manytomany>` 或 :ref:`@OneToMany <annref_onetomany>`
指定的可选的注释，以指定通过哪一 criteria 使用 ORDER BY 子句应该从数据库取回的集合。

This annotation requires a single non-attributed value with an DQL
snippet:

此注释需要使用 DQL 片断的单一的未归因（non-attributed）值：

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @ManyToMany(targetEntity="Group")
     * @OrderBy({"name" = "ASC"})
     */
    private $groups;

The DQL Snippet in OrderBy is only allowed to consist of
unqualified, unquoted field names and of an optional ASC/DESC
positional statement. Multiple Fields are separated by a comma (,).
The referenced field names have to exist on the ``targetEntity``
class of the ``@ManyToMany`` or ``@OneToMany`` annotation.

在 OrderBy 中的 DQL 片断仅允许由非限定、unquoted 的字段名和可选的 ASC/DESC
位置语句组成。多字段由一个英文逗号（,）分隔。引用的字段名必须存在于 ``targetEntity``
类的 ``@ManyToMany`` 或 ``@OneToMany`` 注释上。

.. _annref_postload:

@PostLoad
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PostLoad event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PostLoad 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_postpersist:

@PostPersist
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PostPersist event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PostPersist 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_postremove:

@PostRemove
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PostRemove event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PostRemove 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_postupdate:

@PostUpdate
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PostUpdate event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PostUpdate 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_prepersist:

@PrePersist
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PrePersist event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PrePersist 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_preremove:

@PreRemove
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PreRemove event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PreRemove 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_preupdate:

@PreUpdate
~~~~~~~~~~~~~~

Marks a method on the entity to be called as a @PreUpdate event.
Only works with @HasLifecycleCallbacks in the entity class PHP
DocBlock.

在实体上标记因 @PreUpdate 事件而被调用的方法。仅在实体类的 PHP 文档块中与
@HasLifecycleCallbacks 一起使用。

.. _annref_sequencegenerator:

@SequenceGenerator
~~~~~~~~~~~~~~~~~~~~~

For use with @GeneratedValue(strategy="SEQUENCE") this
annotation allows to specify details about the sequence, such as
the increment size and initial values of the sequence.

为了与 @GeneratedValue(strategy="SEQUENCE") 一起使用，此注释允许指定关于序列的细节，
比如序列的递增大小和初始值。

Required attributes:

必须的属性：

-  **sequenceName**: Name of the sequence
-  **sequenceName**：系列的名称

Optional attributes:

可选的属性：

-  **allocationSize**: Increment the sequence by the allocation size
   when its fetched. A value larger than 1 allows optimization for
   scenarios where you create more than one new entity per request.
   Defaults to 10
-  **allocationSize**：当被取回时，序列通过分配的大小递增。大于1的值允许对每个请求
   创建超过1个新实体的情况进行优化。默认值为10。
-  **initialValue**: Where the sequence starts, defaults to 1.
-  **initialValue**：序列的起始值，默认为 1。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Id
     * @GeneratedValue(strategy="SEQUENCE")
     * @Column(type="integer")
     * @SequenceGenerator(sequenceName="tablename_seq", initialValue=1, allocationSize=100)
     */
    protected $id = null;

.. _annref_sql_resultset_mapping:

@SqlResultSetMapping
~~~~~~~~~~~~~~~~~~~~
The SqlResultSetMapping annotation is used to specify the mapping of the result of a native SQL query.
The SqlResultSetMapping annotation can be applied to an entity or mapped superclass.

SqlResultSetMapping 注释被用于指定原生 SQL 查询的结果的映射。
SqlResultSetMapping 注释可以被应用于一个实体或映射超类。

Required attributes:

必须的属性：

-  **name**: The name given to the result set mapping, and used to refer to it in the methods of the Query API.
-  **name**：给予结果集映射的名称，并用于在查询 API 的方法中引用它。

Optional attributes:

可选的属性：

-  **entities**: Array of @EntityResult, Specifies the result set mapping to entities.
-  **entities**：@EntityResult 的数组，指定结果集映射到实体。
-  **columns**: Array of @ColumnResult, Specifies the result set mapping to scalar values.
-  **columns**：@ColumnResult 的数组，指定结果集映射到标量值。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @NamedNativeQueries({
     *      @NamedNativeQuery(
     *          name            = "fetchUserPhonenumberCount",
     *          resultSetMapping= "mappingUserPhonenumberCount",
     *          query           = "SELECT id, name, status, COUNT(phonenumber) AS numphones FROM cms_users INNER JOIN cms_phonenumbers ON id = user_id WHERE username IN (?) GROUP BY id, name, status, username ORDER BY username"
     *      ),
     *      @NamedNativeQuery(
     *          name            = "fetchMultipleJoinsEntityResults",
     *          resultSetMapping= "mappingMultipleJoinsEntityResults",
     *          query           = "SELECT u.id AS u_id, u.name AS u_name, u.status AS u_status, a.id AS a_id, a.zip AS a_zip, a.country AS a_country, COUNT(p.phonenumber) AS numphones FROM cms_users u INNER JOIN cms_addresses a ON u.id = a.user_id INNER JOIN cms_phonenumbers p ON u.id = p.user_id GROUP BY u.id, u.name, u.status, u.username, a.id, a.zip, a.country ORDER BY u.username"
     *      ),
     * })
     * @SqlResultSetMappings({
     *      @SqlResultSetMapping(
     *          name    = "mappingUserPhonenumberCount",
     *          entities= {
     *              @EntityResult(
     *                  entityClass = "User",
     *                  fields      = {
     *                      @FieldResult(name = "id"),
     *                      @FieldResult(name = "name"),
     *                      @FieldResult(name = "status"),
     *                  }
     *              )
     *          },
     *          columns = {
     *              @ColumnResult("numphones")
     *          }
     *      ),
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
.. _annref_table:

@Table
~~~~~~~

Annotation describes the table an entity is persisted in. It is
placed on the entity-class PHP DocBlock and is optional. If it is
not specified the table name will default to the entity's
unqualified classname.

此注释描述实体被持久化的表。它被放置在实体类的 PHP 文档块上且是可选的。如果它没有被指定，
表名将默认为该实体的非限定类名。

Required attributes:

必须的属性:

-  **name**: Name of the table
-  **name**:表的名称。

Optional attributes:

可选的属性：

-  **indexes**: Array of @Index annotations
-  **indexes**：@Index 注释的数组。
-  **uniqueConstraints**: Array of @UniqueConstraint annotations.
-  **uniqueConstraints**：@UniqueConstraint注释的数组。
-  **schema**: (>= 2.5) Name of the schema the table lies in.
-  **schema**：（>= 2.5）表所在的数据库（schema）名称。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="user",
     *      uniqueConstraints={@UniqueConstraint(name="user_unique",columns={"username"})},
     *      indexes={@Index(name="user_idx", columns={"email"})}
     *      schema="schema_name"
     * )
     */
    class User { }

.. _annref_uniqueconstraint:

@UniqueConstraint
~~~~~~~~~~~~~~~~~~~~~

Annotation is used inside the :ref:`@Table <annref_table>` annotation on
the entity-class level. It allows to hint the SchemaTool to
generate a database unique constraint on the specified table
columns. It only has meaning in the SchemaTool schema generation
context.

此注释在实体类层级上被用在 :ref:`@Table <annref_table>` 注释内部。它允许暗示
SchemaTool 在指定的表列上生成数据库唯一约束。它仅在 SchemaTool 数据库生成上下文中有意义。

Required attributes:

必须的属性：

-  **name**: Name of the Index
-  **name**：索引的名称。
-  **columns**: Array of columns.
-  **columns**：列的数组。

Optional attributes:

可选的属性：

-  **options**: Array of platform specific options:
-  **options**：平台特定的选项数组：

   -  ``where``: SQL WHERE condition to be used for partial indexes. It will
      only have effect on supported platforms.

  -  ``where``：SQL WHERE 条件被用于部分索引。它将仅在支持的平台上具有影响。

Basic example:

基础的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",uniqueConstraints={@UniqueConstraint(name="search_idx", columns={"name", "email"})})
     */
    class ECommerceProduct
    {
    }

Example with partial indexes:

带部分索引的示例：

.. code-block:: php

    <?php
    /**
     * @Entity
     * @Table(name="ecommerce_products",uniqueConstraints={@UniqueConstraint(name="search_idx", columns={"name", "email"}, options={"where": "(((id IS NOT NULL) AND (name IS NULL)) AND (email IS NULL))"})})
     */
    class ECommerceProduct
    {
    }

.. _annref_version:

@Version
~~~~~~~~

Marker annotation that defines a specified column as version attribute used in
an :ref:`optimistic locking <transactions-and-concurrency_optimistic-locking>`
scenario. It only works on :ref:`@Column <annref_column>` annotations that have
the type ``integer`` or ``datetime``. Combining ``@Version`` with
:ref:`@Id <annref_id>` is not supported.

此标记器注释定义一个指定的列作为版本属性用在一个 :ref:`乐观锁 <transactions-and-concurrency_optimistic-locking>`
方案中。它仅工作在拥有 ``integer`` 或 ``datetime``类型的 :ref:`@Column <annref_column>` 注释上。
使用 :ref:`@Id <annref_id>` 组合 ``@Version`` 是不被支持的。

Example:

示例：

.. code-block:: php

    <?php
    /**
     * @Column(type="integer")
     * @Version
     */
    protected $version;

