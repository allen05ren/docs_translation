Metadata Drivers // 元数据驱动
====================================

The heart of an object relational mapper is the mapping information
that glues everything together. It instructs the EntityManager how
it should behave when dealing with the different entities.

对象关联映射的核心是将一切黏合在一起的映射信息。它指示 EntityManager 当处理不同的
实体时应该如何表现。

Core Metadata Drivers // 核心元数据驱动
---------------------------------------------

Doctrine provides a few different ways for you to specify your
metadata:

Doctrine 为你提供了几种不同的方式以指定你的元数据：

-  **XML files** (XmlDriver)
-  **XML 文件** (XmlDriver)
-  **Class DocBlock Annotations** (AnnotationDriver)
-  **类文档块注释** (AnnotationDriver)
-  **YAML files** (YamlDriver)
-  **YAML 文件** (YamlDriver)
-  **PHP Code in files or static functions** (PhpDriver)
-  **在文件或静态函数中的 PHP 代码** (PhpDriver)

Something important to note about the above drivers is they are all
an intermediate step to the same end result. The mapping
information is populated to ``Doctrine\ORM\Mapping\ClassMetadata``
instances. So in the end, Doctrine only ever has to work with the
API of the ``ClassMetadata`` class to get mapping information for
an entity.

关于上述驱动的一些重要注意事项是它们其不都是到同样最终结果的中间步骤。映射信息被填充到
``Doctrine\ORM\Mapping\ClassMetadata`` 实例。所以最后，Doctrine 仅必须使用
``ClassMetadata`` 类的 API 以获得实体的映射信息。
.. note::

    The populated ``ClassMetadata`` instances are also cached
    so in a production environment the parsing and populating only ever
    happens once. You can configure the metadata cache implementation
    using the ``setMetadataCacheImpl()`` method on the
    ``Doctrine\ORM\Configuration`` class:

    已填充的 ``ClassMetadata`` 实例也被缓存，所以在生产环境中解析和填充仅发生一次。
    你可以在 ``Doctrine\ORM\Configuration`` 类上使用 ``setMetadataCacheImpl()``
    配置元数据缓存实现：

    .. code-block:: php

        <?php
        $em->getConfiguration()->setMetadataCacheImpl(new ApcCache());


If you want to use one of the included core metadata drivers you
just need to configure it. All the drivers are in the
``Doctrine\ORM\Mapping\Driver`` namespace:

如果你希望使用所包含的核心元数据驱动之一，你仅需要配置它。所有的驱动都在 ``Doctrine\ORM\Mapping\Driver``
命名空间中：

.. code-block:: php

    <?php
    $driver = new \Doctrine\ORM\Mapping\Driver\XmlDriver('/path/to/mapping/files');
    $em->getConfiguration()->setMetadataDriverImpl($driver);

Implementing Metadata Drivers // 实现元数据驱动
-----------------------------------------------------

In addition to the included metadata drivers you can very easily
implement your own. All you need to do is define a class which
implements the ``Driver`` interface:

除了包含的元数据驱动，你可以非常容易地实现自己的。所有你需要做的是定义一个实现了类 ``Driver`` 接口的类：

.. code-block:: php

    <?php
    namespace Doctrine\ORM\Mapping\Driver;
    
    use Doctrine\ORM\Mapping\ClassMetadataInfo;
    
    interface Driver
    {
        /**
         * Loads the metadata for the specified class into the provided container.
         * 
         * @param string $className
         * @param ClassMetadataInfo $metadata
         */
        function loadMetadataForClass($className, ClassMetadataInfo $metadata);
    
        /**
         * Gets the names of all mapped classes known to this driver.
         * 
         * @return array The names of all mapped classes known to this driver.
         */
        function getAllClassNames(); 
    
        /**
         * Whether the class with the specified name should have its metadata loaded.
         * This is only the case if it is either mapped as an Entity or a
         * MappedSuperclass.
         *
         * @param string $className
         * @return boolean
         */
        function isTransient($className);
    }

If you want to write a metadata driver to parse information from
some file format we've made your life a little easier by providing
the ``AbstractFileDriver`` implementation for you to extend from:

如果你希望写一个元数据驱动以从一些文件格式解析信息，我们为你提供了 ``AbstractFileDriver``
实现让你的生活更容易一些，扩展它：

