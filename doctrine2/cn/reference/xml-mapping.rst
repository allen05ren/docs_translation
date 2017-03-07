XML Mapping // XML 映射
==============================

The XML mapping driver enables you to provide the ORM metadata in
form of XML documents.

XML 映射驱动使你能够以 XML 文档的形式提供 ORM 元数据。

The XML driver is backed by an XML Schema document that describes
the structure of a mapping document. The most recent version of the
XML Schema document is available online at
`http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd <http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd>`_.
In order to point to the latest version of the document of a
particular stable release branch, just append the release number,
i.e.: doctrine-mapping-2.0.xsd The most convenient way to work with
XML mapping files is to use an IDE/editor that can provide
code-completion based on such an XML Schema document. The following
is an outline of a XML mapping document with the proper xmlns/xsi
setup for the latest code in trunk.

XML 驱动由描述映射文档的结构的 XML 模式文档支持。在线可用的最新版本的 XML 模式文档在
`http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd <http://www.doctrine-project.org/schemas/orm/doctrine-mapping.xsd>`_。
为了指向特定稳定版本分支的最新版本的文档，只要追加一个版本号，如：doctrine-mapping-2.0.xsd。
使用 XML 映射文件最方便的方式是使用一个可以提供基于这样一个 XML 模式文档的代码完成（code-completion）
的 IDE/编辑器。下面是一个具有合适 xmlns/xsi 设置的用于主干中的最新代码的 XML 映射文档的概览 。

.. code-block:: xml

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                       https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

        ...

    </doctrine-mapping>

The XML mapping document of a class is loaded on-demand the first
time it is requested and subsequently stored in the metadata cache.
In order to work, this requires certain conventions:

类的 XML 映射文档是第一次被请求时即时加载的，随后被存储在元数据缓存中。为了正常工作，
这需要某些惯例：

-  Each entity/mapped superclass must get its own dedicated XML
   mapping document.
-  每个实体/映射超类必须获得其专门的 XML 映射文档。
-  The name of the mapping document must consist of the fully
   qualified name of the class, where namespace separators are
   replaced by dots (.). For example an Entity with the fully
   qualified class-name "MyProject" would require a mapping file
   "MyProject.Entities.User.dcm.xml" unless the extension is changed.
-  映射文档的名字必须由完全限定的类名组成，其中的命名空间分隔符由点（.）替代。例如，
   具有完全限定的类名的实体“MyProject”，将需要一个映射文件“MyProject.Entities.User.dcm.xml”，
   除非扩展名改变了。
-  All mapping documents should get the extension ".dcm.xml" to
   identify it as a Doctrine mapping file. This is more of a
   convention and you are not forced to do this. You can change the
   file extension easily enough.
-  所有映射文档应该获得扩展名“.dcm.xml”以标识它作为一个 Doctrine 映射文件。这更多的是一个惯例且
   没有强制你要这样做。你可以很容易地改变文件扩展名。

.. code-block:: php

    <?php
    $driver->setFileExtension('.xml');

It is recommended to put all XML mapping documents in a single
folder but you can spread the documents over several folders if you
want to. In order to tell the XmlDriver where to look for your
mapping documents, supply an array of paths as the first argument
of the constructor, like this:

推荐放置所有 XML 映射文档在单个文件夹中，但是如果你希望你可以分散文档到几个文件夹。为了告诉 XmlDriver
在哪里查找你的映射文档，提供一个路径的数组作为构造器的第一个参数，像这样：

.. code-block:: php

    <?php
    $config = new \Doctrine\ORM\Configuration();
    $driver = new \Doctrine\ORM\Mapping\Driver\XmlDriver(array('/path/to/files1', '/path/to/files2'));
    $config->setMetadataDriverImpl($driver);

.. warning::

    Note that Doctrine ORM does not modify any settings for ``libxml``,
    therefore, external XML entities may or may not be enabled or
    configured correctly.
    XML mappings are not XXE/XEE attack vectors since they are not
    related with user input, but it is recommended that you do not
    use external XML entities in your mapping files to avoid running
    into unexpected behaviour.

    注意，Doctrine ORM 不修改 ``libxml`` 的任何设置，因此外部 XML 实体可能/还未启用或正确地配置。
    XML 映射不受 XXE/XEE 攻击向量，因为它们与用户输入不相关，但是推荐你不要在映射文件中使用外部的
    XML 实体以避免注入非期望的行为。
    

Simplified XML Driver // 简单的 XML 驱动
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Symfony project sponsored a driver that simplifies usage of the XML Driver.
The changes between the original driver are:

Symfony 项目贡献以一个驱动，简化 XML 驱动的使用。与原始驱动之间的变更是：

