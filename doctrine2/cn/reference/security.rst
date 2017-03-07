Security // 安全
=======================

The Doctrine library is operating very close to your database and as such needs
to handle and make assumptions about SQL injection vulnerabilities.

Doctrine 库的操作非常接近你的数据并且因此需要处理和假设关于 SQL 注入漏洞。

It is vital that you understand how Doctrine approaches security, because
we cannot protect you from SQL injection.

理解 Doctrine 如何处理安全是必不可少的，因为我们不能避免你免受 SQL 注入。

Please also read the documentation chapter on Security in Doctrine DBAL. This
page only handles Security issues in the ORM.

也请阅读在 Doctrine DBAL 文档中的安全章节。本页面仅处理在 ORM 中的安全问题。

- [DBAL Security Page](https://github.com/doctrine/dbal/blob/master/docs/en/reference/security.rst)
- [DBAL 安全页面](https://github.com/doctrine/dbal/blob/master/docs/en/reference/security.rst)

If you find a Security bug in Doctrine, please report it on Jira and change the
Security Level to "Security Issues". It will be visible to Doctrine Core
developers and you only.

如果你在 Doctrine 中找到一个安全 bug，请在 Jira 上报告它并变更安全等级到“Security Issues”。它将仅对
Doctrine 核心开发者们和你可见。

User input and Doctrine ORM // 用户输入和 Doctrine ORM
-------------------------------------------------------------

The ORM is much better at protecting against SQL injection than the DBAL alone.
You can consider the following APIs to be safe from SQL injection:

针对 SQL 注入，ORM 比单独的 DBAL 有更好的防护。你可以考虑以下 APIs 以免受 SQL 注入的危险：

- ``\Doctrine\ORM\EntityManager#find()`` and ``getReference()``.
- ``\Doctrine\ORM\EntityManager#find()`` 和 ``getReference()``
- All values on Objects inserted and updated through ``Doctrine\ORM\EntityManager#persist()``
- 通过 ``Doctrine\ORM\EntityManager#persist()`` 来完成在对象上的所有值的插入和更新。
- All find methods on ``Doctrine\ORM\EntityRepository``.
- 在 ``Doctrine\ORM\EntityRepository`` 上的所有查找方法。
- User Input set to DQL Queries or QueryBuilder methods through
- 通过这些方法设置用户输入到 DQL 查询或 QueryBuilder
    - ``setParameter()`` or variants
    - ``setParameter()`` 或变体 - 设置参数
    - ``setMaxResults()``
    - ``setMaxResults()`` - 设置最大的结果
    - ``setFirstResult()``
    - ``setFirstResult()`` - 设置第一个结果
- Queries through the Criteria API on ``Doctrine\ORM\PersistentCollection`` and
  ``Doctrine\ORM\EntityRepository``.
- 查询通过在 ``Doctrine\ORM\PersistentCollection`` 和 ``Doctrine\ORM\EntityRepository`` 上的 Criteria API。

You are **NOT** safe from SQL injection when using user input with:

在使用用户输入时，**不能** 避免 SQL 注入：

- Expression API of ``Doctrine\ORM\QueryBuilder``
- ``Doctrine\ORM\QueryBuilder`` 的表达式 API
- Concatenating user input into DQL SELECT, UPDATE or DELETE statements or
  Native SQL.
- 串联用户输入到 DQL SELECT、UPDATE 或 DELETE 语句或者原生 SQL 中。

This means SQL injections can only occur with Doctrine ORM when working with
Query Objects of any kind. The safe rule is to always use prepared statement
parameters for user objects when using a Query object.

这意味着，SQL 注入仅能发生在使用 Doctrine ORM 与任何类型的查询对象一起工作时。当使用查询对象时，
安全法则是对于用户对象始终使用预处理语句参数。

.. warning::

    Insecure code follows, don't copy paste this.

    不安全的代码如下，不要复制粘贴此处。

The following example shows insecure DQL usage:

以下示例展示不安全的 DQL 用法：

.. code-block:: php

    <?php

    // INSECURE
    $dql = "SELECT u
              FROM MyProject\Entity\User u
             WHERE u.status = '" .  $_GET['status'] . "'
         ORDER BY " . $_GET['orderField'] . " ASC";

For Doctrine there is absolutely no way to find out which parts of ``$dql`` are
from user input and which are not, even if we have our own parsing process
this is technically impossible. The correct way is:

对于 Doctrine，没有绝对的办法来找出 ``$dql`` 的哪些部分来自用户输入和哪些部分不是，甚至我们拥有我们
自己的解析过程，这在技术上是不可能的。正确的方法是：

.. code-block:: php

    <?php

    $orderFieldWhitelist = array('email', 'username');
    $orderField = "email";

    if (in_array($_GET['orderField'], $orderFieldWhitelist)) {
        $orderField = $_GET['orderField'];
    }

    $dql = "SELECT u
              FROM MyProject\Entity\User u
             WHERE u.status = ?1
         ORDER BY u." . $orderField . " ASC";

    $query = $entityManager->createQuery($dql);
    $query->setParameter(1, $_GET['status']);


Preventing Mass Assignment Vulnerabilities // 避免 Mass Assignment 漏洞
-------------------------------------------------------------------------------

ORMs are very convenient for CRUD applications and Doctrine is no exception.
However CRUD apps are often vulnerable to mass assignment security problems
when implemented naively.

对于 CRUD 应用程序， ORMS 是非常实用的，Doctrine 也不例外。然而，CRUD 应用程序往往易遭受
mass assignment 安全问题的影响，当天真地执行时。

Doctrine is not vulnerable to this problem out of the box, but you can easily
make your entities vulnerable to mass assignment when you add methods of
the kind ``updateFromArray()`` or ``updateFromJson()`` to them. A vulnerable
entity might look like this:

Doctrine 是开箱即用的不易遭受此问题的影响，但是你可以轻易地令你的实体易遭受 mass assignment 的影响，
当你添加 ``updateFromArray()`` 或 ``updateFromJson()`` 类型的方法给它们时。一个脆弱的实体可能
看起来像这样：

.. code-block:: php

    <?php

    /**
     * @Entity
     */
    class InsecureEntity
    {
        /** @Id @Column(type="integer") @GeneratedValue */
        private $id;
        /** @Column */
        private $email;
        /** @Column(type="boolean") */
        private $isAdmin;

        public function fromArray(array $userInput)
        {
            foreach ($userInput as $key => $value) {
                $this->$key = $value;
            }
        }
    }

Now the possiblity of mass-asignment exists on this entity and can
be exploited by attackers to set the "isAdmin" flag to true on any
object when you pass the whole request data to this method like:

现在此实体存在 mass-asignment 的可能性且可能被攻击者利用以在任何对象上设置“isAdmin”
标识为 true，当你像如下这样传递整个请求的数据到此方法时：

.. code-block:: php

    <?php
    $entity = new InsecureEntity();
    $entity->fromArray($_POST);

    $entityManager->persist($entity);
    $entityManager->flush();

You can spot this problem in this very simple example easily. However
in combination with frameworks and form libraries it might not be
so obvious when this issue arises. Be careful to avoid this
kind of mistake.

你可以在这个非常简单的示例中轻易地看出这个问题。但是，在与框架和表单库的组合中它可能不是那么明显，
当此问题出现时。小心以避免此类型的错误。

How to fix this problem? You should always have a whitelist
of allowed key to set via mass assignment functions.

如何修复此问题？你应该总是拥有一个允许的键的白名单以设置经由 mass assignment 的函数。

.. code-block:: php

    public function fromArray(array $userInput, $allowedFields = array())
    {
        foreach ($userInput as $key => $value) {
            if (in_array($key, $allowedFields)) {
                $this->$key = $value;
            }
        }
    }