.. code-block:: php

    <?php
    class MyMetadataDriver extends AbstractFileDriver
    {
        /**
         * {@inheritdoc}
         */
        protected $_fileExtension = '.dcm.ext';
    
        /**
         * {@inheritdoc}
         */
        public function loadMetadataForClass($className, ClassMetadataInfo $metadata)
        {
            $data = $this->_loadMappingFile($file);
    
            // populate ClassMetadataInfo instance from $data
        }
    
        /**
         * {@inheritdoc}
         */
        protected function _loadMappingFile($file)
        {
            // parse contents of $file and return php data structure
        }
    }

.. note::

    When using the ``AbstractFileDriver`` it requires that you
    only have one entity defined per file and the file named after the
    class described inside where namespace separators are replaced by
    periods. So if you have an entity named ``Entities\User`` and you
    wanted to write a mapping file for your driver above you would need
    to name the file ``Entities.User.dcm.ext`` for it to be
    recognized.

    当使用 ``AbstractFileDriver`` 时，它仅需要你有一个实体定义的一个文件并且
    该文件以类描述命名，内部的命名空间分隔符用句点替换。所以如果你有一个实体命名为
    ``Entities\User`` 并且你希望为上面的你的驱动写一个映射文件，你需要将文件命名为
    ``Entities.User.dcm.ext`` 以便识别它。

Now you can use your ``MyMetadataDriver`` implementation by setting
it with the ``setMetadataDriverImpl()`` method:

现在你可以通过 ``setMetadataDriverImpl()`` 方法设置并使用你的 ``MyMetadataDriver`` 实现：

.. code-block:: php

    <?php
    $driver = new MyMetadataDriver('/path/to/mapping/files');
    $em->getConfiguration()->setMetadataDriverImpl($driver);

ClassMetadata // 类元数据
--------------------------------

The last piece you need to know and understand about metadata in
Doctrine 2 is the API of the ``ClassMetadata`` classes. You need to
be familiar with them in order to implement your own drivers but
more importantly to retrieve mapping information for a certain
entity when needed.

在 Doctrine 2中有关元数据的最后一块你需要知道并理解的是 ``ClassMetadata`` 类
的 API。你需要熟悉它们以便实现你自己的驱动，但是更重要的当需要时为某个实体取回
映射信息。

You have all the methods you need to manually specify the mapping
information instead of using some mapping file to populate it from.
The base ``ClassMetadataInfo`` class is responsible for only data
storage and is not meant for runtime use. It does not require that
the class actually exists yet so it is useful for describing some
entity before it exists and using that information to generate for
example the entities themselves. The class ``ClassMetadata``
extends ``ClassMetadataInfo`` and adds some functionality required
for runtime usage and requires that the PHP class is present and
can be autoloaded.

你拥有手动指定映射信息所需要的所有的方法，而不是使用一些映射文件填充它。其类
``ClassMetadataInfo`` 仅负责数据存储且不是用于运行时使用。它不需要该类实际存在，
所以它是有用的，在类存在之前描述一些实体信息且使用这些信息生成示例实体自身。类
``ClassMetadata`` 扩展了 ``ClassMetadataInfo`` 并为运行时使用添加了一些功能需求
而且需要该 PHP 类是存在的并可以被自动加载。

You can read more about the API of the ``ClassMetadata`` classes in
the PHP Mapping chapter.

你可以在 PHP 映射章节中了解到更多有关 ``ClassMetadata`` 的 API 的更多细节。

Getting ClassMetadata Instances // 获得类元数据实例
---------------------------------------------------------

If you want to get the ``ClassMetadata`` instance for an entity in
your project to programmatically use some mapping information to
generate some HTML or something similar you can retrieve it through
the ``ClassMetadataFactory``:

如果你希望在你的项目中为一个实体获得 ``ClassMetadata`` 实例以编程的方式使用一些
映射信息以便生成一些 HTML 或 类似的事情，你可以通过 ``ClassMetadataFactory``
取回它：

.. code-block:: php

    <?php
    $cmf = $em->getMetadataFactory();
    $class = $cmf->getMetadataFor('MyEntityName');

Now you can learn about the entity and use the data stored in the
``ClassMetadata`` instance to get all mapped fields for example and
iterate over them:

现在你可以学习有关该实体并使用存储在 ``ClassMetadata`` 实例中的数据以获得所有映射字段并
迭代它们：

.. code-block:: php

    <?php
    foreach ($class->fieldMappings as $fieldMapping) {
        echo $fieldMapping['fieldName'] . "\n";
    }