1. File Extension is .orm.xml
1. 文件扩展名是 .orm.xml。
2. Filenames are shortened, "MyProject\Entities\User" will become User.orm.xml
2. 文件名被缩短，“MyProject\Entities\User”将变成 User.orm.xml。
3. You can add a global file and add multiple entities in this file.
3. 你可以添加一个全局文件或在此文件中添加多个实体。

Configuration of this client works a little bit different:

客户端的配置工作有一点点不同：

.. code-block:: php

    <?php
    $namespaces = array(
        '/path/to/files1' => 'MyProject\Entities',
        '/path/to/files2' => 'OtherProject\Entities'
    );
    $driver = new \Doctrine\ORM\Mapping\Driver\SimplifiedXmlDriver($namespaces);
    $driver->setGlobalBasename('global'); // global.orm.xml

Example // 示例
----------------------

As a quick start, here is a small example document that makes use
of several common elements:

作为一个快速开始，这是一个小示例文档，使用了几个常见的元素：

.. code-block:: xml

    // Doctrine.Tests.ORM.Mapping.User.dcm.xml
    <?xml version="1.0" encoding="UTF-8"?>
    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                              http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

        <entity name="Doctrine\Tests\ORM\Mapping\User" table="cms_users">

            <indexes>
                <index name="name_idx" columns="name"/>
                <index columns="user_email"/>
            </indexes>

            <unique-constraints>
                <unique-constraint columns="name,user_email" name="search_idx" />
            </unique-constraints>

            <lifecycle-callbacks>
                <lifecycle-callback type="prePersist" method="doStuffOnPrePersist"/>
                <lifecycle-callback type="prePersist" method="doOtherStuffOnPrePersistToo"/>
                <lifecycle-callback type="postPersist" method="doStuffOnPostPersist"/>
            </lifecycle-callbacks>

            <id name="id" type="integer" column="id">
                <generator strategy="AUTO"/>
                <sequence-generator sequence-name="tablename_seq" allocation-size="100" initial-value="1" />
            </id>

            <field name="name" column="name" type="string" length="50" nullable="true" unique="true" />
            <field name="email" column="user_email" type="string" column-definition="CHAR(32) NOT NULL" />

            <one-to-one field="address" target-entity="Address" inversed-by="user">
                <cascade><cascade-remove /></cascade>
                <join-column name="address_id" referenced-column-name="id" on-delete="CASCADE" on-update="CASCADE"/>
            </one-to-one>

            <one-to-many field="phonenumbers" target-entity="Phonenumber" mapped-by="user">
                <cascade>
                    <cascade-persist/>
                </cascade>
                <order-by>
                    <order-by-field name="number" direction="ASC" />
                </order-by>
            </one-to-many>

            <many-to-many field="groups" target-entity="Group">
                <cascade>
                    <cascade-all/>
                </cascade>
                <join-table name="cms_users_groups">
                    <join-columns>
                        <join-column name="user_id" referenced-column-name="id" nullable="false" unique="false" />
                    </join-columns>
                    <inverse-join-columns>
                        <join-column name="group_id" referenced-column-name="id" column-definition="INT NULL" />
                    </inverse-join-columns>
                </join-table>
            </many-to-many>

        </entity>

    </doctrine-mapping>

Be aware that class-names specified in the XML files should be
fully qualified.

注意，在 XML 文件中指定的类名应该是完全限定的。

XML-Element Reference // XML 元素参考
--------------------------------------------

The XML-Element reference explains all the tags and attributes that
the Doctrine Mapping XSD Schema defines. You should read the
Basic-, Association- and Inheritance Mapping chapters to understand
what each of this definitions means in detail.

XML 元素参考解释 Doctrine 映射 XSD 模式定义的所有的标签和属性。你应该阅读基础映射、关联映射
和继承映射章节以理解每个定义的详细含义。

Defining an Entity // 定义一个实体
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each XML Mapping File contains the definition of one entity,
specified as the ``<entity />`` element as a direct child of the
``<doctrine-mapping />`` element:

每个 XML 映射文件包含一个实体的定义，指定为 ``<entity />`` 元素作为 ``<doctrine-mapping />``
元素的直接子代：

.. code-block:: xml

    <doctrine-mapping>
        <entity name="MyProject\User" table="cms_users" schema="schema_name" repository-class="MyProject\UserRepository">
            <!-- definition here -->
        </entity>
    </doctrine-mapping>

Required attributes:

必须的属性：

-  name - The fully qualified class-name of the entity.
-  name - 实体的完全限定类名。

Optional attributes:

可选的选项：

-  **table** - The Table-Name to be used for this entity. Otherwise the
   Unqualified Class-Name is used by default.
-  **table** - 用于该实体的表名。否则默认为非限定的类名。
-  **repository-class** - The fully qualified class-name of an
   alternative ``Doctrine\ORM\EntityRepository`` implementation to be
   used with this entity.
