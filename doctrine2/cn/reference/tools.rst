Tools // 工具
====================

Doctrine Console // Doctrine 控制台
-------------------------------------------

The Doctrine Console is a Command Line Interface tool for simplifying common
administration tasks during the development of a project that uses Doctrine 2.

Doctrine 控制台是一个命令行接口工具，用于在开发使用 Doctrine 2的项目期间简化通用管理任务。

Take a look at the :doc:`Installation and Configuration <configuration>`
chapter for more information how to setup the console command.

看一看 :doc:`安装与配置 <configuration>` 章节以获得更多如何设置控制台命令的信息。

Display Help Information // 显示帮助信息
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Type ``php vendor/bin/doctrine`` on the command line and you should see an
overview of the available commands or use the --help flag to get
information on the available commands. If you want to know more
about the use of generate entities for example, you can call:

在命令行上输入 ``php vendor/bin/doctrine``，你应该可以看到一个可用命令的概览或者使用 --help
标识获得可用命令上的信息。如果你希望知道有关使用生成实体的例子，你可以调用：

.. code-block:: php

    $> php vendor/bin/doctrine orm:generate-entities --help


Configuration // 配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Whenever the ``doctrine`` command line tool is invoked, it can
access all Commands that were registered by developer. There is no
auto-detection mechanism at work. The Doctrine binary
already registers all the commands that currently ship with
Doctrine DBAL and ORM. If you want to use additional commands you
have to register them yourself.

每当 ``doctrine`` 命令行工具被调用，它可以访问由开发者注册的所有命令。执行上没有自动侦测机制。
Doctrine 二进制已经注册当前随 Doctrine DBAL 和 ORM 附带的所有命令。如果你希望使用额外命令
你必须自己注册它们。

All the commands of the Doctrine Console require access to the ``EntityManager``
or ``DBAL`` Connection. You have to inject them into the console application
using so called Helper-Sets. This requires either the ``db``
or the ``em`` helpers to be defined in order to work correctly.

Doctrine 控制台的所有命令都需要访问 ``EntityManager`` 或 ``DBAL`` 连接。你必须使用所谓的
辅助工具集将它们注入控制台应用程序中。这需要定义 ``db`` 或 ``em`` 辅助工具才能正常工作。

Whenever you invoke the Doctrine binary the current folder is searched for a
``cli-config.php`` file. This file contains the project specific configuration:

每当你调用 Doctrine 二进制，会在当前文件夹搜索 ``cli-config.php`` 文件。此文件包含项目详细配置：

.. code-block:: php

    <?php
    $helperSet = new \Symfony\Component\Console\Helper\HelperSet(array(
        'db' => new \Doctrine\DBAL\Tools\Console\Helper\ConnectionHelper($conn)
    ));
    $cli->setHelperSet($helperSet);

When dealing with the ORM package, the EntityManagerHelper is
required:

当处理 ORM 包时，需要 EntityManagerHelper：

.. code-block:: php

    <?php
    $helperSet = new \Symfony\Component\Console\Helper\HelperSet(array(
        'em' => new \Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper($em)
    ));
    $cli->setHelperSet($helperSet);

The HelperSet instance has to be generated in a separate file (i.e.
``cli-config.php``) that contains typical Doctrine bootstrap code
and predefines the needed HelperSet attributes mentioned above. A
sample ``cli-config.php`` file looks as follows:

HelperSet 实例必须在单独的文件（如，``cli-config.php``）生成，该文件包含典型的 Doctrine 启动
代码和预定义上述提及的需要的 HelperSet 属性。一个简单的 ``cli-config.php`` 文件看起来如下：

.. code-block:: php

    <?php
    // cli-config.php
    require_once 'my_bootstrap.php';

    // Any way to access the EntityManager from  your application
    $em = GetMyEntityManager();
    
    $helperSet = new \Symfony\Component\Console\Helper\HelperSet(array(
        'db' => new \Doctrine\DBAL\Tools\Console\Helper\ConnectionHelper($em->getConnection()),
        'em' => new \Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper($em)
    ));

