PHP Mapping // PHP 映射
==============================

Doctrine 2 also allows you to provide the ORM metadata in the form
of plain PHP code using the ``ClassMetadata`` API. You can write
the code in PHP files or inside of a static function named
``loadMetadata($class)`` on the entity class itself.

Doctrine 2也允许你使用 ``ClassMetadata`` API 以纯 PHP 代码的形式提供 ORM 元数据。
你可以在 PHP 文件中写代码或者在实体类本身上的一个命名为 ``loadMetadata($class)``
的静态函数内部。

PHP Files // PHP 文件
----------------------------

If you wish to write your mapping information inside PHP files that
are named after the entity and included to populate the metadata
for an entity you can do so by using the ``PHPDriver``:

如果你希望在以实体命名的 PHP 文件内部写你的映射信息，并且包含在填充实体的元数据中，
你可以通过使用 ``PHPDriver``：

.. code-block:: php

    <?php
    $driver = new PHPDriver('/path/to/php/mapping/files');
    $em->getConfiguration()->setMetadataDriverImpl($driver);

Now imagine we had an entity named ``Entities\User`` and we wanted
to write a mapping file for it using the above configured
``PHPDriver`` instance:

现在设想我们已经有一个命名为 ``Entities\User`` 的实体，并且我们希望使用上面配置的
``PHPDriver`` 实例为它写一个映射文件：

.. code-block:: php

    <?php
    namespace Entities;

    class User
    {
        private $id;
        private $username;
    }

To write the mapping information you just need to create a file
named ``Entities.User.php`` inside of the
``/path/to/php/mapping/files`` folder:

为了写该映射信息你仅需要在 ``/path/to/php/mapping/files`` 文件夹内创建一个命名为
``Entities.User.php`` 的文件：

.. code-block:: php

    <?php
    // /path/to/php/mapping/files/Entities.User.php

    $metadata->mapField(array(
       'id' => true,
       'fieldName' => 'id',
       'type' => 'integer'
    ));

    $metadata->mapField(array(
       'fieldName' => 'username',
       'type' => 'string',
       'options' => array(
           'fixed' => true,
           'comment' => "User's login name"
       )
    ));

    $metadata->mapField(array(
       'fieldName' => 'login_count',
       'type' => 'integer',
       'nullable' => false,
       'options' => array(
           'unsigned' => true,
           'default' => 0
       )
    ));

Now we can easily retrieve the populated ``ClassMetadata`` instance
where the ``PHPDriver`` includes the file and the
``ClassMetadataFactory`` caches it for later retrieval:

现在我们可以容易地取回填充了 ``ClassMetadata`` 的实例，其中的 ``PHPDriver`` 包含该文件，并且
为了稍后取回 ``ClassMetadataFactory`` 将缓存它：

.. code-block:: php

    <?php
    $class = $em->getClassMetadata('Entities\User');
    // or
    $class = $em->getMetadataFactory()->getMetadataFor('Entities\User');

Static Function // 静态函数
----------------------------------

In addition to the PHP files you can also specify your mapping
information inside of a static function defined on the entity class
itself. This is useful for cases where you want to keep your entity
and mapping information together but don't want to use annotations.
For this you just need to use the ``StaticPHPDriver``:

此外，PHP 文件你也可以在实体类自身上定义的一个静态函数内部指定你的映射信息。对于你希望保留你的实体
和映射信息在一起但不希望使用注释的情况，这很有用。为此你仅需要使用 ``StaticPHPDriver``：

.. code-block:: php

    <?php
    $driver = new StaticPHPDriver('/path/to/entities');
    $em->getConfiguration()->setMetadataDriverImpl($driver);

Now you just need to define a static function named
``loadMetadata($metadata)`` on your entity:

现在你仅需要在你的实体上定义一个命名为 ``loadMetadata($metadata)`` 的静态函数：