-  **repository-class** - 与该实体一起使用的替换 ``Doctrine\ORM\EntityRepository``
   实现的完全限定类名。
-  **inheritance-type** - The type of inheritance, defaults to none. A
   more detailed description follows in the
   *Defining Inheritance Mappings* section.
-  **inheritance-type** - 继承的类型，默认为 none。在下面*定义继承映射*章节中有更详细的描述。
-  **read-only** - (>= 2.1) Specifies that this entity is marked as read only and not
   considered for change-tracking. Entities of this type can be persisted
   and removed though.
-  **read-only** - （>= 2.1）指定该实体被标记为只读并不考虑变更的跟踪。尽管此类型的实体可以被持久和移除。
-  **schema** - (>= 2.5) The schema the table lies in, for platforms that support schemas
-  **schema** - （>= 2.5）表所在的 schema，用于支持 schema 的平台。

Defining Fields // 定义字段
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each entity class can contain zero to infinite fields that are
managed by Doctrine. You can define them using the ``<field />``
element as a children to the ``<entity />`` element. The field
element is only used for primitive types that are not the ID of the
entity. For the ID mapping you have to use the ``<id />`` element.

每个实体类可以包含0到无穷多个由 Doctrine managed的字段。你可以使用 ``<field />``
元素作为 ``<entity />`` 元素的子元素来定义它们。字段元素仅被用于非实体的 ID 的原始类型。
对于 ID 的映射你必须使用 ``<id />`` 元素。

.. code-block:: xml

    <entity name="MyProject\User">

        <field name="name" type="string" length="50" />
        <field name="username" type="string" unique="true" />
        <field name="age" type="integer" nullable="true" />
        <field name="isActive" column="is_active" type="boolean" />
        <field name="weight" type="decimal" scale="5" precision="2" />
        <field name="login_count" type="integer" nullable="false">
            <options>
                <option name="comment">The number of times the user has logged in.</option>
                <option name="default">0</option>
            </options>
        </field>
    </entity>

Required attributes:

必须的属性：

-  name - The name of the Property/Field on the given Entity PHP
   class.
-  name - 在给定实体 PHP 类上的属性/字段的名称。

Optional attributes:

可选的属性：

-  type - The ``Doctrine\DBAL\Types\Type`` name, defaults to
   "string"
-  type - ``Doctrine\DBAL\Types\Type`` 的名称，默认为“string”。
-  column - Name of the column in the database, defaults to the
   field name.
-  column - 在数据库中的列名称，默认为字段名称。
-  length - The length of the given type, for use with strings
   only.
-  length - 给定类型的长度，仅与字符串（string）一起使用。
-  unique - Should this field contain a unique value across the
   table? Defaults to false.
-  unique - 此字段是否应在表中包含唯一值？ 默认为 false。
-  nullable - Should this field allow NULL as a value? Defaults to
   false.
-  nullable - 此字段是否应允许 NULL 作为值？ 默认为 false。
-  version - Should this field be used for optimistic locking? Only
   works on fields with type integer or datetime.
-  version - 此字段是否应被用于乐观锁？仅在字段上与 integer 或 datetime 类型一起使用。
-  scale - Scale of a decimal type.
-  scale - decimal 类型的 scale。
-  precision - Precision of a decimal type.
-  precision - decimal 类型的精度。
-  options - Array of additional options:
-  options - 额外选项的数组：

   -  default - The default value to set for the column if no value
      is supplied.
   -  default - 如果列没有提供值，用此默认值设置。
   -  unsigned - Boolean value to determine if the column should
      be capable of representing only non-negative integers
      (applies only for integer column and might not be supported by
      all vendors).
   -  unsigned - 布尔值，用于确定列是否应该只能表示非负整数（仅适用于 integer 列且
      可能不被所有的提供商所支持）。
   -  fixed - Boolean value to determine if the specified length of
      a string column should be fixed or varying (applies only for
      string/binary column and might not be supported by all vendors).
   -  fixed - 布尔值，用于确定指定的字符串列的长度应该是固定的或可变的仅适用于 string/binar 列且
      可能不被所有的提供商所支持）。
   -  comment - The comment of the column in the schema (might not
      be supported by all vendors).
   -  comment - 在数据库（schema）中列的注释（可能不被所有提供商所支持）。
   -  customSchemaOptions - Array of additional schema options
      which are mostly vendor specific.
   -  customSchemaOptions - 额外的数据库（schema）选项的数组，通常是提供商特定的选项。
-  column-definition - Optional alternative SQL representation for
   this column. This definition begin after the field-name and has to
   specify the complete column definition. Using this feature will
   turn this field dirty for Schema-Tool update commands at all
   times.
