Basic Mapping // 基础映射
==============================

This guide explains the basic mapping of entities and properties.
After working through this guide you should know:

本指南解释实体和属性的基础映射。
通过本指南的学习你应当知道：

- How to create PHP objects that can be saved to the database with Doctrine;
- 如何创建可以使用 Doctrine 保存至数据库的 PHP 对象；
- How to configure the mapping between columns on tables and properties on
  entities;
- 如何在表上的列和实体上的属性之间配置映射；
- What Doctrine mapping types are;
- Doctrine 有哪些映射类型；
- Defining primary keys and how identifiers are generated by Doctrine;
- 定义主键和如何由 Doctrine 生成标识；
- How quoting of reserved symbols works in Doctrine.
- 在 Doctrine 中如何引用被保留符号。

Mapping of associations will be covered in the next chapter on
:doc:`Association Mapping <association-mapping>`.

关联的映射将在下一章节 :doc:`关联映射 <association-mapping>` 中被覆盖。

Guide Assumptions // 本指南假设
------------------------------------

You should have already :doc:`installed and configure <configuration>`
Doctrine.

你应该已经 :doc:`安装和配置 <configuration>` Doctrine。

Creating Classes for the Database // 为数据库创建类
-------------------------------------------------------

Every PHP object that you want to save in the database using Doctrine
is called an "Entity". The term "Entity" describes objects
that have an identity over many independent requests. This identity is
usually achieved by assigning a unique identifier to an entity.
In this tutorial the following ``Message`` PHP class will serve as the
example Entity:

每个你想保存在数据库中的 PHP 对象，在 Doctrine 中被称为一个“实体”。术语“实体”描述的对象
有一个遍及许多独立请求的身份。这个身份通常由分配一个唯一标识符至一个实体而获得的。
在本教程中，下面的 ``Message`` PHP 类将充当实体的例子：

.. code-block:: php

    <?php
    class Message
    {
        private $id;
        private $text;
        private $postedAt;
    }

Because Doctrine is a generic library, it only knows about your
entities because you will describe their existence and structure using
mapping metadata, which is configuration that tells Doctrine how your
entity should be stored in the database. The documentation will often
speak of "mapping something", which means writing the mapping metadata
that describes your entity.

因为 Doctrine 是一个通用库，它仅理解你的实体，因为你将使用映射元数据（mapping metadata）
描述他们的存在和结构，映射元数据是告诉 Doctrine 你的实体应该如何被存储在数据库中的配置。
本文档将经常提及”映射什么（mapping something）“，这意味着正在编写此映射元数据以描述你的实体。

Doctrine provides several different ways to specify object-relational
mapping metadata:

Doctrine 提供了几个不同的方式指定对象关系映射元数据（object-relational
mapping metadata）：

-  :doc:`文档块注释（Docblock Annotations） <annotations-reference>`
-  :doc:`XML <xml-mapping>`
-  :doc:`YAML <yaml-mapping>`
-  :doc:`PHP code <php-mapping>`

This manual will usually show mapping metadata via docblock annotations, though
many examples also show the equivalent configuration in YAML and XML.

本手册将经常通过文档块注释来展示映射元数据，尽管一些例子也展示等价的 YAML 和 XML 配置。

.. note::

    All metadata drivers perform equally. Once the metadata of a class has been
    read from the source (annotations, xml or yaml) it is stored in an instance
    of the ``Doctrine\ORM\Mapping\ClassMetadata`` class and these instances are
    stored in the metadata cache.  If you're not using a metadata cache (not
    recommended!) then the XML driver is the fastest.

    所有的元数据驱动表现一样。每当一个类的元数据从源（注释，xml或yaml）被读取，
    它会被存储在一个 ``Doctrine\ORM\Mapping\ClassMetadata`` 类的实例中，并且这些实例被存储在元数据缓存中。
    如果你不使用元数据缓存（并不推荐）那么xml驱动是最快的。

