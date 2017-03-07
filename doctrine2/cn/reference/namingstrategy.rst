Implementing a NamingStrategy // 实现一个命名策略
======================================================

.. versionadded:: 2.3

Using a naming strategy you can provide rules for generating database identifiers,
column or table names when the column or table name is not given. This feature helps
reduce the verbosity of the mapping document, eliminating repetitive noise (eg: ``TABLE_``).

使用命名策略,你可以在未给定列或表的名称时提供规则以生成数据库标识符、列或表的名称。此功能帮助减少冗长的映射文档，
消除重复性噪音（如 ``TABLE_``）。

Configuring a naming strategy // 配置一个命名策略
------------------------------------------------------

The default strategy used by Doctrine is quite minimal.

Doctrine 使用的默认策略非常小。

By default the ``Doctrine\ORM\Mapping\DefaultNamingStrategy``
uses the simple class name and the attribute names to generate tables and columns.

默认，``Doctrine\ORM\Mapping\DefaultNamingStrategy`` 使用简单的类名和属性名以生成表和列。

You can specify a different strategy by calling ``Doctrine\ORM\Configuration#setNamingStrategy()``:

你可以通过调用 ``Doctrine\ORM\Configuration#setNamingStrategy()`` 指定一个不同的策略：

.. code-block:: php

    <?php
    $namingStrategy = new MyNamingStrategy();
    $configuration()->setNamingStrategy($namingStrategy);

Underscore naming strategy // 下划线命名策略
--------------------------------------------------

``\Doctrine\ORM\Mapping\UnderscoreNamingStrategy`` is a built-in strategy.

``\Doctrine\ORM\Mapping\UnderscoreNamingStrategy`` 是内建的策略。

.. code-block:: php

    <?php
    $namingStrategy = new \Doctrine\ORM\Mapping\UnderscoreNamingStrategy(CASE_UPPER);
    $configuration()->setNamingStrategy($namingStrategy);

For SomeEntityName the strategy will generate the table SOME_ENTITY_NAME with the
``CASE_UPPER`` option, or some_entity_name with the ``CASE_LOWER`` option.

对于 SomeEntityName 使用 ``CASE_UPPER`` 选项此策略将生成表 SOME_ENTITY_NAME 或者使用
``CASE_LOWER`` 选项则生成 some_entity_name。

Naming strategy interface // 命名策略接口
-----------------------------------------------
The interface ``Doctrine\ORM\Mapping\NamingStrategy`` allows you to specify
a naming strategy for database tables and columns.

接口 ``Doctrine\ORM\Mapping\NamingStrategy`` 允许你为数据库表和列指定一个命名策略。

.. code-block:: php

    <?php
    /**
     * Return a table name for an entity class
     * 为实体类返回表名称
     *
     * @param string $className The fully-qualified class name
     *                          完全限定类名
     * @return string A table name
     *                表名称
     */
    function classToTableName($className);

    /**
     * Return a column name for a property
     * 为属性返回列名
     *
     * @param string $propertyName A property
     *                             属性
     * @return string A column name
     *                列名
     */
    function propertyToColumnName($propertyName);

    /**
     * Return the default reference column name
     * 返回默认的引用列名
     *
     * @return string A column name
     *                列名
     */
    function referenceColumnName();

    /**
     * Return a join column name for a property
     * 为属性返回联结列的名称
     *
     * @param string $propertyName A property
     *                             属性
     * @return string A join column name
     *                联结列名
     */
    function joinColumnName($propertyName, $className = null);

    /**
     * Return a join table name
     * 返回联结表名
     *
     * @param string $sourceEntity The source entity
     *                             源实体
     * @param string $targetEntity The target entity
     *                             目标实体
     * @param string $propertyName A property
     *                             属性
     * @return string A join table name
     *                联结表的名称
     */
    function joinTableName($sourceEntity, $targetEntity, $propertyName = null);

    /**
     * Return the foreign key column name for the given parameters
     * 为给定参数返回外键列的名称
     *
     * @param string $entityName A entity
     *                           实体
     * @param string $referencedColumnName A property
     *                                     属性
     * @return string A join column name
     *                联结列的名称
     */
    function joinKeyColumnName($entityName, $referencedColumnName = null);

Implementing a naming strategy // 实现一个命名策略
--------------------------------------------------------

If you have database naming standards, like all table names should be prefixed
by the application prefix, all column names should be upper case, you can easily
achieve such standards by implementing a naming strategy.

如果你由数据库命名标准，类似所有表名应该添加应用程序的前缀作为前缀，所有的列名应该使用大写，通过实现
一个命名策略你可以容易地完成这样的标准。

You need to create a class which implements ``Doctrine\ORM\Mapping\NamingStrategy``.

你需要创建一个类，它实现了 ``Doctrine\ORM\Mapping\NamingStrategy``。

.. code-block:: php

    <?php
    class MyAppNamingStrategy implements NamingStrategy
    {
        public function classToTableName($className)
        {
            return 'MyApp_' . substr($className, strrpos($className, '\\') + 1);
        }
        public function propertyToColumnName($propertyName)
        {
            return $propertyName;
        }
        public function referenceColumnName()
        {
            return 'id';
        }
        public function joinColumnName($propertyName, $className = null)
        {
            return $propertyName . '_' . $this->referenceColumnName();
        }
        public function joinTableName($sourceEntity, $targetEntity, $propertyName = null)
        {
            return strtolower($this->classToTableName($sourceEntity) . '_' .
                    $this->classToTableName($targetEntity));
        }
        public function joinKeyColumnName($entityName, $referencedColumnName = null)
        {
            return strtolower($this->classToTableName($entityName) . '_' .
                    ($referencedColumnName ?: $this->referenceColumnName()));
        }
    }