-  column-definition - 可选的用于替换此列的 SQL 表示。此定义在字段名后面开始，
   必须指定完整的列定义。无论何时，对于 Schema-Tool 的更新命令，使用此功能将使此字段变“赃”。

.. note::

    For more detailed information on each attribute, please refer to
    the DBAL ``Schema-Representation`` documentation.

    有关每个属性的更多详细信息，请参阅 DBAL  ``Schema-Representation`` 文档。

Defining Identity and Generator Strategies // 定义标识和生成器策略
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

An entity has to have at least one ``<id />`` element. For
composite keys you can specify more than one id-element, however
surrogate keys are recommended for use with Doctrine 2. The Id
field allows to define properties of the identifier and allows a
subset of the ``<field />`` element attributes:

实体必须拥有至少一个 ``<id />`` 元素。对于复合键你可以指定超过一个 ID 元素，但是
推荐代理键与 Doctrine 2一起使用。ID 字段允许定义标识符的属性且允许 ``<field />``
元素属性的子集。

.. code-block:: xml

    <entity name="MyProject\User">
        <id name="id" type="integer" column="user_id" />
    </entity>

Required attributes:

必须的属性：

-  name - The name of the Property/Field on the given Entity PHP
   class.
-  name - 在给定实体 PHP 类上的属性/字段的名称。
-  type - The ``Doctrine\DBAL\Types\Type`` name, preferably
   "string" or "integer".
-  type - ``Doctrine\DBAL\Types\Type`` 的名称，最好“string” 或 “integer”。

Optional attributes:

可选的属性：

-  column - Name of the column in the database, defaults to the
   field name.
-  column - 在数据库中列的名称，默认为字段名称。

Using the simplified definition above Doctrine will use no
identifier strategy for this entity. That means you have to
manually set the identifier before calling
``EntityManager#persist($entity)``. This is the so called
``NONE`` strategy.

使用以上简化的定义，Doctrine 将不为此实体使用标识符策略。这意味着，在调用
``EntityManager#persist($entity)`` 之前你必须手动设置标识符。这就是所谓的
``NONE`` 策略。

If you want to switch the identifier generation strategy you have
to nest a ``<generator />`` element inside the id-element. This of
course only works for surrogate keys. For composite keys you always
have to use the ``NONE`` strategy.

如果你希望切换标识符生成策略你必须在 ID 元素内嵌套 ``<generator />`` 元素。
这当然只适用于代理键。 对于复合键，你始终必须使用 ``NONE`` 策略。

.. code-block:: xml

    <entity name="MyProject\User">
        <id name="id" type="integer" column="user_id">
            <generator strategy="AUTO" />
        </id>
    </entity>

The following values are allowed for the ``<generator />`` strategy
attribute:

 ``<generator />`` 的策略属性允许使用下列值：

-  AUTO - Automatic detection of the identifier strategy based on
   the preferred solution of the database vendor.
-  AUTO - 基于数据库提供商首先解决方案自动检测标识符策略。
-  IDENTITY - Use of a IDENTIFY strategy such as Auto-Increment IDs
   available to Doctrine AFTER the INSERT statement has been executed.
-  IDENTITY - 使用 IDENTIFY 策略，比如自增 IDs 可用于 Doctrine 在执行 INSERT 语句之后。
-  SEQUENCE - Use of a database sequence to retrieve the
   entity-ids. This is possible before the INSERT statement is
   executed.
-  SEQUENCE - 使用数据库 SEQUENCE 以取回实体的ids。这在 INSERT 语句被执行之前是可能的。

If you are using the SEQUENCE strategy you can define an additional
element to describe the sequence:

如果你使用 SEQUENCE 策略你可以定义一个额外元素以描述此 SEQUENCE：

.. code-block:: xml

    <entity name="MyProject\User">
        <id name="id" type="integer" column="user_id">
            <generator strategy="SEQUENCE" />
            <sequence-generator sequence-name="user_seq" allocation-size="5" initial-value="1" />
        </id>
    </entity>

Required attributes for ``<sequence-generator />``:

对于 ``<sequence-generator />`` 必须的选项：

-  sequence-name - The name of the sequence
-  sequence-name - sequence 的名称

Optional attributes for ``<sequence-generator />``:

对于 ``<sequence-generator />`` 可选的选项：

-  allocation-size - By how much steps should the sequence be
   incremented when a value is retrieved. Defaults to 1
-  allocation-size - 当取回值时，sequence 应该增加多少。默认为 1.
-  initial-value - What should the initial value of the sequence
   be.
-  initial-value - sequence 的初始值应该是多所。

    **NOTE**

    If you want to implement a cross-vendor compatible application you
    have to specify and additionally define the <sequence-generator />
    element, if Doctrine chooses the sequence strategy for a
    platform.

    如果你希望实现一个跨提供商兼容的应用程序你必须指定并额外地定义 <sequence-generator />
    元素，如果 Doctrine 为一个平台选择 sequence 策略。


