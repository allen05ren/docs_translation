Inheritance Mapping // 继承映射
====================================

Mapped Superclasses // 映射超类
-------------------------------------

A mapped superclass is an abstract or concrete class that provides
persistent entity state and mapping information for its subclasses,
but which is not itself an entity. Typically, the purpose of such a
mapped superclass is to define state and mapping information that
is common to multiple entity classes.

映射超类是一个抽象的或具体的类，它为其子类提供持久化的实体状态和映射信息，但本身不是一个实体。
通常，映射超类的目的是定义多个实体类共用的状态和映射信息。

Mapped superclasses, just as regular, non-mapped classes, can
appear in the middle of an otherwise mapped inheritance hierarchy
(through Single Table Inheritance or Class Table Inheritance).

映射超类，如同规则的、无映射类，可以出现在其他映射继承层次结构中间（通过单一表继承或类表继承）。

.. note::

    A mapped superclass cannot be an entity, it is not query-able and
    persistent relationships defined by a mapped superclass must be
    unidirectional (with an owning side only). This means that One-To-Many
    associations are not possible on a mapped superclass at all.
    Furthermore Many-To-Many associations are only possible if the
    mapped superclass is only used in exactly one entity at the moment.
    For further support of inheritance, the single or
    joined table inheritance features have to be used.

    映射超类不能是实体，这是毋庸置疑的，并且通过映射超类定义的持久化的关联必须是单向的（仅具有 owning 一侧）。
    这意味着 One-To-Many 关联在映射超类上是根本不可能的。此外，Many-To-Many 关联仅当映射超类被用在合适的当前实体上时才有可能。
    为了更进一步支持继承，单一或联结表继承特性必须被使用。


Example:

例子：

.. code-block:: php

    <?php
    /** @MappedSuperclass */
    class MappedSuperclassBase
    {
        /** @Column(type="integer") */
        protected $mapped1;
        /** @Column(type="string") */
        protected $mapped2;
        /**
         * @OneToOne(targetEntity="MappedSuperclassRelated1")
         * @JoinColumn(name="related1_id", referencedColumnName="id")
         */
        protected $mappedRelated1;
    
        // ... more fields and methods
    }
    
    /** @Entity */
    class EntitySubClass extends MappedSuperclassBase
    {
        /** @Id @Column(type="integer") */
        private $id;
        /** @Column(type="string") */
        private $name;
    
        // ... more fields and methods
    }

The DDL for the corresponding database schema would look something
like this (this is for SQLite):

对于相应的数据库 schema，DDL 看起来像这样（对于 SQLite）：

.. code-block:: sql

    CREATE TABLE EntitySubClass (mapped1 INTEGER NOT NULL, mapped2 TEXT NOT NULL, id INTEGER NOT NULL, name TEXT NOT NULL, related1_id INTEGER DEFAULT NULL, PRIMARY KEY(id))

As you can see from this DDL snippet, there is only a single table
for the entity subclass. All the mappings from the mapped
superclass were inherited to the subclass as if they had been
defined on that class directly.

正如你看到的这个 DDL 片断，仅有单一表的实体子类。子类继承了映射超类的所有映射，就好像是它们直接在子类中定义的一样。

Single Table Inheritance // 单一表继承
-------------------------------------------

`Single Table Inheritance <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
is an inheritance mapping strategy where all classes of a hierarchy
are mapped to a single database table. In order to distinguish
which row represents which type in the hierarchy a so-called
discriminator column is used.

`单一表继承 <http://martinfowler.com/eaaCatalog/singleTableInheritance.html>`_
是一个继承映射策略，层次结构的所有类被映射至数据库单一表中。
为了区分哪一行代表哪一类型，在层次结构中会使用一个号称鉴别器的列。

Example:

例子：

