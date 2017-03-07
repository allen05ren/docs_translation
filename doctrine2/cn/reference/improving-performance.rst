Improving Performance // 增强性能
======================================

Bytecode Cache // 字节码缓存
---------------------------------

It is highly recommended to make use of a bytecode cache like APC.
A bytecode cache removes the need for parsing PHP code on every
request and can greatly improve performance.

强烈建议使用像APC这样的字节码（bytecode）缓存。字节码缓存消除了对每个请求解析 PHP 代码的需要，并可以大大增强性能。

    "If you care about performance and don't use a bytecode
    cache then you don't really care about performance. Please get one
    and start using it."

    “如果你关心性能但不使用字节码缓存，那么你实际上并不关心性能。请开始使用它吧。”

    *Stas Malyshev, Core Contributor to PHP and Zend Employee*

    *Stas Malyshev, PHP 核心贡献者 和 Zend 雇员*

Metadata and Query caches // 元数据与查询缓存
-------------------------------------------------

As already mentioned earlier in the chapter about configuring
Doctrine, it is strongly discouraged to use Doctrine without a
Metadata and Query cache (preferably with APC or Memcache as the
cache driver). Operating Doctrine without these caches means
Doctrine will need to load your mapping information on every single
request and has to parse each DQL query on every single request.
This is a waste of resources.

正如之前在关于 Doctrine 配置的章节中已经提及的，强烈地避免没有元数据和查询缓存
（最好使用 APC 或 Memcache 作为缓存驱动）使用 Doctrine。没有这些缓存操作
Doctrine 意味着 Doctrine 将需要在每个请求加载你的映射信息并且必须在每个请求
解析每个 DQL 查询。这是资源的浪费。

Alternative Query Result Formats // 可选择的查询结果格式
-----------------------------------------------------------

Make effective use of the available alternative query result
formats like nested array graphs or pure scalar results, especially
in scenarios where data is loaded for read-only purposes.

有效使用可选择的查询结果格式，如嵌套的数组图或纯标量结果，尤其在为只读目的加载数据
的情况。

Read-Only Entities // 只读实体
-----------------------------------

Starting with Doctrine 2.1 you can mark entities as read only (See metadata mapping
references for details). This means that the entity marked as read only is never considered
for updates, which means when you call flush on the EntityManager these entities are skipped
even if properties changed. Read-Only allows to persist new entities of a kind and remove existing
ones, they are just not considered for updates.

自 Doctrine 2.1开始，你可以标记实体为只读（详情请查看元数据映射参考）。这意味着标记为只读的实体永远不考虑更新，那意味着当
你在 EntityManager 上调用 flush 时即使属性变更了这些实体也会被跳过。只读允许持久化某种新实体和移除现有的，它们只是不考虑
更新。

Extra-Lazy Collections  // Extra-Lazy 集合
------------------------------------------------

If entities hold references to large collections you will get performance and memory problems initializing them.
To solve this issue you can use the EXTRA_LAZY fetch-mode feature for collections. See the :doc:`tutorial <../tutorials/extra-lazy-associations>`
for more information on how this fetch mode works.

如果实体保存了大集合的引用，初始化它们你将得到性能和内存问题。为解决此问题你可以为集合
使用 EXTRA_LAZY 取回模式功能。查看 :doc:`教程 <../tutorials/extra-lazy-associations>` 获得此取回模式如何工作的更多信息。

Temporarily change fetch mode in DQL // 在 DQL 中临时地变更取回模式
----------------------------------------------------------------------

See :ref:`Doctrine Query Language chapter <dql-temporarily-change-fetch-mode>`

查看：:ref:`Doctrine 查询语言章节 <dql-temporarily-change-fetch-mode>`

Apply Best Practices // 应用最佳实践
------------------------------------------

A lot of the points mentioned in the Best Practices chapter will
also positively affect the performance of Doctrine.

在最佳实践章节中已经提及了许多的点将同样正面地影响 Doctrine 的性能。

Change Tracking policies // 变更跟踪策略
---------------------------------------------

See: :doc:`Change Tracking Policies <reference/change-tracking-policies>`

查看：:doc:`变更跟踪策略 <reference/change-tracking-policies>`