Defining a Mapped Superclass // 定义一个映射超类
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Sometimes you want to define a class that multiple entities inherit
from, which itself is not an entity however. The chapter on
*Inheritance Mapping* describes a Mapped Superclass in detail. You
can define it in XML using the ``<mapped-superclass />`` tag.

有时你希望定义一个多个实体继承的类，但是本身不是一个实体。本章在*继承映射*上描述映射超类的详情。
你可以使用 ``<mapped-superclass />`` 标签在 XML 中定义它。 

.. code-block:: xml

    <doctrine-mapping>
        <mapped-superclass name="MyProject\BaseClass">
            <field name="created" type="datetime" />
            <field name="updated" type="datetime" />
        </mapped-superclass>
    </doctrine-mapping>

Required attributes:

必须的属性：

-  name - Class name of the mapped superclass.
-  name - 映射超类的类名。

You can nest any number of ``<field />`` and unidirectional
``<many-to-one />`` or ``<one-to-one />`` associations inside a
mapped superclass.

你可以在映射超类中嵌套任何数量的 ``<field />`` 和单向的 ``<many-to-one />``
或 ``<one-to-one />`` 关联。

Defining Inheritance Mappings // 定义继承映射
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are currently two inheritance persistence strategies that you
can choose from when defining entities that inherit from each
other. Single Table inheritance saves the fields of the complete
inheritance hierarchy in a single table, joined table inheritance
creates a table for each entity combining the fields using join
conditions.

当前有两种继承持久化策略,当定义彼此之间继承的实体时你可以从中选择。单一表继承在单个表中
保存完整的继承层次结构的字段，联结的表继承为每个实体创建一张表使用联结条件组合字段。

You can specify the inheritance type in the ``<entity />`` element
and then use the ``<discriminator-column />`` and
``<discriminator-mapping />`` attributes.

你可以在 ``<entity />`` 中指定继承类型，然后使用 ``<discriminator-column />`` 和
``<discriminator-mapping />``。

.. code-block:: xml

    <entity name="MyProject\Animal" inheritance-type="JOINED">
        <discriminator-column name="discr" type="string" />
        <discriminator-map>
            <discriminator-mapping value="cat" class="MyProject\Cat" />
            <discriminator-mapping value="dog" class="MyProject\Dog" />
            <discriminator-mapping value="mouse" class="MyProject\Mouse" />
        </discriminator-map>
    </entity>

The allowed values for inheritance-type attribute are ``JOINED`` or
``SINGLE_TABLE``.

对于继承类型属性允许的值是 ``JOINED`` 或 ``SINGLE_TABLE``。

.. note::

    All inheritance related definitions have to be defined on the root
    entity of the hierarchy.

    所有继承关联的定义必须在层次结构的根实体定义。

Defining Lifecycle Callbacks // 定义生命周期回调
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can define the lifecycle callback methods on your entities
using the ``<lifecycle-callbacks />`` element:

你可以在你的实体上使用 ``<lifecycle-callbacks />`` 定义生命周期回调：

.. code-block:: xml

    <entity name="Doctrine\Tests\ORM\Mapping\User" table="cms_users">

        <lifecycle-callbacks>
            <lifecycle-callback type="prePersist" method="onPrePersist" />
        </lifecycle-callbacks>
    </entity>

Defining One-To-One Relations // 定义 One-To-One 关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can define One-To-One Relations/Associations using the
``<one-to-one />`` element. The required and optional attributes
depend on the associations being on the inverse or owning side.

你可以使用 ``<one-to-one />`` 元素定义 One-To-One 关联。必须的和可选的属性依赖于关联所属的
inverse 或 owning 侧。

For the inverse side the mapping is as simple as:

对于 inverse 侧的映射是很简单的：

.. code-block:: xml

    <entity class="MyProject\User">
        <one-to-one field="address" target-entity="Address" mapped-by="user" />
    </entity>

Required attributes for inverse One-To-One:

对于 inverse One-To-One 必须的属性：

-  field - Name of the property/field on the entity's PHP class.
-  field - 在实体的 PHP 类上的属性/字段的名称。
-  target-entity - Name of the entity associated entity class. If
   this is not qualified the namespace of the current class is
   prepended. *IMPORTANT:* No leading backslash!
-  target-entity - 关联的实体类的名称。如果这不是限定的，预值当前类的命名空间。
   *重要：* 无前导反斜杠！
-  mapped-by - Name of the field on the owning side (here Address
   entity) that contains the owning side association.
-  mapped-by - 在 owning 侧（这里的 Address 实体）上字段的名称，包含 owning 侧关联。