.. configuration-block::

    .. code-block:: php
    
        <?php
        namespace MyProject\Model;
        
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
         */
        class Employee extends Person
        {
            // ...
        }

    .. code-block:: yaml
    
        MyProject\Model\Person:
          type: entity
          inheritanceType: SINGLE_TABLE
          discriminatorColumn:
            name: discr
            type: string
          discriminatorMap:
            person: Person
            employee: Employee
                
        MyProject\Model\Employee:
          type: entity
            
Things to note:

注意事项：

-  The @InheritanceType and @DiscriminatorColumn must be specified 
   on the topmost class that is part of the mapped entity hierarchy.
-  @InheritanceType 和 @DiscriminatorColumn 必须在最顶层的类被指定，它们是映射实体层次结构的一部分。
-  The @DiscriminatorMap specifies which values of the
   discriminator column identify a row as being of a certain type. In
   the case above a value of "person" identifies a row as being of
   type ``Person`` and "employee" identifies a row as being of type
   ``Employee``.
-  @DiscriminatorMap 指定了鉴别器列的值所标识的行是某个类型。
   上述例子中，“person”的值所标识的行是 ``Person`` 类型，“employee”的值所标识的行是 ``Employee`` 类型。
-  All entity classes that is part of the mapped entity hierarchy
   (including the topmost class) should be specified in the
   @DiscriminatorMap. In the case above Person class included.
-  映射实体层次结构的所有实体类（包括最顶层类）都应该在 @DiscriminatorMap 中指定。
   上述例子中，包括了 Person 类。
-  The names of the classes in the discriminator map do not need to
   be fully qualified if the classes are contained in the same
   namespace as the entity class on which the discriminator map is
   applied.
-  鉴别器映射（discriminator map）中类的名称不需要是完全限定的，如果这些类包含在应用了鉴别器映射的实体类相同的命名空间中。
-  If no discriminator map is provided, then the map is generated
   automatically. The automatically generated discriminator map 
   contains the lowercase short name of each class as key.
-  如果鉴别器映射没有被提供，那么映射会自动生成。自动生成的鉴别器映射包含每个类的小写短名称作为键。

Design-time considerations // 设计时注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This mapping approach works well when the type hierarchy is fairly
simple and stable. Adding a new type to the hierarchy and adding
fields to existing supertypes simply involves adding new columns to
the table, though in large deployments this may have an adverse
impact on the index and column layout inside the database.

当类型层次结构足够简单和稳定时，此映射方法工作良好。
添加一个新类型至层次结构和添加字段至现有超类型，只涉及添加新列至表，
尽管大规模部署这可能在数据库内部的索引和列规划上有不良的影响。

Performance impact // 性能影响
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This strategy is very efficient for querying across all types in
the hierarchy or for specific types. No table joins are required,
only a WHERE clause listing the type identifiers. In particular,
relationships involving types that employ this mapping strategy are
very performant.

此策略是非常高效的，对于查询跨层次结构中的所有类型或特定类型。
无需表联结，仅一个 WHERE 字句列出那些类型的标识符。
尤其是，涉及部署该映射策略的类型的关联是非常高效的。

There is a general performance consideration with Single Table
Inheritance: If the target-entity of a many-to-one or one-to-one 
association is an STI entity, it is preferable for performance reasons that it 
be a leaf entity in the inheritance hierarchy, (ie. have no subclasses). 
Otherwise Doctrine *CANNOT* create proxy instances
of this entity and will *ALWAYS* load the entity eagerly.

使用单一表继承出于性能考虑：如果一个 many-to-one 或 one-to-one 关联的目标实体是一个 STI（Single Table
Inheritance）实体，出于性能考虑，在继承层次结构中它最好是叶片实体（leaf entity）（即没有子类）。此外，
Doctrine *不能* 创建该实体的代理实例并且将 *总是* 急切地加载该实体。