It is important to define a correct HelperSet that Doctrine binary
script will ultimately use. The Doctrine Binary will automatically
find the first instance of HelperSet in the global variable
namespace and use this.

重要的是定义一个正确的 HelperSet，Doctrine 二进制脚本终将使用。Doctrine 二进制将自动地在全局变量空间查找第一个
HelperSet 实例并使用它。

.. note:: 

    You have to adjust this snippet for your specific application or framework
    and use their facilities to access the Doctrine EntityManager and
    Connection Resources.

    你必须为你的特定应用程序或框架调整此代码段并适用它们的设施以访问 Doctrine EntityManager及连接资源。

Command Overview // 命令概览
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following Commands are currently available:

当前可用的命令如下：

-  ``help`` Displays help for a command (?)
-  ``help`` 显示命令的帮助（？）。
-  ``list`` Lists commands
-  ``list`` 列出命令。
-  ``dbal:import`` Import SQL file(s) directly to Database.
-  ``dbal:import`` 直接地导入 SQL 文件到数据库。
-  ``dbal:run-sql`` Executes arbitrary SQL directly from the
   command line.
-  ``dbal:run-sql`` 直接地从命令和执行任意的 SQL。
-  ``orm:clear-cache:metadata`` Clear all metadata cache of the
   various cache drivers.
-  ``orm:clear-cache:metadata`` 清除各种缓存驱动的所有元数据缓存。
-  ``orm:clear-cache:query`` Clear all query cache of the various
   cache drivers.
-  ``orm:clear-cache:query`` 清除各种缓存驱动的所有查询缓存。
-  ``orm:clear-cache:result`` Clear result cache of the various
   cache drivers.
-  ``orm:clear-cache:result`` 清除各种缓存驱动的结果缓存。
-  ``orm:convert-d1-schema`` Converts Doctrine 1.X schema into a
   Doctrine 2.X schema.
-  ``orm:convert-d1-schema`` 转换 Doctrine 1.X 数据库（schema）到 Doctrine 2.X 中。
-  ``orm:convert-mapping`` Convert mapping information between
   supported formats.
-  ``orm:convert-mapping`` 在支持的格式之间转换映射信息。
-  ``orm:ensure-production-settings`` Verify that Doctrine is
   properly configured for a production environment.
-  ``orm:ensure-production-settings`` 为生产环境验证 Doctrine 恰当配置。
-  ``orm:generate-entities`` Generate entity classes and method
   stubs from your mapping information.
-  ``orm:generate-entities`` 从你的映射信息生成实体类和方法桩件（stubs）。
-  ``orm:generate-proxies`` Generates proxy classes for entity
   classes.
-  ``orm:generate-proxies`` 为实体类生成代理类。
-  ``orm:generate-repositories`` Generate repository classes from
   your mapping information.
-  ``orm:generate-repositories`` 从你的映射信息生成仓库（repository）类。
-  ``orm:run-dql`` Executes arbitrary DQL directly from the command
   line.
-  ``orm:run-dql`` 直接地从命令行执行任意 DQL。
-  ``orm:schema-tool:create`` Processes the schema and either
   create it directly on EntityManager Storage Connection or generate
   the SQL output.
-  ``orm:schema-tool:create`` 处理数据库（schema）并直接地在 EntityManager
   存储连接上创建它或导出生成的 SQL。
-  ``orm:schema-tool:drop`` Processes the schema and either drop
   the database schema of EntityManager Storage Connection or generate
   the SQL output.
-  ``orm:schema-tool:drop`` 处理数据库（schema）并删除（drop）EntityManager
   存储连接的数据库 schema或导出生成的 SQL。
-  ``orm:schema-tool:update`` Processes the schema and either
   update the database schema of EntityManager Storage Connection or
   generate the SQL output.