For the owning side this mapping would look like:

对于 owning 侧这个映射将看起来像：

.. code-block:: xml

    <entity class="MyProject\Address">
        <one-to-one field="user" target-entity="User" inversed-by="address" />
    </entity>

Required attributes for owning One-to-One:

对于 owning One-to-One 的必须的属性:

-  field - Name of the property/field on the entity's PHP class.
-  field - 在实体的 PHP 类上的属性/字段的名称。
-  target-entity - Name of the entity associated entity class. If
   this is not qualified the namespace of the current class is
   prepended. *IMPORTANT:* No leading backslash!
-  target-entity - 关联的实体类的名称。如果这不是限定的，预值当前类的命名空间。
   *重要：* 无前导反斜杠！

Optional attributes for owning One-to-One:

对于 owning One-to-One 的可选的属性：

-  inversed-by - If the association is bidirectional the
   inversed-by attribute has to be specified with the name of the
   field on the inverse entity that contains the back-reference.
-  inversed-by - 如果关联是双向的，inversed-by 属性必须使用包含反向引用的
   inverse 实体上的字段的名称指定。
-  orphan-removal - If true, the inverse side entity is always
   deleted when the owning side entity is. Defaults to false.
-  orphan-removal - 如果为 true，当 owning 侧实体被删除，inverse 侧实体
   始终被删除，默认为 false。
-  fetch - Either LAZY or EAGER, defaults to LAZY. This attribute
   makes only sense on the owning side, the inverse side *ALWAYS* has
   to use the ``FETCH`` strategy.
-  fetch - LAZY 或 EAGER，默认为 LAZY。该属性仅在 owning 侧有意义，inverse 侧
   **始终** 必须使用 ``FETCH`` 策略。

The definition for the owning side relies on a bunch of mapping
defaults for the join column names. Without the nested
``<join-column />`` element Doctrine assumes to foreign key to be
called ``user_id`` on the Address Entities table. This is because
the ``MyProject\Address`` entity is the owning side of this
association, which means it contains the foreign key.

owning 侧的定义依赖于联结列名的一组映射默认值。没有嵌套的 ``<join-column />`` 元素，
Doctrine 假设外键在 Address 实体表上被称为 ``user_id``。这是为何 ``MyProject\Address``
实体是此关联的 owning 侧，这意味着它包含该外键。

The completed explicitly defined mapping is:

完整的明确定义的映射是：

.. code-block:: xml

    <entity class="MyProject\Address">
        <one-to-one field="user" target-entity="User" inversed-by="address">
            <join-column name="user_id" referenced-column-name="id" />
        </one-to-one>
    </entity>

Defining Many-To-One Associations // 定义 Many-To-One 关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The many-to-one association is *ALWAYS* the owning side of any
bidirectional association. This simplifies the mapping compared to
the one-to-one case. The minimal mapping for this association looks
like:

many-to-one 关联**始终**是任何双向的关联的 owning 侧。相较于 one-to-one 的情况，这简化了映射。
对于该关联的最小映射看上去像：

.. code-block:: xml

    <entity class="MyProject\Article">
        <many-to-one field="author" target-entity="User" />
    </entity>

Required attributes:

必须的属性：

-  field - Name of the property/field on the entity's PHP class.
-  field - 在实体的 PHP 类上的属性/字段的名称。
-  target-entity - Name of the entity associated entity class. If
   this is not qualified the namespace of the current class is
   prepended. *IMPORTANT:* No leading backslash!
-  target-entity - 关联的实体类的名称。如果这不是限定的，预值当前类的命名空间。
   *重要：* 无前导反斜杠！

Optional attributes:

可选的属性：

-  inversed-by - If the association is bidirectional the
   inversed-by attribute has to be specified with the name of the
   field on the inverse entity that contains the back-reference.
-  inversed-by - 如果关联是双向的，inversed-by 属性必须使用包含反向引用的
   inverse 实体上的字段的名称指定。
-  orphan-removal - If true the entity on the inverse side is
   always deleted when the owning side entity is and it is not
   connected to any other owning side entity anymore. Defaults to
   false.
-  orphan-removal - 如果为 true，当 owning 侧实体被删除，inverse 侧实体
   始终被删除并且它不在与任何其他 owning 侧实体连接。默认为 false。
-  fetch - Either LAZY or EAGER, defaults to LAZY.
-  fetch - LAZY 或 EAGER，默认为 LAZY。

This definition relies on a bunch of mapping defaults with regards
to the naming of the join-column/foreign key. The explicitly
defined mapping includes a ``<join-column />`` tag nested inside
the many-to-one association tag:

此定义依赖于使用有关联结列/外键的命名的一组映射默认值。明确定义的映射包含嵌套在 many-to-one
关联标签内的 ``<join-column />`` 标签。