SQL Schema considerations // SQL Schema 注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For Single-Table-Inheritance to work in scenarios where you are
using either a legacy database schema or a self-written database
schema you have to make sure that all columns that are not in the
root entity but in any of the different sub-entities has to allows
null values. Columns that have NOT NULL constraints have to be on
the root entity of the single-table inheritance hierarchy.

为使单一表继承能够在旧版的数据库 schema 或自写的数据库 schema 情景中工作，你必须确保所有不在根实体中
却在任何不同的子实体中的列必须允许空值（null values）。有非空（NOT NULL）约束的列
必须是在单一表继承层次结构的根实体上。

Class Table Inheritance // 类表继承
----------------------------------------

`Class Table Inheritance <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
is an inheritance mapping strategy where each class in a hierarchy
is mapped to several tables: its own table and the tables of all
parent classes. The table of a child class is linked to the table
of a parent class through a foreign key constraint. Doctrine 2
implements this strategy through the use of a discriminator column
in the topmost table of the hierarchy because this is the easiest
way to achieve polymorphic queries with Class Table Inheritance.

`类表继承 <http://martinfowler.com/eaaCatalog/classTableInheritance.html>`_
是一个继承映射策略，层次结构中的每一个类被映射至几个表：它自身表和所有父类的表。
子类的表通过一个外键约束被链接至一个父类的表。Doctrine 2通过在层次结构中最顶层的表中
使用一个鉴别器列实现该策略，因为这是最简单的用类表继承达到多态查询的方式。

Example:

例子：

.. code-block:: php

    <?php
    namespace MyProject\Model;
    
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
    
    /** @Entity */
    class Employee extends Person
    {
        // ...
    }

Things to note:

注意事项：

-  The @InheritanceType, @DiscriminatorColumn and @DiscriminatorMap
   must be specified on the topmost class that is part of the mapped
   entity hierarchy.
-  @InheritanceType、 @DiscriminatorColumn 和 @DiscriminatorMap 必须在最顶层的类中被指定，
   因为它们是映射实体层次结构的一部分。
-  The @DiscriminatorMap specifies which values of the
   discriminator column identify a row as being of which type. In the
   case above a value of "person" identifies a row as being of type
   ``Person`` and "employee" identifies a row as being of type
   ``Employee``.
-  @DiscriminatorMap 指定了鉴别器列的值所标识的行是某个类型。上述例子中，“person”的值所标识的行是
   ``Person`` 类型，“employee”的值所标识的行是 ``Employee`` 类型。
-  The names of the classes in the discriminator map do not need to
   be fully qualified if the classes are contained in the same
   namespace as the entity class on which the discriminator map is
   applied.
-  鉴别器映射（discriminator map）中类的名称不需要是完全限定的，如果这些类包含在应用了鉴别器映射的实体类相同的命名空间中。
-  If no discriminator map is provided, then the map is generated
   automatically. The automatically generated discriminator map 
   contains the lowercase short name of each class as key.
-  如果鉴别器映射没有被提供，那么映射会自动生成。自动生成的鉴别器映射包含每个类的小写短名称作为键。

.. note::

    When you do not use the SchemaTool to generate the
    required SQL you should know that deleting a class table
    inheritance makes use of the foreign key property
    ``ON DELETE CASCADE`` in all database implementations. A failure to
    implement this yourself will lead to dead rows in the database.

    当你不使用 SchemaTool 来生成所需的 SQL，你应当知道利用在所有数据库中实现的外键属性
    ``ON DELETE CASCADE`` 来删除一个类表继承，你自己未能实现，这将导致死行（dead rows）
    在数据库中。


Design-time considerations // 设计时注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Introducing a new type to the hierarchy, at any level, simply
involves interjecting a new table into the schema. Subtypes of that
type will automatically join with that new type at runtime.
Similarly, modifying any entity type in the hierarchy by adding,
modifying or removing fields affects only the immediate table
mapped to that type. This mapping strategy provides the greatest
flexibility at design time, since changes to any type are always
limited to that type's dedicated table.