-  ``orm:schema-tool:update`` 处理数据库（schema）并更新（update）EntityManager
   存储连接的数据库 schema或导出生成的 SQL。

For these commands are also available aliases:

这些命令的可用别名：

-  ``orm:convert:d1-schema`` is alias for ``orm:convert-d1-schema``.
-  ``orm:convert:d1-schema`` 是 ``orm:convert-d1-schema`` 的别名。
-  ``orm:convert:mapping`` is alias for ``orm:convert-mapping``.
-  ``orm:convert:mapping`` 是 ``orm:convert-mapping`` 的别名。
-  ``orm:generate:entities`` is alias for ``orm:generate-entities``.
-  ``orm:generate:entities`` 是 ``orm:generate-entities`` 的别名。
-  ``orm:generate:proxies`` is alias for ``orm:generate-proxies``.
-  ``orm:generate:proxies`` 是 ``orm:generate-proxies`` 的别名。
-  ``orm:generate:repositories`` is alias for ``orm:generate-repositories``.
-  ``orm:generate:repositories`` 是 ``orm:generate-repositories`` 的别名。

.. note::

    Console also supports auto completion, for example, instead of
    ``orm:clear-cache:query`` you can use just ``o:c:q``.

    控制台也支持自动完成，例如，替换 ``orm:clear-cache:query`` 你可以仅使用
    ``o:c:q``。

Database Schema Generation // 数据库 Schema 生成
-------------------------------------------------------

.. note::

    SchemaTool can do harm to your database. It will drop or alter
    tables, indexes, sequences and such. Please use this tool with
    caution in development and not on a production server. It is meant
    for helping you develop your Database Schema, but NOT with
    migrating schema from A to B in production. A safe approach would
    be generating the SQL on development server and saving it into SQL
    Migration files that are executed manually on the production
    server.

    SchemaTool 能够损害你的数据库。它将删除或修改表、索引、序列等等。请在开发中小心使用
    此工具且不在生成服务器上使用。它的用意是帮助你开发你的数据库 Schema，但是不是在生产中
    用于从 A 迁移数据库（schema）到 B。一个安全的方法是在开发服务器上生成 SQL 并保存到
    SQL 迁移文件中，在生产服务器上手动执行迁移文件。

    SchemaTool assumes your Doctrine Project uses the given database on
    its own. Update and Drop commands will mess with other tables if
    they are not related to the current project that is using Doctrine.
    Please be careful!

    SchemaTool 假设你的 Doctrine 项目在其自身上使用给定数据库。如果它们与当前使用
    Doctrine 的项目不相关的话，更新和删除命令将弄乱其他表。请小心！

To generate your database schema from your Doctrine mapping files
you can use the ``SchemaTool`` class or the ``schema-tool`` Console
Command.

为了从你的 Doctrine 映射文件生成你的数据库 schema，你可以使用 ``SchemaTool``类或
``schema-tool`` 控制台命令。

When using the SchemaTool class directly, create your schema using
the ``createSchema()`` method. First create an instance of the
``SchemaTool`` and pass it an instance of the ``EntityManager``
that you want to use to create the schema. This method receives an
array of ``ClassMetadataInfo`` instances.

当直接地使用 SchemaTool 类，使用 ``createSchema()`` 方法创建你的数据库（schema）。
首先创建一个 ``SchemaTool`` 的实例并传递给你希望使用它创建数据库（schema）的
``EntityManager`` 实例。此方法接受一个 ``ClassMetadataInfo`` 实例的数组。

.. code-block:: php

    <?php
    $tool = new \Doctrine\ORM\Tools\SchemaTool($em);
    $classes = array(
      $em->getClassMetadata('Entities\User'),
      $em->getClassMetadata('Entities\Profile')
    );
    $tool->createSchema($classes);

To drop the schema you can use the ``dropSchema()`` method.