.. code-block:: xml

    <entity class="MyProject\Article">
        <many-to-one field="author" target-entity="User">
            <join-column name="author_id" referenced-column-name="id" />
        </many-to-one>
    </entity>

The join-column attribute ``name`` specifies the column name of the
foreign key and the ``referenced-column-name`` attribute specifies
the name of the primary key column on the User entity.

在 User 实体上联结列属性 ``name`` 指定外键的列名并且 ``referenced-column-name``
属性指定主键列的名称。

Defining One-To-Many Associations // 定义 One-To-Many 关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The one-to-many association is *ALWAYS* the inverse side of any
association. There exists no such thing as a uni-directional
one-to-many association, which means this association only ever
exists for bi-directional associations.

one-to-many 关联**始终**是任何关联的 inverse 侧。不存在诸如单向的 one-to-many
关联之类的东西，这意味着这种关联仅存在于双向关联。

.. code-block:: xml

    <entity class="MyProject\User">
        <one-to-many field="phonenumbers" target-entity="Phonenumber" mapped-by="user" />
    </entity>

Required attributes:

必须的属性：

-  field - Name of the property/field on the entity's PHP class.
-  field - 在实体的 PHP 类上的属性/字段的名称。
-  target-entity - Name of the entity associated entity class. If
   this is not qualified the namespace of the current class is
   prepended. *IMPORTANT:* No leading backslash!
-  target-entity - 关联的实体类的名称。如果这不是限定的，预值当前类的命名空间。
   *重要：* 无前导反斜杠！
-  mapped-by - Name of the field on the owning side (here
   Phonenumber entity) that contains the owning side association.
-  mapped-by - 在 owning 侧（这里的 Phonenumber 实体）上字段的名称，包含 owning 侧关联。

Optional attributes:

可选的属性：

-  fetch - Either LAZY, EXTRA_LAZY or EAGER, defaults to LAZY.
-  fetch - LAZY、EXTRA_LAZY 或 EAGER，默认为 LAZY。
-  index-by - Index the collection by a field on the target entity.
-  index-by - 由目标实体上的字段索引的集合。

Defining Many-To-Many Associations // 定义 Many-To-Many 关联
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From all the associations the many-to-many has the most complex
definition. When you rely on the mapping defaults you can omit many
definitions and rely on their implicit values.

所有的关联，many-to-many 拥有最复杂的定义。当你依赖于映射默认值，你可以忽略许多定义并且
依赖于它们的隐式值。

.. code-block:: xml

    <entity class="MyProject\User">
        <many-to-many field="groups" target-entity="Group" />
    </entity>

Required attributes:

必须的属性：

-  field - Name of the property/field on the entity's PHP class.
-  field - 在实体的 PHP 类上的属性/字段的名称。
-  target-entity - Name of the entity associated entity class. If
   this is not qualified the namespace of the current class is
   prepended. *IMPORTANT:* No leading backslash!
-  target-entity - 关联的实体类的名称。如果这不是限定的，预值当前类的命名空间。
   *重要：* 无前导反斜杠！

Optional attributes:

可选的属性：

-  mapped-by - Name of the field on the owning side that contains
   the owning side association if the defined many-to-many association
   is on the inverse side.
-  mapped-by - 在 owning 侧上字段的名称，包含 owning 侧关联，如果定义的 many-to-many
   关联是在 inverse 侧上。
-  inversed-by - If the association is bidirectional the
   inversed-by attribute has to be specified with the name of the
   field on the inverse entity that contains the back-reference.
-  inversed-by - 如果关联是双向的，inversed-by 属性必须使用包含反向引用的
   inverse 实体上的字段的名称指定。
-  fetch - Either LAZY, EXTRA_LAZY or EAGER, defaults to LAZY.
-  fetch - LAZY、EXTRA_LAZY 或 EAGER，默认为 LAZY。
-  index-by - Index the collection by a field on the target entity.
-  index-by - 由目标实体上的字段索引的集合。

The mapping defaults would lead to a join-table with the name
"User\_Group" being created that contains two columns "user\_id"
and "group\_id". The explicit definition of this mapping would be:

映射的默认值将导致一个名为“User\_Group”的联结表被创建，它包含两个列“user\_id”和“group\_id”。
此映射的明确定义将是：

.. code-block:: xml

    <entity class="MyProject\User">
        <many-to-many field="groups" target-entity="Group">
            <join-table name="cms_users_groups">
                <join-columns>
                    <join-column name="user_id" referenced-column-name="id"/>
                </join-columns>
                <inverse-join-columns>
                    <join-column name="group_id" referenced-column-name="id"/>
                </inverse-join-columns>
            </join-table>
        </many-to-many>
    </entity>