在层次结构中的任何层次引入一个新的类型，仅涉及插入一个新表到数据库（schema）中。该类型的子类型将自动与新的类型联结在运行时。
类似地，在层次结构中通过添加、修改、移除字段修改任何实体类型仅影响映射至该实体类型的当前表。在设计阶段，
该映射策略提供了最好的灵活性，因为更改为任何类型总是限制在那个类型的专门的表。

Performance impact // 性能影响
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This strategy inherently requires multiple JOIN operations to
perform just about any query which can have a negative impact on
performance, especially with large tables and/or large hierarchies.
When partial objects are allowed, either globally or on the
specific query, then querying for any type will not cause the
tables of subtypes to be OUTER JOINed which can increase
performance but the resulting partial objects will not fully load
themselves on access of any subtype fields, so accessing fields of
subtypes after such a query is not safe.

该策略本质上需要多联结操作以执行几乎任何查询，这在性能上会有负面影响，特别是与大的表和/或大的层次结构。
当全局地或在特定的查询上，不完整对象（partial objects）被允许时，那么对于任何类型的查询将不引起
子类型的表被外联结，那可以提高性能但导致了在任何子类型字段的访问上不完整对象将不会完全加载自身，
所以这样的一个查询之后访问子类型的字段是不安全的。

There is a general performance consideration with Class Table
Inheritance: If the target-entity of a many-to-one or one-to-one 
association is a CTI entity, it is preferable for performance reasons that it 
be a leaf entity in the inheritance hierarchy, (ie. have no subclasses). 
Otherwise Doctrine *CANNOT* create proxy instances
of this entity and will *ALWAYS* load the entity eagerly.

使用类表继承出于性能考虑：如果一个 many-to-one 或 one-to-one 的目标实体是一个 CTI（Class Table
Inheritance）实体，出于性能考虑，在继承层次结构中它最好是叶片实体（leaf entity）（即没有子类）。此外，
Doctrine *不能* 创建该实体的代理实例并且将 *总是* 急切地加载该实体。

SQL Schema considerations // SQL Schema 注意事项
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For each entity in the Class-Table Inheritance hierarchy all the
mapped fields have to be columns on the table of this entity.
Additionally each child table has to have an id column that matches
the id column definition on the root table (except for any sequence
or auto-increment details). Furthermore each child table has to
have a foreign key pointing from the id column to the root table id
column and cascading on delete.

类表继承的层次结构中的每个实体的所有映射字段必须是该实体的表上的列。此外，
每个子表必须有一个 id 列，该列匹配定义在根表上的 id 列（除了任何序列或自增的细节）。
另外，每个子表必须有一个外键从那个 id 列指向根表的 id 列且级联删除。

.. _inheritence_mapping_overrides:

Overrides // 重载
------------------------
Used to override a mapping for an entity field or relationship.
May be applied to an entity that extends a mapped superclass
to override a relationship or field mapping defined by the mapped superclass.

用于重载实体字段或关联的映射。
可被应用于扩展自映射超类的实体重载通过该映射超类定义的关联或字段映射。

Association Override // 关联重载
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Override a mapping for an entity relationship.

重载实体关联的映射。

Could be used by an entity that extends a mapped superclass
to override a relationship mapping defined by the mapped superclass.

能够被用于扩展自映射超类的实体重载通过该映射超类定义的关联的映射。

Example:

例子：

