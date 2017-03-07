YAML Mapping // YAML 映射
=================================

The YAML mapping driver enables you to provide the ORM metadata in
form of YAML documents.

YAML 映射驱动使你能够以 YAML 文档的形式提供 ORM 元数据。

The YAML mapping document of a class is loaded on-demand the first
time it is requested and subsequently stored in the metadata cache.
In order to work, this requires certain conventions:

一个类的 YAML 映射文档是第一次被请求时按需加载，随后存储在元数据缓存中。为了正常工作，
这需要某些惯例：

-  Each entity/mapped superclass must get its own dedicated YAML
   mapping document.
-  每个实体/映射超类必须获得其专门的 YAML 映射文档。
-  The name of the mapping document must consist of the fully
   qualified name of the class, where namespace separators are
   replaced by dots (.).
-  映射文档的名称必须由类的完全限定名称组成，其中的命名空间分隔符由点（.）替换。
-  All mapping documents should get the extension ".dcm.yml" to
   identify it as a Doctrine mapping file. This is more of a
   convention and you are not forced to do this. You can change the
   file extension easily enough.
-  所有的映射文档应该获得扩展名“.dcm.yml”以标识它作为 Doctrine 映射文件。这更多是
   一个惯例，并不强制你这样做。你可以很轻易地修改此文件扩展名。

.. code-block:: php

    <?php
    $driver->setFileExtension('.yml');

It is recommended to put all YAML mapping documents in a single
folder but you can spread the documents over several folders if you
want to. In order to tell the YamlDriver where to look for your
mapping documents, supply an array of paths as the first argument
of the constructor, like this:

推荐在单个文件夹中放置所有 YAML 映射文档，但是你可以分布文档在几个文件夹，如果你希望的话。
为了告诉 YamlDriver 在哪儿查找你的映射文档，提供一个路径的数组作为构造器的第一个参数，像这样：

.. code-block:: php

    <?php
    use Doctrine\ORM\Mapping\Driver\YamlDriver;

    // $config instanceof Doctrine\ORM\Configuration
    $driver = new YamlDriver(array('/path/to/files'));
    $config->setMetadataDriverImpl($driver);

Simplified YAML Driver // 简化的 YAML 驱动
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Symfony project sponsored a driver that simplifies usage of the YAML Driver.
The changes between the original driver are:

Symfony 项目赞助了一个驱动，简化了 YAML 驱动的使用。与原始驱动之间的变更是：

- File Extension is .orm.yml
- 文件扩展名是 .orm.yml。
- Filenames are shortened, "MyProject\\Entities\\User" will become User.orm.yml
- 文件名被缩短，“MyProject\\Entities\\User” 将变成 User.orm.yml。
- You can add a global file and add multiple entities in this file.
- 你可以添加一个全局文件并在此文件中添加多个实体。

Configuration of this client works a little bit different:

这个客户端的配置工作有一点点不同：

.. code-block:: php

    <?php
    $namespaces = array(
        '/path/to/files1' => 'MyProject\Entities',
        '/path/to/files2' => 'OtherProject\Entities'
    );
    $driver = new \Doctrine\ORM\Mapping\Driver\SimplifiedYamlDriver($namespaces);
    $driver->setGlobalBasename('global'); // global.orm.yml

Example // 示例
-----------------------

As a quick start, here is a small example document that makes use
of several common elements:

作为一个快速开始，这里是一个小的示例文档，使用几个常见元素：

.. code-block:: yaml

    # Doctrine.Tests.ORM.Mapping.User.dcm.yml
    Doctrine\Tests\ORM\Mapping\User:
      type: entity
      repositoryClass: Doctrine\Tests\ORM\Mapping\UserRepository
      table: cms_users
      schema: schema_name # The schema the table lies in, for platforms that support schemas (Optional, >= 2.5)
      readOnly: true
      indexes:
        name_index:
          columns: [ name ]
      id:
        id:
          type: integer
          generator:
            strategy: AUTO
      fields:
        name:
          type: string
          length: 50
        email:
          type: string
          length: 32
          column: user_email
          unique: true
          options:
            fixed: true
            comment: User's email address
        loginCount:
          type: integer
          column: login_count
          nullable: false
          options:
            unsigned: true
            default: 0
      oneToOne:
        address:
          targetEntity: Address
          joinColumn:
            name: address_id
            referencedColumnName: id
            onDelete: CASCADE
      oneToMany:
        phonenumbers:
          targetEntity: Phonenumber
          mappedBy: user
          cascade: ["persist", "merge"]
      manyToMany:
        groups:
          targetEntity: Group
          joinTable:
            name: cms_users_groups
            joinColumns:
              user_id:
                referencedColumnName: id
            inverseJoinColumns:
              group_id:
                referencedColumnName: id
      lifecycleCallbacks:
        prePersist: [ doStuffOnPrePersist, doOtherStuffOnPrePersistToo ]
        postPersist: [ doStuffOnPostPersist ]

Be aware that class-names specified in the YAML files should be
fully qualified.

请注意，在 YAML 文件中指定的类名应该是完全限定的。

Reference // 参考
~~~~~~~~~~~~~~~~~~~~~~~~

Unique Constraints // 唯一约束
------------------------------------

It is possible to define unique constraints by the following declaration:

通过以下声明定义唯一约束是可能的：

.. code-block:: yaml

    # ECommerceProduct.orm.yml
    ECommerceProduct:
      type: entity
      fields:
        # definition of some fields
      uniqueConstraints:
        search_idx:
          columns: [ name, email ]