.. code-block:: php

    <?php
    namespace Entities;

    use Doctrine\ORM\Mapping\ClassMetadata;

    class User
    {
        // ...

        public static function loadMetadata(ClassMetadata $metadata)
        {
            $metadata->mapField(array(
               'id' => true,
               'fieldName' => 'id',
               'type' => 'integer'
            ));

            $metadata->mapField(array(
               'fieldName' => 'username',
               'type' => 'string'
            ));
        }
    }

ClassMetadataBuilder // 类元数据构建器
--------------------------------------------

To ease the use of the ClassMetadata API (which is very raw) there is a ``ClassMetadataBuilder`` that you can use.

为了缓解 ClassMetadata API（它非常的原始<raw>）的使用，你可以使用 ``ClassMetadataBuilder``。

.. code-block:: php

    <?php
    namespace Entities;

    use Doctrine\ORM\Mapping\ClassMetadata;
    use Doctrine\ORM\Mapping\Builder\ClassMetadataBuilder;

    class User
    {
        // ...

        public static function loadMetadata(ClassMetadata $metadata)
        {
            $builder = new ClassMetadataBuilder($metadata);
            $builder->createField('id', 'integer')->isPrimaryKey()->generatedValue()->build();
            $builder->addField('username', 'string');
        }
    }

The API of the ClassMetadataBuilder has the following methods with a fluent interface:

ClassMetadataBuilder 的 API 拥有以下具有流畅接口的方法：

-   ``addField($name, $type, array $mapping)``
-   ``setMappedSuperclass()``
-   ``setReadOnly()``
-   ``setCustomRepositoryClass($className)``
-   ``setTable($name)``
-   ``addIndex(array $columns, $indexName)``
-   ``addUniqueConstraint(array $columns, $constraintName)``
-   ``addNamedQuery($name, $dqlQuery)``
-   ``setJoinedTableInheritance()``
-   ``setSingleTableInheritance()``
-   ``setDiscriminatorColumn($name, $type = 'string', $length = 255)``
-   ``addDiscriminatorMapClass($name, $class)``
-   ``setChangeTrackingPolicyDeferredExplicit()``
-   ``setChangeTrackingPolicyNotify()``
-   ``addLifecycleEvent($methodName, $event)``
-   ``addManyToOne($name, $targetEntity, $inversedBy = null)``
-   ``addInverseOneToOne($name, $targetEntity, $mappedBy)``
-   ``addOwningOneToOne($name, $targetEntity, $inversedBy = null)``
-   ``addOwningManyToMany($name, $targetEntity, $inversedBy = null)``
-   ``addInverseManyToMany($name, $targetEntity, $mappedBy)``
-   ``addOneToMany($name, $targetEntity, $mappedBy)``

It also has several methods that create builders (which are necessary for advanced mappings):

它仅拥有几个创建构建器（对于高级的映射所需）的方法：

-   ``createField($name, $type)`` returns a ``FieldBuilder`` instance
-   ``createManyToOne($name, $targetEntity)`` returns an ``AssociationBuilder`` instance
-   ``createOneToOne($name, $targetEntity)`` returns an ``AssociationBuilder`` instance
-   ``createManyToMany($name, $targetEntity)`` returns an ``ManyToManyAssociationBuilder`` instance
-   ``createOneToMany($name, $targetEntity)`` returns an ``OneToManyAssociationBuilder`` instance

ClassMetadataInfo API // 类元数据信息 API
-----------------------------------------------

The ``ClassMetadataInfo`` class is the base data object for storing
the mapping metadata for a single entity. It contains all the
getters and setters you need populate and retrieve information for
an entity.

``ClassMetadataInfo`` 类是为单个实体存储映射元数据的基础数据对象。它包含为实体填充和取回信息的所有获取器（getter）和设置器（setter）。

General Setters // 通用设置器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``setTableName($tableName)``
-  ``setPrimaryTable(array $primaryTableDefinition)``
-  ``setCustomRepositoryClass($repositoryClassName)``
-  ``setIdGeneratorType($generatorType)``
-  ``setIdGenerator($generator)``
-  ``setSequenceGeneratorDefinition(array $definition)``
-  ``setChangeTrackingPolicy($policy)``
-  ``setIdentifier(array $identifier)``