.. configuration-block::

    .. code-block:: php

        <?php
        // user mapping
        namespace MyProject\Model;
        /**
         * @MappedSuperclass
         */
        class User
        {
            //other fields mapping

            /**
             * @ManyToMany(targetEntity="Group", inversedBy="users")
             * @JoinTable(name="users_groups",
             *  joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *  inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
             * )
             */
            protected $groups;

            /**
             * @ManyToOne(targetEntity="Address")
             * @JoinColumn(name="address_id", referencedColumnName="id")
             */
            protected $address;
        }

        // admin mapping
        namespace MyProject\Model;
        /**
         * @Entity
         * @AssociationOverrides({
         *      @AssociationOverride(name="groups",
         *          joinTable=@JoinTable(
         *              name="users_admingroups",
         *              joinColumns=@JoinColumn(name="adminuser_id"),
         *              inverseJoinColumns=@JoinColumn(name="admingroup_id")
         *          )
         *      ),
         *      @AssociationOverride(name="address",
         *          joinColumns=@JoinColumn(
         *              name="adminaddress_id", referencedColumnName="id"
         *          )
         *      )
         * })
         */
        class Admin extends User
        {
        }

    .. code-block:: xml

        <!-- user mapping -->
        <doctrine-mapping>
          <mapped-superclass name="MyProject\Model\User">
                <!-- other fields mapping -->
                <many-to-many field="groups" target-entity="Group" inversed-by="users">
                    <cascade>
                        <cascade-persist/>
                        <cascade-merge/>
                        <cascade-detach/>
                    </cascade>
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </mapped-superclass>
        </doctrine-mapping>

        <!-- admin mapping -->
        <doctrine-mapping>
            <entity name="MyProject\Model\Admin">
                <association-overrides>
                    <association-override name="groups">
                        <join-table name="users_admingroups">
                            <join-columns>
                                <join-column name="adminuser_id"/>
                            </join-columns>
                            <inverse-join-columns>
                                <join-column name="admingroup_id"/>
                            </inverse-join-columns>
                        </join-table>
                    </association-override>
                    <association-override name="address">
                        <join-columns>
                            <join-column name="adminaddress_id" referenced-column-name="id"/>
                        </join-columns>
                    </association-override>
                </association-overrides>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        # user mapping
        MyProject\Model\User:
          type: mappedSuperclass
          # other fields mapping
          manyToOne:
            address:
              targetEntity: Address
              joinColumn:
                name: address_id
                referencedColumnName: id
              cascade: [ persist, merge ]
          manyToMany:
            groups:
              targetEntity: Group
              joinTable:
                name: users_groups
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  group_id:
                    referencedColumnName: id
              cascade: [ persist, merge, detach ]

        # admin mapping
        MyProject\Model\Admin:
          type: entity
          associationOverride:
            address:
              joinColumn:
                adminaddress_id:
                  name: adminaddress_id
                  referencedColumnName: id
            groups:
              joinTable:
                name: users_admingroups
                joinColumns:
                  adminuser_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  admingroup_id:
                    referencedColumnName: id


Things to note:

注意事项：

-  The "association override" specifies the overrides base on the property name.
-  关联重载指定基于属性名之上的重载。
-  This feature is available for all kind of associations. (OneToOne, OneToMany, ManyToOne, ManyToMany)
-  该特性对所有的关联可用。（OneToOne, OneToMany, ManyToOne, ManyToMany）
-  The association type *CANNOT* be changed.
-  关联类型*不能*被更改。
-  The override could redefine the joinTables or joinColumns depending on the association type.
-  重载能够重新定义联结表或联结列，根据关联类型。
-  The override could redefine inversedBy to reference more than one extended entity.
-  重载能够重新定义 inversedBy 引用超过一个扩展的实体。

Attribute Override // 属性重载
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Override the mapping of a field.

重载一个字段的映射。

Could be used by an entity that extends a mapped superclass to override a field mapping defined by the mapped superclass.

能够被用于扩展自映射超类的实体重载通过该映射超类定义的字段的映射。