为了删除数据库（schema）你可以使用 ``dropSchema()`` 方法。

.. code-block:: php

    <?php
    $tool->dropSchema($classes);

This drops all the tables that are currently used by your metadata
model. When you are changing your metadata a lot during development
you might want to drop the complete database instead of only the
tables of the current model to clean up with orphaned tables.

此方法删除所有当前你的元数据模型使用的表。当你在开发期间变更了许多元数据，你可能希望
删除完整的数据库而不是仅删除当前模型的表以清除孤立的表。

.. code-block:: php

    <?php
    $tool->dropSchema($classes, \Doctrine\ORM\Tools\SchemaTool::DROP_DATABASE);

You can also use database introspection to update your schema
easily with the ``updateSchema()`` method. It will compare your
existing database schema to the passed array of
``ClassMetdataInfo`` instances.

你也可以使用数据库内省（introspection）以更新你的数据库（schema），简单地使用 ``updateSchema()``
方法。它会将你现有的数据库 schema与传递的 ``ClassMetdataInfo`` 实例数组进行比较。

.. code-block:: php

    <?php
    $tool->updateSchema($classes);

If you want to use this functionality from the command line you can
use the ``schema-tool`` command.

如果你希望从命令行使用此功能你可以使用 ``schema-tool`` 命令。

To create the schema use the ``create`` command:

使用 ``create`` 创建数据库（schema）：

.. code-block:: php

    $ php doctrine orm:schema-tool:create

To drop the schema use the ``drop`` command:

使用 ``drop`` 删除数据库（schema）：

.. code-block:: php

    $ php doctrine orm:schema-tool:drop

If you want to drop and then recreate the schema then use both
options:

如果你希望删除并之后重新创建此数据库，那么使用两个选项：

.. code-block:: php

    $ php doctrine orm:schema-tool:drop
    $ php doctrine orm:schema-tool:create

As you would think, if you want to update your schema use the
``update`` command:

正如你可能想到的，如果你希望更新你的数据库（schema），使用 ``update`` 命令：

.. code-block:: php

    $ php doctrine orm:schema-tool:update

All of the above commands also accept a ``--dump-sql`` option that
will output the SQL for the ran operation.

所有上面的命令也接受一个 ``--dump-sql`` 选项，它将为执行的操作导出 SQL。

.. code-block:: php

    $ php doctrine orm:schema-tool:create --dump-sql

Before using the orm:schema-tool commands, remember to configure
your cli-config.php properly.

使用 orm:schema-tool 命令之前，记住正确地配置你的 cli-config.php。

.. note::

    When using the Annotation Mapping Driver you have to either setup
    your autoloader in the cli-config.php correctly to find all the
    entities, or you can use the second argument of the
    ``EntityManagerHelper`` to specify all the paths of your entities
    (or mapping files), i.e.
    ``new \Doctrine\ORM\Tools\Console\Helper\EntityManagerHelper($em, $mappingPaths);``

    当使用注释映射驱动时，你必须在  cli-config.php 中正确地配置你的自动加载器以查找
    所有的实体，或你可以使用 ``EntityManagerHelper`` 的第二个参数以指定你的实体（
    映射文件）的所有部分，等等。

Entity Generation // 实体生成
------------------------------------

Generate entity classes and method stubs from your mapping information.

从你的映射信息生成实体类和方法桩件。

.. code-block:: php

    $ php doctrine orm:generate-entities
    $ php doctrine orm:generate-entities --update-entities
    $ php doctrine orm:generate-entities --regenerate-entities

This command is not suited for constant usage. It is a little helper and does
not support all the mapping edge cases very well. You still have to put work
in your entities after using this command.

此命令不适合常量使用。它是一个小辅助工具，且不能很好地支持所有映射边缘情况。你仍然必须在使用此命令后
将精力放在你的实体中。

