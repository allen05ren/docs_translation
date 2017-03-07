Filters // 过滤器
======================

.. versionadded:: 2.2

Doctrine 2.2 features a filter system that allows the developer to add SQL to
the conditional clauses of queries, regardless the place where the SQL is
generated (e.g. from a DQL query, or by loading associated entities).

Doctrine 2.2 的功能一个过滤器系统，它允许开发人员添加 SQL 到查询的条件子句，而不管 SQL 生成的
位置（如，从一个 DQL 查询或通过加载关联的实体）。

The filter functionality works on SQL level. Whether a SQL query is generated
in a Persister, during lazy loading, in extra lazy collections or from DQL.
Each time the system iterates over all the enabled filters, adding a new SQL
part as a filter returns.

过滤器功能在 SQL 级别上工作。不管 SQL 查询是在一个持久器中、懒加载期间、更懒的集合或来自 DQL
被生成。每次该系统基于所有启用的过滤器迭代，添加新的 SQL 部分作为过滤器的返回。

By adding SQL to the conditional clauses of queries, the filter system filters
out rows belonging to the entities at the level of the SQL result set. This
means that the filtered entities are never hydrated (which can be expensive).

通过添加 SQL 到查询的条件子句，过滤器系统在 SQL 结果集的层级上过滤掉所属实体的行。这意味着，
过滤的实体永不水合（那是“昂贵的”）。

Example filter class // 示例过滤器类
-----------------------------------------

Throughout this document the example ``MyLocaleFilter`` class will be used to
illustrate how the filter feature works. A filter class must extend the base
``Doctrine\ORM\Query\Filter\SQLFilter`` class and implement the ``addFilterConstraint``
method. The method receives the ``ClassMetadata`` of the filtered entity and the
table alias of the SQL table of the entity.

遍及本文档的那个示例类 ``MyLocaleFilter`` 将被用于解释过滤器功能如何工作。过滤器类必须扩展基类
``Doctrine\ORM\Query\Filter\SQLFilter`` 并实现 ``addFilterConstraint`` 方法。
此方法接收过滤的实体的 ``ClassMetadata`` 和该实体的 SQL 表的别名。

.. note::

    In the case of joined or single table inheritance, you always get passed the ClassMetadata of the
    inheritance root. This is necessary to avoid edge cases that would break the SQL when applying the filters.

    联结的或单一表继承的情况，你始终获得传递的继承根的 ClassMetadata。当应用过滤器时，这是必须的
    以避免可能中断 SQL 的边缘情况。

Parameters for the query should be set on the filter object by
``SQLFilter#setParameter()``. Only parameters set via this function can be used
in filters.  The ``SQLFilter#getParameter()`` function takes care of the
proper quoting of parameters.

查询的参数应该通过 ``SQLFilter#setParameter()`` 设置在过滤器对象上。通过此功能设置参数仅可以被用于
过滤器中。``SQLFilter#getParameter()`` 函数负责正确的 quoting 参数。

.. code-block:: php

    <?php
    namespace Example;
    use Doctrine\ORM\Mapping\ClassMetaData,
        Doctrine\ORM\Query\Filter\SQLFilter;

    class MyLocaleFilter extends SQLFilter
    {
        public function addFilterConstraint(ClassMetadata $targetEntity, $targetTableAlias)
        {
            // Check if the entity implements the LocalAware interface
            if (!$targetEntity->reflClass->implementsInterface('LocaleAware')) {
                return "";
            }

            return $targetTableAlias.'.locale = ' . $this->getParameter('locale'); // getParameter applies quoting automatically
        }
    }


Configuration // 配置
----------------------------

Filter classes are added to the configuration as following:

过滤器类被添加到配置，如下：

.. code-block:: php

    <?php
    $config->addFilter("locale", "\Doctrine\Tests\ORM\Functional\MyLocaleFilter");


The ``Configuration#addFilter()`` method takes a name for the filter and the name of the
class responsible for the actual filtering.

``Configuration#addFilter()`` 方法接受过滤器的名称和负责实际过滤的类的名称。

Disabling/Enabling Filters and Setting Parameters // 禁用/启用过滤器和设置参数
--------------------------------------------------------------------------------

Filters can be disabled and enabled via the ``FilterCollection`` which is
stored in the ``EntityManager``. The ``FilterCollection#enable($name)`` method
will retrieve the filter object. You can set the filter parameters on that
object.

过滤器可以通过存储在 ``EntityManager`` 中的 ``FilterCollection`` 禁用和启用。
``FilterCollection#enable($name)`` 方法接收一个过滤器对象。你可以在此对象上设置过滤器参数。

.. code-block:: php

    <?php
    $filter = $em->getFilters()->enable("locale");
    $filter->setParameter('locale', 'en');

    // Disable it
    $filter = $em->getFilters()->disable("locale");

.. warning::
    Disabling and enabling filters has no effect on managed entities. If you
    want to refresh or reload an object after having modified a filter or the
    FilterCollection, then you should clear the EntityManager and re-fetch your
    entities, having the new rules for filtering applied.

    禁用和启用过滤器在 managed 的实体上没有影响。如果你希望修改一个过滤器或 FilterCollection
    之后 refresh 或 reload 一个实体，那么你应该清除（clear）EntityManager 并 re-fetch 你的
    实体，为过滤器应用新的规则。