.. configuration-block::

    .. code-block:: php

        <?php
        // user mapping
        namespace MyProject\Model;
        /**
         * @MappedSuperclass
         */
        class User
        {
            /** @Id @GeneratedValue @Column(type="integer", name="user_id", length=150) */
            protected $id;

            /** @Column(name="user_name", nullable=true, unique=false, length=250) */
            protected $name;

            // other fields mapping
        }

        // guest mapping
        namespace MyProject\Model;
        /**
         * @Entity
         * @AttributeOverrides({
         *      @AttributeOverride(name="id",
         *          column=@Column(
         *              name     = "guest_id",
         *              type     = "integer",
                        length   = 140
         *          )
         *      ),
         *      @AttributeOverride(name="name",
         *          column=@Column(
         *              name     = "guest_name",
         *              nullable = false,
         *              unique   = true,
                        length   = 240
         *          )
         *      )
         * })
         */
        class Guest extends User
        {
        }

    .. code-block:: xml

        <!-- user mapping -->
        <doctrine-mapping>
          <mapped-superclass name="MyProject\Model\User">
                <id name="id" type="integer" column="user_id" length="150">
                    <generator strategy="AUTO"/>
                </id>
                <field name="name" column="user_name" type="string" length="250" nullable="true" unique="false" />
                <many-to-one field="address" target-entity="Address">
                    <cascade>
                        <cascade-persist/>
                        <cascade-merge/>
                    </cascade>
                    <join-column name="address_id" referenced-column-name="id"/>
                </many-to-one>
                <!-- other fields mapping -->
            </mapped-superclass>
        </doctrine-mapping>

        <!-- admin mapping -->
        <doctrine-mapping>
            <entity name="MyProject\Model\Guest">
                <attribute-overrides>
                    <attribute-override name="id">
                        <field column="guest_id" length="140"/>
                    </attribute-override>
                    <attribute-override name="name">
                        <field column="guest_name" type="string" length="240" nullable="false" unique="true" />
                    </attribute-override>
                </attribute-overrides>
            </entity>
        </doctrine-mapping>
    .. code-block:: yaml

        # user mapping
        MyProject\Model\User:
          type: mappedSuperclass
          id:
            id:
              type: integer
              column: user_id
              length: 150
              generator:
                strategy: AUTO
          fields:
            name:
              type: string
              column: user_name
              length: 250
              nullable: true
              unique: false
          #other fields mapping


        # guest mapping
        MyProject\Model\Guest:
          type: entity
          attributeOverride:
            id:
              column: guest_id
              type: integer
              length: 140
            name:
              column: guest_name
              type: string
              length: 240
              nullable: false
              unique: true

Things to note:
注意事项：

-  The "attribute override" specifies the overrides base on the property name.
-  关联重载指定基于属性名之上的重载。
-  The column type *CANNOT* be changed. If the column type is not equal you get a ``MappingException``
-  列的类型*不能*被更改。如果列的类型不相等，你会得到一个 ``MappingException`` 异常。
-  The override can redefine all the columns except the type.
-  重载能够重新定义所有列属性除了列的类型。

Query the Type // 查询类型
---------------------------------

It may happen that the entities of a special type should be queried. Because there
is no direct access to the discriminator column, Doctrine provides the
``INSTANCE OF`` construct.

经常有查询实体类型的需要。由于不能直接访问鉴别器列，Doctrine 提供了 ``INSTANCE OF`` 构造。

The following example shows how to use ``INSTANCE OF``. There is a three level hierarchy
with a base entity ``NaturalPerson`` which is extended by ``Staff`` which in turn
is extended by ``Technician``.

下面的例子展示了如何使用 ``INSTANCE OF``。这里有一个三层层次结构的基础实体 ``NaturalPerson``，
``Staff`` 扩展自它，``Technician`` 扩展自 ``Staff``。

Querying for the staffs without getting any technicians can be achieved by this DQL:
通过以下 DQL 查询不包括任何技师（technicians）的职工（staffs）：
.. code-block:: php

    <?php
    $query = $em->createQuery("SELECT staff FROM MyProject\Model\Staff staff WHERE staff NOT INSTANCE OF MyProject\Model\Technician");
    $staffs = $query->getResult();