It is possible to use the EntityGenerator on code that you have already written. It will
not be lost. The EntityGenerator will only append new code to your
file and will not delete the old code. However this approach may still be prone
to error and we suggest you use code repositories such as GIT or SVN to make
backups of your code.

在你已经写的代码上使用 EntityGenerator 是可能的。它将不会丢失。EntityGenerator 将仅追加新的代码
到你的文件并且将不会删除旧的代码。但是此方法可能仍然容易出错并且我们建议你使用代码仓库，如 GIT 或 SVN，
备份你的代码。

It makes sense to generate the entity code if you are using entities as Data
Access Objects only and don't put much additional logic on them. If you are
however putting much more logic on the entities you should refrain from using
the entity-generator and code your entities manually.

如果你仅使用实体作为数据访问对象并不放置太多额外逻辑在上面，生成实体代码是有意义的。如果你无论如何要放置
很多逻辑在该实体上你应该克制使用此实体生成器并手动编写你的实体。

.. note::

    Even if you specified Inheritance options in your
    XML or YAML Mapping files the generator cannot generate the base and
    child classes for you correctly, because it doesn't know which
    class is supposed to extend which. You have to adjust the entity
    code manually for inheritance to work!

    即使你在你的 XML 或 YAML 映射文件中指定了继承选项，生成器不能生成为你正确地生成基类及子类，
    因为它不知道哪个类应该扩展哪个。你必须为继承手动调整该实体代码以便正常工作。


Convert Mapping Information // 转换映射信息
--------------------------------------------------

Convert mapping information between supported formats.

在支持的格式之间转换映射信息。

This is an **execute one-time** command. It should not be necessary for
you to call this method multiple times, especially when using the ``--from-database``
flag.

这是一个 **执行一次性（execute one-time）** 的命令。它并不需要调用此方法多次，尤其当使用
``--from-database`` 标识时。

Converting an existing database schema into mapping files only solves about 70-80%
of the necessary mapping information. Additionally the detection from an existing
database cannot detect inverse associations, inheritance types,
entities with foreign keys as primary keys and many of the
semantical operations on associations such as cascade.

转换现有数据库 schema 到映射文件仅解决了大约70-80%的必要映射信息。此外，来自现有数据库的检测不能检测
inverse 关联、继承类型、使用外键作为主键的实体和许多在关联上的语义化操作，如级联。

.. note::

    There is no need to convert YAML or XML mapping files to annotations
    every time you make changes. All mapping drivers are first class citizens
    in Doctrine 2 and can be used as runtime mapping for the ORM. See the
    docs on XML and YAML Mapping for an example how to register this metadata
    drivers as primary mapping source.

    每次你进行变更时不需要转换 YAML 或 XML 映射文件到注释。在 Doctrine 2中，所有映射驱动都是
    一等公民（first class citizens）且可以用作 ORM 的运行时映射。有关如何注册你的元数据驱动
    作为主映射源的例子，请查看 XML 和 YAML 映射的文档。 

To convert some mapping information between the various supported
formats you can use the ``ClassMetadataExporter`` to get exporter
instances for the different formats:

为了在各种受支持的格式之间转换一些映射信息，你可以使用 ``ClassMetadataExporter`` 获取不同的格式
导出器实例：

.. code-block:: php

    <?php
    $cme = new \Doctrine\ORM\Tools\Export\ClassMetadataExporter();

Once you have a instance you can use it to get an exporter. For
example, the yml exporter:

一旦你有了一个实例，你可以使用它获取一个导出器。例如，yml 导出器：

.. code-block:: php

    <?php
    $exporter = $cme->getExporter('yml', '/path/to/export/yml');

Now you can export some ``ClassMetadata`` instances:

现在你可以导出一些 ``ClassMetadata`` 实例：

.. code-block:: php

    <?php
    $classes = array(
      $em->getClassMetadata('Entities\User'),
      $em->getClassMetadata('Entities\Profile')
    );
    $exporter->setMetadata($classes);
    $exporter->export();