Here both the ``<join-columns>`` and ``<inverse-join-columns>``
tags are necessary to tell Doctrine for which side the specified
join-columns apply. These are nested inside a ``<join-table />``
attribute which allows to specify the table name of the
many-to-many join-table.

这里的 ``<join-columns>`` 和 ``<inverse-join-columns>`` 标签是必须的以便告诉 Doctrine
联结列被指定用于哪一侧。这些嵌套在一个 ``<join-table />`` 属性中，允许指定 many-to-many 联结表的表名。

Cascade Element // 级联元素
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Doctrine allows cascading of several UnitOfWork operations to
related entities. You can specify the cascade operations in the
``<cascade />`` element inside any of the association mapping
tags.

Doctrine 允许几个 UnitOfWork 操作级联到关联的实体。你可以在任何的关联映射标签内的``<cascade />``
中指定级联操作。

.. code-block:: xml

    <entity class="MyProject\User">
        <many-to-many field="groups" target-entity="Group">
            <cascade>
                <cascade-all/>
            </cascade>
        </many-to-many>
    </entity>

Besides ``<cascade-all />`` the following operations can be
specified by their respective tags:

除了 ``<cascade-all />``，以下操作可以由各自的标签指定：

-  ``<cascade-persist />``
-  ``<cascade-merge />``
-  ``<cascade-remove />``
-  ``<cascade-refresh />``

Join Column Element // 联结列元素
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In any explicitly defined association mapping you will need the
``<join-column />`` tag. It defines how the foreign key and primary
key names are called that are used for joining two entities.

在任何显式定义的关联映射中你将需要 ``<join-column />`` 标签。它定义如何命名外键和主键的名称以
用于联结两个实体。

Required attributes:

必要的属性：

-  name - The column name of the foreign key.
-  name - 外键的列名称。
-  referenced-column-name - The column name of the associated
   entities primary key
-  referenced-column-name - 关联的实体主键列的名称。

Optional attributes:

可选的属性：

-  unique - If the join column should contain a UNIQUE constraint.
   This makes sense for Many-To-Many join-columns only to simulate a
   one-to-many unidirectional using a join-table.
-  unique - 是否联结列应包含唯一约束。对于 Many-To-Many 的联结列只使用联结表来模拟一个
   单向的 one-to-many 时是有意义的。
-  nullable - should the join column be nullable, defaults to true.
-  nullable - 联结列是否可空，默认为 true。
-  on-delete - Foreign Key Cascade action to perform when entity is
   deleted, defaults to NO ACTION/RESTRICT but can be set to
   "CASCADE".
-  on-delete - 当实体被删除时执行的外键级联操作，默认为 NO ACTION/RESTRICT，但是可以被设置为
   “CASCADE”。

Defining Order of To-Many Associations // 定义 To-Many 关联的顺序
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can require one-to-many or many-to-many associations to be
retrieved using an additional ``ORDER BY``.

你可能需要使用额外的 ``ORDER BY`` 来取回 one-to-many 或 many-to-many 的关联。

.. code-block:: xml

    <entity class="MyProject\User">
        <many-to-many field="groups" target-entity="Group">
            <order-by>
                <order-by-field name="name" direction="ASC" />
            </order-by>
        </many-to-many>
    </entity>

Defining Indexes or Unique Constraints // 定义索引或唯一约束
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To define additional indexes or unique constraints on the entities
table you can use the ``<indexes />`` and
``<unique-constraints />`` elements:

为在实体表上定义额外的索引或唯一约束，你可以使用 ``<indexes />`` 和 ``<unique-constraints />`` 元素：

.. code-block:: xml

    <entity name="Doctrine\Tests\ORM\Mapping\User" table="cms_users">

        <indexes>
            <index name="name_idx" columns="name"/>
            <index columns="user_email"/>
        </indexes>

        <unique-constraints>
            <unique-constraint columns="name,user_email" name="search_idx" />
        </unique-constraints>
    </entity>

You have to specify the column and not the entity-class field names
in the index and unique-constraint definitions.

你必须在索引和唯一约束定义中指定列而不是实体类的字段名。

Derived Entities ID syntax // 派生实体 ID 语法
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the primary key of an entity contains a foreign key to another entity we speak of a derived
entity relationship. You can define this in XML with the "association-key" attribute in the ``<id>`` tag.

如果实体的主键包含另一个实体的一个外键，我们叫派生实体关联。你可以在 XML 中使用在 ``<id>`` 标签中的“association-key”属性定义它。

.. code-block:: xml

    <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                        http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

         <entity name="Application\Model\ArticleAttribute">
            <id name="article" association-key="true" />
            <id name="attribute" type="string" />

            <field name="value" type="string" />

            <many-to-one field="article" target-entity="Article" inversed-by="attributes" />
         </entity>

    </doctrine-mapping>
