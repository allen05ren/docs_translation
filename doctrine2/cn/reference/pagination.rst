Pagination // 分页
=========================

.. versionadded:: 2.2

Starting with version 2.2 Doctrine ships with a Paginator for DQL queries. It
has a very simple API and implements the SPL interfaces ``Countable`` and
``IteratorAggregate``.

自 2.3 版本起，Doctrine 附带了一个用于 DQL 查询分页器。它拥有非常简单的 API 且实现了 SPL 的
``Countable`` 和 ``IteratorAggregate`` 接口。

.. code-block:: php

    <?php
    use Doctrine\ORM\Tools\Pagination\Paginator;

    $dql = "SELECT p, c FROM BlogPost p JOIN p.comments c";
    $query = $entityManager->createQuery($dql)
                           ->setFirstResult(0)
                           ->setMaxResults(100);

    $paginator = new Paginator($query, $fetchJoinCollection = true);

    $c = count($paginator);
    foreach ($paginator as $post) {
        echo $post->getHeadline() . "\n";
    }

Paginating Doctrine queries is not as simple as you might think in the
beginning. If you have complex fetch-join scenarios with one-to-many or
many-to-many associations using the "default" LIMIT functionality of database
vendors is not sufficient to get the correct results.

分页 Doctrine 查询不是你在开始时可能想象的那么简单。如果你有复杂 fetch-join 的场景与 one-to-many 或
many-to-many 关联，使用数据库供应商“默认”的LIMIT 功能不足以获得正确的结果

By default the pagination extension does the following steps to compute the
correct result:

默认，分页扩展执行以下步骤计算正确的结果：

- Perform a Count query using `DISTINCT` keyword.
- 使用 `DISTINCT` 关键字执行一个计算查询。
- Perform a Limit Subquery with `DISTINCT` to find all ids of the entity in from on the current page.
- 使用 `DISTINCT` 执行一个 LIMIT 子查询 以查找在来自当前页面中的实体的所有 ids。
- Perform a WHERE IN query to get all results for the current page.
- 执行一个 WHERE IN 查询以获得所有当前页面的结果。

This behavior is only necessary if you actually fetch join a to-many
collection. You can disable this behavior by setting the
``$fetchJoinCollection`` flag to ``false``; in that case only 2 instead of the 3 queries
described are executed. We hope to automate the detection for this in
the future.

如果你实际上 fetch join 一个 to-many 集合，才需要此行为。你可以通过设置 ``$fetchJoinCollection`` 标识为
``false`` 来禁用此行为，在此情况下仅执行描述的2个查询而不是3个查询。我们希望在未来自动检测。