Marking our ``Message`` class as an entity for Doctrine is straightforward:

标记我们的 ``Message`` 类作为一个 Doctrine 实体是很容易的：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Message
        {
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
              <!-- ... -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          # ...

With no additional information, Doctrine expects the entity to be saved
into a table with the same name as the class in our case ``Message``.
You can change this by configuring information about the table:

没有额外信息，Doctrine 期望该实体被保存进同名的表，如在我们例子中的 ``Message`` 类。
你可以通过有关此表的配置信息修改它：

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * @Entity
         * @Table(name="message")
         */
        class Message
        {
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message" table="message">
              <!-- ... -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          table: message
          # ...

Now the class ``Message`` will be saved and fetched from the table ``message``.

现在 ``Message`` 类将被保存且取回从表 ``Message``。

Property Mapping // 属性映射
---------------------------------

The next step after marking a PHP class as an entity is mapping its properties
to columns in a table.

让一个 PHP 类作为实体之后，下一步是映射它的属性到表中的列。

To configure a property use the ``@Column`` docblock annotation. The ``type``
attribute specifies the :ref:`Doctrine Mapping Type <reference-mapping-types>`
to use for the field. If the type is not specified, ``string`` is used as the
default.

使用 ``@Column`` 文档块注释配置一个属性。``type`` 属性（attribute）指定应用于该字段的
:ref:`Doctrine 映射类型（Mapping Type） <reference-mapping-types>`。如果类型没有被指定，默认使用
``string``。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Message
        {
            /** @Column(type="integer") */
            private $id;
            /** @Column(length=140) */
            private $text;
            /** @Column(type="datetime", name="posted_at") */
            private $postedAt;
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <field name="id" type="integer" />
            <field name="text" length="140" />
            <field name="postedAt" column="posted_at" type="datetime" />
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          fields:
            id:
              type: integer
            text:
              length: 140
            postedAt:
              type: datetime
              column: posted_at

When we don't explicitly specify a column name via the ``name`` option, Doctrine
assumes the field name is also the column name. This means that:

当我们没有明确地通过 ``name`` 选项指定一个列的名称，Doctrine 假定该字段名称即是该列的名称。
这意味着：

* the ``id`` property will map to the column ``id`` using the type ``integer``;
* ``id`` 属性将映射至列 ``id``，使用 ``integer`` 类型;
* the ``text`` property will map to the column ``text`` with the default mapping type ``string``;
* ``text`` 属性将映射至列 ``text``，默认 ``string`` 类型；
* the ``postedAt`` property will map to the ``posted_at`` column with the ``datetime`` type.
* ``postedAt`` 属性将映射至列 ``posted_at``，默认 ``datetime`` 类型。

The Column annotation has some more attributes. Here is a complete
list:
Column 注释有许多属性（attributes）。这里是一个完整的列表：

- ``type``: (optional, defaults to 'string') The mapping type to
  use for the column.
- ``type``: （可选项，默认 ’string‘ ）该列使用的映射类型。
- ``name``: (optional, defaults to field name) The name of the
  column in the database.
- ``name``: （可选项，默认字段名）数据库中该列的名称。
- ``length``: (optional, default 255) The length of the column in
  the database. (Applies only if a string-valued column is used).
- ``length``: （可选项，默认255）数据库中该列的长度。（仅被用于字符串（string）类型值）。
- ``unique``: (optional, default FALSE) Whether the column is a
  unique key.
- ``unique``: （可选项，默认FALSE）是否该列是唯一键（unique key）。
- ``nullable``: (optional, default FALSE) Whether the database
  column is nullable.
- ``nullable``: （可选项，默认FALSE）是否该数据库列可空。
- ``precision``: (optional, default 0) The precision for a decimal
  (exact numeric) column (applies only for decimal column),
  which is the maximum number of digits that are stored for the values.
- ``precision``: （可选项，默认0） decimal （精确的数字）列（仅用于 decimal 列类型），
  它是一个值的最大存储位数。
- ``scale``: (optional, default 0) The scale for a decimal (exact
  numeric) column (applies only for decimal column), which represents
  the number of digits to the right of the decimal point and must
  not be greater than *precision*.
- ``scale``: （可选项，默认0） decimal 的刻度（精确的数字）列（仅用于 decimal 列类型），
  它代表小数点右边的位数并且不能超过 *precision*。
- ``columnDefinition``: (optional) Allows to define a custom
  DDL snippet that is used to create the column. Warning: This normally
  confuses the SchemaTool to always detect the column as changed.
- ``columnDefinition``: （可选项）允许自定义被用于创建列的 DDL 片段。
  警告：这通常混淆 SchemaTool 总是检测列的改变。
- ``options``: (optional) Key-value pairs of options that get passed
  to the underlying database platform when generating DDL statements.
- ``options``: （可选项）生成 DDL 语句时传递至底层数据库平台的键值对选项。

.. _reference-mapping-types:

Doctrine Mapping Types // Doctrine 映射类型
------------------------------------------------

The ``type`` option used in the ``@Column`` accepts any of the existing
Doctrine types or even your own custom types. A Doctrine type defines
the conversion between PHP and SQL types, independent from the database vendor
you are using. All Mapping Types that ship with Doctrine are fully portable
between the supported database systems.

用于 ``@Column`` 中的 ``type`` 选项接受任何现有的 Doctrine 类型或甚至自定义类型。
一个 Doctrine 类型定义了在 PHP 和 SQL 类型之间的转换，独立于你所使用的数据库提供商。
Doctrine 附带的所有的映射类型在受支持的数据库系统之间是完全可移植的。

As an example, the Doctrine Mapping Type ``string`` defines the
mapping from a PHP string to a SQL VARCHAR (or VARCHAR2 etc.
depending on the RDBMS brand). Here is a quick overview of the
built-in mapping types:

举个例子，Doctrine 映射类型 ``string`` 定义了从 PHP 字符串到 SQL VARCHAR （或 VARCHAR2 等。取决于 RDBMS 品牌）的映射。
这里有一个内建映射类型的快速预览：

-  ``string``: Type that maps a SQL VARCHAR to a PHP string.
-  ``string``: SQL VARCHAR 类型至 PHP 字符串的映射。
-  ``integer``: Type that maps a SQL INT to a PHP integer.
-  ``integer``: SQL INT 类型至 PHP 整型数的映射。
-  ``smallint``: Type that maps a database SMALLINT to a PHP
   integer.
-  ``smallint``: 数据库 SMALLINT 类型至 PHP 整型数的映射。
-  ``bigint``: Type that maps a database BIGINT to a PHP string.
-  ``bigint``: 数据库 GIGINT 类型至 PHP 字符串的映射。
-  ``boolean``: Type that maps a SQL boolean or equivalent (TINYINT) to a PHP boolean.
-  ``boolean``: SQL boolean 或 等价（TINYINT）类型至 PHP boolean 的映射。
-  ``decimal``: Type that maps a SQL DECIMAL to a PHP string.
-  ``decimal``: SQL DECIMAL 类型至PHP 字符串映射。
-  ``date``: Type that maps a SQL DATETIME to a PHP DateTime
   object.
-  ``date``: SQL DATETIME 类型至 PHP DateTime 对象的映射。
-  ``time``: Type that maps a SQL TIME to a PHP DateTime object.
-  ``time``: SQL TIME 类型至 PHP DateTime 对象的映射。
-  ``datetime``: Type that maps a SQL DATETIME/TIMESTAMP to a PHP
   DateTime object.
-  ``datetime``: SQL DATETIME/TIMESTAMP 类型至 PHP DateTime 对象的映射。
-  ``datetimetz``: Type that maps a SQL DATETIME/TIMESTAMP to a PHP
   DateTime object with timezone.
-  ``datetimetz``: SQL DATETIME/TIMESTAMP 类型至带时区的 PHP DateTime 对象的映射。
-  ``text``: Type that maps a SQL CLOB to a PHP string.
-  ``text``: SQL CLOB 类型至 PHP 字符串的映射。
-  ``object``: Type that maps a SQL CLOB to a PHP object using
   ``serialize()`` and ``unserialize()``
-  ``object``: SQL CLOB 类型至 PHP 对象的映射，使用 ``serialize()`` 和 ``unserialize()``。
-  ``array``: Type that maps a SQL CLOB to a PHP array using
   ``serialize()`` and ``unserialize()``
-  ``array``: SQL CLOB 类型至 PHP 数组的映射，使用 ``serialize()`` 和 ``unserialize()``。
-  ``simple_array``: Type that maps a SQL CLOB to a PHP array using
   ``implode()`` and ``explode()``, with a comma as delimiter. *IMPORTANT*
   Only use this type if you are sure that your values cannot contain a ",".
-  ``simple_array``: SQL CLOB 类型至 PHP 数组的映射，使用 ``implode()`` 和 ``explode()``，用英文逗号（“,”）作为分割。
   *重要*：仅在你确认你的数据不包含英文逗号（“,”)时使用该类型。
-  ``json_array``: Type that maps a SQL CLOB to a PHP array using
   ``json_encode()`` and ``json_decode()``
-  ``json_array``: SQL CLOB 类型至 PHP 数组的映射，使用 ``json_encode()`` and ``json_decode()``。
-  ``float``: Type that maps a SQL Float (Double Precision) to a
   PHP double. *IMPORTANT*: Works only with locale settings that use
   decimal points as separator.
-  ``float``: SQL Float （双精度）类型至 PHP double 的映射。*重要*：
   仅在本地化设置使用小数点作为分割符时才能正常工作。
-  ``guid``: Type that maps a database GUID/UUID to a PHP string. Defaults to
   varchar but uses a specific type if the platform supports it.
-  ``guid``: 数据库 GUID/UUID 类型至 PHP 字符串的映射。默认 varchar 类型，但是如果平台支持则使用特定的类型。
-  ``blob``: Type that maps a SQL BLOB to a PHP resource stream
-  ``blob``: SQL BLOB 类型至 PHP 资源流（resource stream）的映射。

A cookbook article shows how to define :doc:`your own custom mapping types
<../cookbook/custom-mapping-types>`.

一篇 cookbook 的文章展示了如何定义 :doc:`你自己的映射类型 <../cookbook/custom-mapping-types>`。

.. note::

    DateTime and Object types are compared by reference, not by value. Doctrine
    updates this values if the reference changes and therefore behaves as if
    these objects are immutable value objects.

    DateTime 和 Object 类型是通过引用被比较，而不是通过值。
    如果引用改变了 Doctrine 会更新相应的值，也因此表现的犹如这些对象是不可变对象。

.. warning::

    All Date types assume that you are exclusively using the default timezone
    set by `date_default_timezone_set() <http://docs.php.net/manual/en/function.date-default-timezone-set.php>`_
    or by the php.ini configuration ``date.timezone``. Working with
    different timezones will cause troubles and unexpected behavior.

    所有的日期类型都假定你仅使用默认的时区设置，通过 `date_default_timezone_set() <http://docs.php.net/manual/en/function.date-default-timezone-set.php>`_ 或 php.ini 配置的。
    使用不同的时区将导致问题和非预期的行为。

    If you need specific timezone handling you have to handle this
    in your domain, converting all the values back and forth from UTC.
    There is also a :doc:`cookbook entry <../cookbook/working-with-datetime>`
    on working with datetimes that gives hints for implementing
    multi timezone applications.

    如果你需要指定时区，在你的域中不得不这样做的话，请从 UTC 来回转换所有的值。
    这也有一篇 cookbook 文章 :doc:`使用 datetime <../cookbook/working-with-datetime>` 给出了实现多时区的应用程序的提示。

Identifiers / Primary Keys // 标识符/主键
---------------------------------------------

Every entity class must have an identifier/primary key. You can select
the field that serves as the identifier with the ``@Id``
annotation.

每个实体类必须有一个标识符/主键。你可以使用 ``@Id`` 注释选择充当该标识符的字段。

.. configuration-block::

    .. code-block:: php

        <?php
        class Message
        {
            /**
             * @Id @Column(type="integer")
             * @GeneratedValue
             */
            private $id;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <id name="id" type="integer">
                <generator strategy="AUTO" />
            </id>
            <!-- -->
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: AUTO
          fields:
            # fields here

In most cases using the automatic generator strategy (``@GeneratedValue``) is
what you want. It defaults to the identifier generation mechanism your current
database vendor prefers: AUTO_INCREMENT with MySQL, SERIAL with PostgreSQL,
Sequences with Oracle and so on.

在多数案例中，使用自动生成器策略（``@GeneratedValue``）是你想要的。你当前数据库供应商提供的默认标识符生成机制为：
MySQL的 AUTO_INCREMENT、PostgreSQL 的 SERIAL 、Oracle 的 Sequences 等等。

Identifier Generation Strategies // 标识符生成策略
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The previous example showed how to use the default identifier
generation strategy without knowing the underlying database with
the AUTO-detection strategy. It is also possible to specify the
identifier generation strategy more explicitly, which allows you to
make use of some additional features.

前面的例子展示了如何使用默认的标识符生成策略不用知道底层的数据库用的自动侦测策略。它也可能指定更明确的标识符生成策略，允许你使用一些额外的特性。

Here is the list of possible generation strategies:

这里有一个可能的生成策略列表：

-  ``AUTO`` (default): Tells Doctrine to pick the strategy that is
   preferred by the used database platform. The preferred strategies
   are IDENTITY for MySQL, SQLite, MsSQL and SQL Anywhere and SEQUENCE
   for Oracle and PostgreSQL. This strategy provides full portability.
- ``AUTO`` （默认）：告诉 Doctrine 选取所使用的数据库平台推荐的策略。
   对于 MySQL、SQLite、MsSQL 和其他 SQL 数据库推荐的策略是 IDENTITY，
   对于 Oracle 和 PostgreSQL 推荐的策略是 SEQUENCE。
   这个策略提供完全的可移植性。
-  ``SEQUENCE``: Tells Doctrine to use a database sequence for ID
   generation. This strategy does currently not provide full
   portability. Sequences are supported by Oracle, PostgreSql and
   SQL Anywhere.
-  ``SEQUENCE``: 告诉 Doctrine 使用数据库 sequence 作为 ID 的生成。
   这个策略当前不提供完全的可移植性。
   Sequences 被 Oracle, PostgreSql 和 其他 SQL 数据库所支持。
-  ``IDENTITY``: Tells Doctrine to use special identity columns in
   the database that generate a value on insertion of a row. This
   strategy does currently not provide full portability and is
   supported by the following platforms: MySQL/SQLite/SQL Anywhere
   (AUTO\_INCREMENT), MSSQL (IDENTITY) and PostgreSQL (SERIAL).
-  ``IDENTITY``: 告诉 Doctrine 使用数据库中专门的 identity 列，插入行时生成的值。
   这个策略当前不提供完全可移植性，被以下平台所支持：MySQL/SQLite/SQL 数据库
   (AUTO\_INCREMENT)、 MSSQL (IDENTITY) 和 PostgreSQL (SERIAL)。
-  ``UUID``: Tells Doctrine to use the built-in Universally Unique Identifier
   generator. This strategy provides full portability.
-  ``UUID``: 告诉 Doctrine 使用内建的通用唯一标识符生成器。这个策略提供完全的可移植性。
-  ``TABLE``: Tells Doctrine to use a separate table for ID
   generation. This strategy provides full portability.
   ***This strategy is not yet implemented!***
-  ``TABLE``: 告诉 Doctrine 使用一个单独的表作为 ID 生成。这个策略提供完全的可移植性。
   ***此策略还未被实现！***
-  ``NONE``: Tells Doctrine that the identifiers are assigned (and
   thus generated) by your code. The assignment must take place before
   a new entity is passed to ``EntityManager#persist``. NONE is the
   same as leaving off the @GeneratedValue entirely.
-  ``NONE``: 告诉 Doctrine 标识符由你的代码分配（从而生成的）。
   该分配必须在新的实体被传递给 ``EntityManager#persist`` 以前发生。NONE 同样完全地弃用了 @GeneratedValue。
-  ``CUSTOM``: With this option, you can use the ``@CustomIdGenerator`` annotation.
   It will allow you to pass a :doc:`class of your own to generate the identifiers.<_annref_customidgenerator>`
-  ``CUSTOM``: 有了该选项，你可以使用 ``@CustomIdGenerator`` 注释。
   它将允许你传递一个 :doc:`你自己的用于生成标识符的类 <_annref_customidgenerator>`。

Sequence Generator // Sequence 生成器
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The Sequence Generator can currently be used in conjunction with
Oracle or Postgres and allows some additional configuration options
besides specifying the sequence's name:

Sequence 生成器当前可以被用在与 Oracle 或 Postgres 连接（conjunction）中且除了 sequence 的名称之外还允许一些额外配置选项：

.. configuration-block::

    .. code-block:: php

        <?php
        class Message
        {
            /**
             * @Id
             * @GeneratedValue(strategy="SEQUENCE")
             * @SequenceGenerator(sequenceName="message_seq", initialValue=1, allocationSize=100)
             */
            protected $id = null;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
          <entity name="Message">
            <id name="id" type="integer">
                <generator strategy="SEQUENCE" />
                <sequence-generator sequence-name="message_seq" allocation-size="100" initial-value="1" />
            </id>
          </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Message:
          type: entity
          id:
            id:
              type: integer
              generator:
                strategy: SEQUENCE
              sequenceGenerator:
                sequenceName: message_seq
                allocationSize: 100
                initialValue: 1

The initial value specifies at which value the sequence should
start.

初始值指定了 sequence 生成器应该开始的值。

The allocationSize is a powerful feature to optimize INSERT
performance of Doctrine. The allocationSize specifies by how much
values the sequence is incremented whenever the next value is
retrieved. If this is larger than 1 (one) Doctrine can generate
identifier values for the allocationSizes amount of entities. In
the above example with ``allocationSize=100`` Doctrine 2 would only
need to access the sequence once to generate the identifiers for
100 new entities.

allocationSize 是一个强大的特性，用于优化 Doctrine 的 INSERT 性能。
allocationSize 指定由每当下一个值被取回时 sequences 生成器被递增多少数值。
如果这个值大于1，Doctrine 能够为 allocationSizes 数量的实体生成标识符。
上面的例子中使用 ``allocationSize=100``，Doctrine 2仅需要访问该 sequences 生成器一次
为100个新实体生成标识符。

*The default allocationSize for a @SequenceGenerator is currently 10.*

*@SequenceGenerator 当前的默认 allocationSize 值是 10。*

.. caution::

    The allocationSize is detected by SchemaTool and
    transformed into an "INCREMENT BY " clause in the CREATE SEQUENCE
    statement. For a database schema created manually (and not
    SchemaTool) you have to make sure that the allocationSize
    configuration option is never larger than the actual sequences
    INCREMENT BY value, otherwise you may get duplicate keys.

    在CREATE SEQUENCE 语句中，allocationSize 通过 SchemaTool 被侦测并转变成一个 "INCREMENT BY " 子句。
    对于手动创建一个数据库 schema（而不是 SchemaTool），你必须确保 allocationSize 配置选项永不大于
    实际的 sequences INCREMENT BY 值，否则你可能得到重复的键。


.. note::

    It is possible to use strategy="AUTO" and at the same time
    specifying a @SequenceGenerator. In such a case, your custom
    sequence settings are used in the case where the preferred strategy
    of the underlying platform is SEQUENCE, such as for Oracle and
    PostgreSQL.

    使用 strategy="AUTO" 并同时指定 @SequenceGenerator 是可能的。
    在这种情况下，当底层平台推荐的策略是 SEQUENCE 时，你自定义 sequence 设置将被使用，比如 Oracle 和 PostgreSQL。

Composite Keys // 复合键
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

With Doctrine 2 you can use composite primary keys, using ``@Id`` on more then
one column. Some restrictions exist opposed to using a single identifier in
this case: The use of the ``@GeneratedValue`` annotation is not supported,
which means you can only use composite keys if you generate the primary key
values yourself before calling ``EntityManager#persist()`` on the entity.

使用 Doctrine 2你可以使用复合主键，通过在超过一个列上使用 ``@Id``。相对于使用单个标识符，存在一些限制，比如：
使用``@GeneratedValue`` 注释不被支持，这意味着你如果在实体上调用 ``EntityManager#persist()`` 之前自己生成主键值，你仅能使用复合键。

More details on composite primary keys are discussed in a :doc:`dedicated tutorial
<../tutorials/composite-primary-keys>`.

复合主键的更多详情将在 :doc:`专门的教程 <../tutorials/composite-primary-keys>` 中被讨论。

Quoting Reserved Words // Quoting 被保留字
-----------------------------------------------

Sometimes it is necessary to quote a column or table name because of reserved
word conflicts. Doctrine does not quote identifiers automatically, because it
leads to more problems than it would solve. Quoting tables and column names
needs to be done explicitly using ticks in the definition.

有时需要 Quote 一个列或表名称由于被保留字冲突。Doctrine 不能自动地 Quote 标识符，因为它会导致很多问题超过它解决的问题。
Quoting 表或列名称需要在定义中明确地使用标记（反引号“`”）完成。

.. code-block:: php

    <?php
    /** @Column(name="`number`", type="integer") */
    private $number;

Doctrine will then quote this column name in all SQL statements
according to the used database platform.

Doctrine 将根据所使用的数据库平台在所有的 SQL 语句中 Quote 该列名。

.. warning::

    Identifier Quoting does not work for join column names or discriminator
    column names unless you are using a custom ``QuoteStrategy``.

    对于 join 列名和 discriminator 列名标识符 Quoting 不能使用，除非你使用自定义 ``QuoteStrategy``。

.. _reference-basic-mapping-custom-mapping-types:

.. versionadded: 2.3

For more control over column quoting the ``Doctrine\ORM\Mapping\QuoteStrategy`` interface
was introduced in 2.3. It is invoked for every column, table, alias and other
SQL names. You can implement the QuoteStrategy and set it by calling
``Doctrine\ORM\Configuration#setQuoteStrategy()``.

对于更多在列之上的 Quoting 控制在2.3中引入了 ``Doctrine\ORM\Mapping\QuoteStrategy`` 接口。
它被所有的列、表、别名和其他 SQL 名称所调用。你能够实现 QuoteStrategy 并通过调用
``Doctrine\ORM\Configuration#setQuoteStrategy()`` 设置它。

.. versionadded: 2.4

The ANSI Quote Strategy was added, which assumes quoting is not necessary for any SQL name.
You can use it with the following code:

ANSI Quote 策略被添加，对于任何 SQL 名称假设 Quoting 是不需要。你可以用下面的代码使用它：

.. code-block:: php

    <?php
    use Doctrine\ORM\Mapping\AnsiQuoteStrategy;

    $configuration->setQuoteStrategy(new AnsiQuoteStrategy());