This functionality is also available from the command line to
convert your loaded mapping information to another format. The
``orm:convert-mapping`` command accepts two arguments, the type to
convert to and the path to generate it:

此功能也可用于从命令行转换你的已加载映射信息到另一个格式。``orm:convert-mapping`` 命令
接受两个参数，需要转换的类型和用于生成它的路径：

.. code-block:: php

    $ php doctrine orm:convert-mapping xml /path/to/mapping-path-converted-to-xml

Reverse Engineering // 逆向工程
---------------------------------------

You can use the ``DatabaseDriver`` to reverse engineer a database
to an array of ``ClassMetadataInfo`` instances and generate YAML,
XML, etc. from them.

你可以使用 ``DatabaseDriver`` 逆向工程一个数据库到一个 ``ClassMetadataInfo`` 实例的数组并从它们生成
YAML、XML等等。

.. note::

    Reverse Engineering is a **one-time** process that can get you started with a project.
    Converting an existing database schema into mapping files only detects about 70-80%
    of the necessary mapping information. Additionally the detection from an existing
    database cannot detect inverse associations, inheritance types,
    entities with foreign keys as primary keys and many of the
    semantical operations on associations such as cascade.

    逆向工程是一个 **一次性（one-time）** 的过程，可以让你开始一个项目。转换现有数据库 schema 到映射文件仅
    检测了大约70-80%的必要映射信息。此外，来自现有数据库的检测不能检测 inverse 关联、继承类型、使用外键作为
    主键的实体和许多在关联上的语义化操作，如级联。

First you need to retrieve the metadata instances with the
``DatabaseDriver``:

首先你需要使用 ``DatabaseDriver`` 取回元数据实例：

.. code-block:: php

    <?php
    $em->getConfiguration()->setMetadataDriverImpl(
        new \Doctrine\ORM\Mapping\Driver\DatabaseDriver(
            $em->getConnection()->getSchemaManager()
        )
    );
    
    $cmf = new \Doctrine\ORM\Tools\DisconnectedClassMetadataFactory();
    $cmf->setEntityManager($em);
    $metadata = $cmf->getAllMetadata();

Now you can get an exporter instance and export the loaded metadata
to yml:

现在你可以获得一个导出器实例并导出已加载的元数据到 yml：

.. code-block:: php

    <?php
    $cme = new \Doctrine\ORM\Tools\Export\ClassMetadataExporter();
    $exporter = $cme->getExporter('yml', '/path/to/export/yml');
    $exporter->setMetadata($metadata);
    $exporter->export();

You can also reverse engineer a database using the
``orm:convert-mapping`` command:

你也可以使用 ``orm:convert-mapping`` 命令逆向工程一个数据库：

.. code-block:: php

    $ php doctrine orm:convert-mapping --from-database yml /path/to/mapping-path-converted-to-yml

.. note::

    Reverse Engineering is not always working perfectly
    depending on special cases. It will only detect Many-To-One
    relations (even if they are One-To-One) and will try to create
    entities from Many-To-Many tables. It also has problems with naming
    of foreign keys that have multiple column names. Any Reverse
    Engineered Database-Schema needs considerable manual work to become
    a useful domain model.

    根据特殊情况，逆向工程并不总是完美地工作。它将仅侦测 Many-To-One 关联（即使它们是
    One-To-One）并且将尝试从 Many-To-Many 表创建实体。使用多列名的命名外键，它也有
    问题。任何逆向工程数据库都需要考虑手动工作以变成一个有用的领域模型。


Runtime vs Development Mapping Validation // 运行时 vs 开发时的映射验证
----------------------------------------------------------------------------

For performance reasons Doctrine 2 has to skip some of the
necessary validation of metadata mappings. You have to execute
this validation in your development workflow to verify the
associations are correctly defined.