Inheritance Setters // 继承设置器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``setInheritanceType($type)``
-  ``setSubclasses(array $subclasses)``
-  ``setParentClasses(array $classNames)``
-  ``setDiscriminatorColumn($columnDef)``
-  ``setDiscriminatorMap(array $map)``

Field Mapping Setters // 字段映射设置器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``mapField(array $mapping)``
-  ``mapOneToOne(array $mapping)``
-  ``mapOneToMany(array $mapping)``
-  ``mapManyToOne(array $mapping)``
-  ``mapManyToMany(array $mapping)``

Lifecycle Callback Setters // 生命周期回调设置器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``addLifecycleCallback($callback, $event)``
-  ``setLifecycleCallbacks(array $callbacks)``

Versioning Setters // 版本设置器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``setVersionMapping(array &$mapping)``
-  ``setVersioned($bool)``
-  ``setVersionField()``

General Getters // 通用获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``getTableName()``
-  ``getSchemaName()``
-  ``getTemporaryIdTableName()``

Identifier Getters // 标识符获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``getIdentifierColumnNames()``
-  ``usesIdGenerator()``
-  ``isIdentifier($fieldName)``
-  ``isIdGeneratorIdentity()``
-  ``isIdGeneratorSequence()``
-  ``isIdGeneratorTable()``
-  ``isIdentifierNatural()``
-  ``getIdentifierFieldNames()``
-  ``getSingleIdentifierFieldName()``
-  ``getSingleIdentifierColumnName()``

Inheritance Getters // 继承获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``isInheritanceTypeNone()``
-  ``isInheritanceTypeJoined()``
-  ``isInheritanceTypeSingleTable()``
-  ``isInheritanceTypeTablePerClass()``
-  ``isInheritedField($fieldName)``
-  ``isInheritedAssociation($fieldName)``

Change Tracking Getters // 变更跟踪获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``isChangeTrackingDeferredExplicit()``
-  ``isChangeTrackingDeferredImplicit()``
-  ``isChangeTrackingNotify()``

Field & Association Getters // 字段&关联获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``isUniqueField($fieldName)``
-  ``isNullable($fieldName)``
-  ``getColumnName($fieldName)``
-  ``getFieldMapping($fieldName)``
-  ``getAssociationMapping($fieldName)``
-  ``getAssociationMappings()``
-  ``getFieldName($columnName)``
-  ``hasField($fieldName)``
-  ``getColumnNames(array $fieldNames = null)``
-  ``getTypeOfField($fieldName)``
-  ``getTypeOfColumn($columnName)``
-  ``hasAssociation($fieldName)``
-  ``isSingleValuedAssociation($fieldName)``
-  ``isCollectionValuedAssociation($fieldName)``

Lifecycle Callback Getters // 生命周期回调获取器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~


-  ``hasLifecycleCallbacks($lifecycleEvent)``
-  ``getLifecycleCallbacks($event)``

ClassMetadata API // 类元数据 API
---------------------------------------

The ``ClassMetadata`` class extends ``ClassMetadataInfo`` and adds
the runtime functionality required by Doctrine. It adds a few extra
methods related to runtime reflection for working with the entities
themselves.

``ClassMetadata`` 类扩展了 ``ClassMetadataInfo`` 并通过 Doctrine 添加了
需要的运行时功能。它添加了一些为与实体自身一起工作的运行时反射相关的额外方法：

-  ``getReflectionClass()``
-  ``getReflectionProperties()``
-  ``getReflectionProperty($name)``
-  ``getSingleIdReflectionProperty()``
-  ``getIdentifierValues($entity)``
-  ``setIdentifierValues($entity, $id)``
-  ``setFieldValue($entity, $field, $value)``
-  ``getFieldValue($entity, $field)``