由于性能的原因 Doctrine 2 必须跳过一些必要的元数据映射的验证。你必须在你的开发工作流中执行此验证以
检验关联正确地定义了。

You can either use the Doctrine Command Line Tool:

你可以使用 Doctrine 命令行工具：

.. code-block:: php

    doctrine orm:validate-schema

Or you can trigger the validation manually:

或你可以手动地触发该验证：

.. code-block:: php

    <?php
    use Doctrine\ORM\Tools\SchemaValidator;

    $validator = new SchemaValidator($entityManager);
    $errors = $validator->validateMapping();

    if (count($errors) > 0) {
        // Lots of errors!
        echo implode("\n\n", $errors);
    }

If the mapping is invalid the errors array contains a positive
number of elements with error messages.

如果映射无效的，错误数组将包含带错误消息的正数个元素。

.. warning::

    One mapping option that is not validated is the use of the referenced column name.
    It has to point to the equivalent primary key otherwise Doctrine will not work.

    一个没有验证的映射选项是使用引用的列名称。它必须指向等价的主键，否则 Doctrine 将不会工作。

.. note::

    One common error is to use a backlash in front of the
    fully-qualified class-name. Whenever a FQCN is represented inside a
    string (such as in your mapping definitions) you have to drop the
    prefix backslash. PHP does this with ``get_class()`` or Reflection
    methods for backwards compatibility reasons.

    一个常见的错误时在完全限定的类名前使用反斜杠。每当一个 FQCN 在一个字符串内（如，在你的映射定义中）
    被表示，你必须删除前导反斜杠。为了向后兼容的原因，PHP 使用 `get_class()`` 或反射方法来这样做。

Adding own commands // 添加自己的命令
-------------------------------------------

You can also add your own commands on-top of the Doctrine supported
tools if you are using a manually built console script.

你也可以基于 Doctrine 支持的工具添加自己的命令，如果你正在使用一个手动构建的控制台脚本的话。

To include a new command on Doctrine Console, you need to do modify the
``doctrine.php`` file a little:

为了在 Doctrine 控制台包含一个新的命令，你需要对 ``doctrine.php`` 文件做一点点修改：

.. code-block:: php

    <?php
    // doctrine.php
    use Symfony\Component\Console\Application;

    // as before ...

    // replace the ConsoleRunner::run() statement with:
    $cli = new Application('Doctrine Command Line Interface', \Doctrine\ORM\Version::VERSION);
    $cli->setCatchExceptions(true);
    $cli->setHelperSet($helperSet);

    // Register All Doctrine Commands
    ConsoleRunner::addCommands($cli);

    // Register your own command
    $cli->addCommand(new \MyProject\Tools\Console\Commands\MyCustomCommand);

    // Runs console application
    $cli->run();

Additionally, include multiple commands (and overriding previously
defined ones) is possible through the command:

另外，通过命令可以包含多个命令（和覆盖之前定义的）：

.. code-block:: php

    <?php

    $cli->addCommands(array(
        new \MyProject\Tools\Console\Commands\MyCustomCommand(),
        new \MyProject\Tools\Console\Commands\SomethingCommand(),
        new \MyProject\Tools\Console\Commands\AnotherCommand(),
        new \MyProject\Tools\Console\Commands\OneMoreCommand(),
    ));


Re-use console application // 重用控制台应用程序
-----------------------------------------------------

You are also able to retrieve and re-use the default console application.
Just call ``ConsoleRunner::createApplication(...)`` with an appropriate
HelperSet, like it is described in the configuration section.

你也能够取回和重用默认的控制台应用程序。只要使用一个合适的 HelperSet 调用
``ConsoleRunner::createApplication(...)``，类似的它在配置部分被描述。

.. code-block:: php

    <?php

    // Retrieve default console application
    $cli = ConsoleRunner::createApplication($helperSet);

    // Runs console application
    $cli->run();

