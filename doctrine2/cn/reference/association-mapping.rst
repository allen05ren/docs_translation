Association Mapping // 关联映射
====================================

This chapter explains mapping associations between objects.

本章解释对象之间的映射关联。

Instead of working with foreign keys in your code, you will always work with
references to objects instead and Doctrine will convert those references
to foreign keys internally.

取代在代码中使用外键，你将总是使用对象的引用替代，Doctrine 将内部转换这些引用为外键。

- A reference to a single object is represented by a foreign key.
- 单个对象的引用由一个外键表示。
- A collection of objects is represented by many foreign keys pointing to the object holding the collection
- 对象的集合由指向集合的对象的多个外键表示

This chapter is split into three different sections.

本章分为三个不同的部分

- A list of all the possible association mapping use-cases is given.
- 给出了一个所有可能的关联映射用例的列表。
- :ref:`association_mapping_defaults` are explained that simplify the use-case examples.
- 解释 :ref:`关联映射默认值 <association_mapping_defaults>` ，使用简化的用例。
- :ref:`collections` are introduced that contain entities in associations.
- 介绍 :ref:`集合 <collections>`，包含关联中的实体。

One tip for working with relations is to read the relation from left to right, where the left word refers to the current Entity. For example:

使用关联的一个小技巧是从左至右阅读关联，左边单词是指当前实体。例如：

- OneToMany - One instance of the current Entity has Many instances (references) to the referred Entity.
- OneToMany - 当前实体的一个实例有多个实例（引用）至所提及的实体。
- ManyToOne - Many instances of the current Entity refer to One instance of the referred Entity.
- ManyToOne - 当前实体的许多实例引用所提及实体的一个实例
- OneToOne - One instance of the current Entity refers to One instance of the referred Entity.
- OneToOne - 当前实体的一个实例引用所提及的实体的一个实例。

See below for all the possible relations. 

所有可能的关联请看下面。

To gain a full understanding of associations you should also read about :doc:`owning and
inverse sides of associations <unitofwork-associations>`

更进一步地完整理解关联，你应当也阅读关于 :doc:`关联的 owning 和 inverse 侧 <unitofwork-associations>`。

Many-To-One, Unidirectional // Many-To-One, 单向的
--------------------------------------------------------

A many-to-one association is the most common association between objects.

一个 many-to-one 关联是对象之间最普通的关联。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many Users have One Address.
             * @ManyToOne(targetEntity="Address")
             * @JoinColumn(name="address_id", referencedColumnName="id")
             */
            private $address;
        }

        /** @Entity */
        class Address
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-one field="address" target-entity="Address">
                    <join-column name="address_id" referenced-column-name="id" />
                </many-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToOne:
            address:
              targetEntity: Address
              joinColumn:
                name: address_id
                referencedColumnName: id


.. note::

    The above ``@JoinColumn`` is optional as it would default
    to ``address_id`` and ``id`` anyways. You can omit it and let it
    use the defaults.

    上面的 ``@JoinColumn`` 是可选项，因为它将默认为``address_id`` 和 ``id`` 不管怎样。
    你可以忽略它并让它使用默认值。

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        address_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE Address (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    ALTER TABLE User ADD FOREIGN KEY (address_id) REFERENCES Address(id);

One-To-One, Unidirectional // One-To-One, 单向的
-----------------------------------------------------

Here is an example of a one-to-one association with a ``Product`` entity that
references one ``Shipping`` entity. The ``Shipping`` does not reference back to
the ``Product`` so that the reference is said to be unidirectional, in one
direction only.

这是一个 one-to-one 关联的例子，有一个 ``Product``实体，该实体引用了一个 ``Shipping`` 实体。
``Shipping`` 不能反向引用 ``Product``，所以该引用称作是单向的，仅在一个方向。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Product
        {
            // ...

            /**
             * One Product has One Shipping.
             * @OneToOne(targetEntity="Shipping")
             * @JoinColumn(name="shipping_id", referencedColumnName="id")
             */
            private $shipping;

            // ...
        }

        /** @Entity */
        class Shipping
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipping" target-entity="Shipping">
                    <join-column name="shipping_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipping:
              targetEntity: Shipping
              joinColumn:
                name: shipping_id
                referencedColumnName: id

Note that the @JoinColumn is not really necessary in this example,
as the defaults would be the same.

注意 @JoinColumn 在此例中不是真的需要，因为默认的值和它一样。

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE Product (
        id INT AUTO_INCREMENT NOT NULL,
        shipping_id INT DEFAULT NULL,
        UNIQUE INDEX UNIQ_6FBC94267FE4B2B (shipping_id),
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Shipping (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Product ADD FOREIGN KEY (shipping_id) REFERENCES Shipping(id);

One-To-One, Bidirectional // One-To-One, 双向的
----------------------------------------------------

Here is a one-to-one relationship between a ``Customer`` and a
``Cart``. The ``Cart`` has a reference back to the ``Customer`` so
it is bidirectional.

这是一个 ``Customer`` 和 ``Cart`` 之间的 one-to-one 关联。
``Cart`` 有一个反向引用 ``Customer``，所以它是双向的。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Customer
        {
            // ...

            /**
             * One Customer has One Cart.
             * @OneToOne(targetEntity="Cart", mappedBy="customer")
             */
            private $cart;

            // ...
        }

        /** @Entity */
        class Cart
        {
            // ...

            /**
             * One Cart has One Customer.
             * @OneToOne(targetEntity="Customer", inversedBy="cart")
             * @JoinColumn(name="customer_id", referencedColumnName="id")
             */
            private $customer;

            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Customer">
                <one-to-one field="cart" target-entity="Cart" mapped-by="customer" />
            </entity>
            <entity name="Cart">
                <one-to-one field="customer" target-entity="Customer" inversed-by="cart">
                    <join-column name="customer_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Customer:
          oneToOne:
            cart:
              targetEntity: Cart
              mappedBy: customer
        Cart:
          oneToOne:
            customer:
              targetEntity: Customer
              inversedBy: cart
              joinColumn:
                name: customer_id
                referencedColumnName: id

Note that the @JoinColumn is not really necessary in this example,
as the defaults would be the same.

注意 @JoinColumn 在此例中不是真的需要，因为默认的值和它一样。

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE Cart (
        id INT AUTO_INCREMENT NOT NULL,
        customer_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Customer (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Cart ADD FOREIGN KEY (customer_id) REFERENCES Customer(id);

See how the foreign key is defined on the owning side of the
relation, the table ``Cart``.

观察外键如何被定义在关联的 owning 一侧，表 ``Cart``。

One-To-One, Self-referencing // One-To-One, 自引用
-------------------------------------------------------

You can define a self-referencing one-to-one relationships like
below.

你可以定义一个自引用的 one-to-one 关联像如下。

.. code-block:: php

    <?php
    /** @Entity */
    class Student
    {
        // ...

        /**
         * One Student has One Student.
         * @OneToOne(targetEntity="Student")
         * @JoinColumn(name="mentor_id", referencedColumnName="id")
         */
        private $mentor;

        // ...
    }

Note that the @JoinColumn is not really necessary in this example,
as the defaults would be the same.

注意 @JoinColumn 在此例中不是真的需要，因为默认的值和它一样。

With the generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE Student (
        id INT AUTO_INCREMENT NOT NULL,
        mentor_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Student ADD FOREIGN KEY (mentor_id) REFERENCES Student(id);

One-To-Many, Bidirectional // One-To-Many, 双向的
------------------------------------------------------

A one-to-many association has to be bidirectional, unless you are using an
additional join-table. This is necessary, because of the foreign key
in a one-to-many association being defined on the "many" side. Doctrine
needs a many-to-one association that defines the mapping of this
foreign key.

一个 one-to-many 关联必须是双向的，除非你使用一个额外的 join-table。
这是必须的，因为一个 one-to-many 关联的外键在 “many” 一侧被定义。
Doctrine 需要一个 many-to-one 关联，它定义了该外键的映射。

This bidirectional mapping requires the ``mappedBy`` attribute on the
``OneToMany`` association and the ``inversedBy`` attribute on the ``ManyToOne``
association.

这个双向映射需要在 ``OneToMany`` 关联上的 ``mappedBy`` 属性和在 ``ManyToOne`` 关联上的 ``inversedBy`` 属性。

.. configuration-block::

    .. code-block:: php

        <?php
        use Doctrine\Common\Collections\ArrayCollection;

        /** @Entity */
        class Product
        {
            // ...
            /**
             * One Product has Many Features.
             * @OneToMany(targetEntity="Feature", mappedBy="product")
             */
            private $features;
            // ...

            public function __construct() {
                $this->features = new ArrayCollection();
            }
        }

        /** @Entity */
        class Feature
        {
            // ...
            /**
             * Many Features have One Product.
             * @ManyToOne(targetEntity="Product", inversedBy="features")
             * @JoinColumn(name="product_id", referencedColumnName="id")
             */
            private $product;
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Product">
                <one-to-many field="features" target-entity="Feature" mapped-by="product" />
            </entity>
            <entity name="Feature">
                <many-to-one field="product" target-entity="Product" inversed-by="features">
                    <join-column name="product_id" referenced-column-name="id" />
                </many-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToMany:
            features:
              targetEntity: Feature
              mappedBy: product
        Feature:
          type: entity
          manyToOne:
            product:
              targetEntity: Product
              inversedBy: features
              joinColumn:
                name: product_id
                referencedColumnName: id

Note that the @JoinColumn is not really necessary in this example,
as the defaults would be the same.

注意 @JoinColumn 在此例中不是真的需要，因为默认的值和它一样。

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE Product (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE Feature (
        id INT AUTO_INCREMENT NOT NULL,
        product_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Feature ADD FOREIGN KEY (product_id) REFERENCES Product(id);

One-To-Many, Unidirectional with Join Table // One-To-Many, 具有 Join Table 的单向的
----------------------------------------------------------------------------------------

A unidirectional one-to-many association can be mapped through a
join table. From Doctrine's point of view, it is simply mapped as a
unidirectional many-to-many whereby a unique constraint on one of
the join columns enforces the one-to-many cardinality.

一个单向 one-to-many 关联可以通过联结表（join table）来映射。从 Doctrine 的角度看，
它简单地映射为一个单向的 many-to-many，凭借在 join 列之一上的唯一约束强制该 one-to-many 基数性（cardinality）。

The following example sets up such a unidirectional one-to-many association:

以下的例子配置了这样一个单向 one-to-many 关联：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many User have Many Phonenumbers.
             * @ManyToMany(targetEntity="Phonenumber")
             * @JoinTable(name="users_phonenumbers",
             *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="phonenumber_id", referencedColumnName="id", unique=true)}
             *      )
             */
            private $phonenumbers;

            public function __construct()
            {
                $this->phonenumbers = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

        /** @Entity */
        class Phonenumber
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="phonenumbers" target-entity="Phonenumber">
                    <join-table name="users_phonenumbers">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="phonenumber_id" referenced-column-name="id" unique="true" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            phonenumbers:
              targetEntity: Phonenumber
              joinTable:
                name: users_phonenumbers
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  phonenumber_id:
                    referencedColumnName: id
                    unique: true


Generates the following MySQL Schema:

生成以下 MySQL Schema：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    CREATE TABLE users_phonenumbers (
        user_id INT NOT NULL,
        phonenumber_id INT NOT NULL,
        UNIQUE INDEX users_phonenumbers_phonenumber_id_uniq (phonenumber_id),
        PRIMARY KEY(user_id, phonenumber_id)
    ) ENGINE = InnoDB;

    CREATE TABLE Phonenumber (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;

    ALTER TABLE users_phonenumbers ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE users_phonenumbers ADD FOREIGN KEY (phonenumber_id) REFERENCES Phonenumber(id);

One-To-Many, Self-referencing // One-To-Many, 自引用
---------------------------------------------------------

You can also setup a one-to-many association that is
self-referencing. In this example we setup a hierarchy of
``Category`` objects by creating a self referencing relationship.
This effectively models a hierarchy of categories and from the
database perspective is known as an adjacency list approach.

你也可以配置一个自引用的 one-to-many 关联。在这个例子中我们通过创建一个自引用关联
配置了一个 ``Category`` 对象的层次结构。这有效地塑造了一个分类的层次结构，
从数据库的角度被称为邻接表方法。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class Category
        {
            // ...
            /**
             * One Category has Many Categories.
             * @OneToMany(targetEntity="Category", mappedBy="parent")
             */
            private $children;

            /**
             * Many Categories have One Category.
             * @ManyToOne(targetEntity="Category", inversedBy="children")
             * @JoinColumn(name="parent_id", referencedColumnName="id")
             */
            private $parent;
            // ...

            public function __construct() {
                $this->children = new \Doctrine\Common\Collections\ArrayCollection();
            }
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="Category">
                <one-to-many field="children" target-entity="Category" mapped-by="parent" />
                <many-to-one field="parent" target-entity="Category" inversed-by="children" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Category:
          type: entity
          oneToMany:
            children:
              targetEntity: Category
              mappedBy: parent
          manyToOne:
            parent:
              targetEntity: Category
              inversedBy: children

Note that the @JoinColumn is not really necessary in this example,
as the defaults would be the same.

注意 @JoinColumn 在此例中不是真的需要，因为默认的值和它一样。

Generated MySQL Schema:

生成以下 MySQL Schema：

.. code-block:: sql

    CREATE TABLE Category (
        id INT AUTO_INCREMENT NOT NULL,
        parent_id INT DEFAULT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE Category ADD FOREIGN KEY (parent_id) REFERENCES Category(id);

Many-To-Many, Unidirectional // Many-To-Many，单向的
---------------------------------------------------------

Real many-to-many associations are less common. The following
example shows a unidirectional association between User and Group
entities:

真正的 many-to-many 关联不太常见。
下面这个例子展示一个 User 和 Group 实体之间的单向关联

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group")
             * @JoinTable(name="users_groups",
             *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="group_id", referencedColumnName="id")}
             *      )
             */
            private $groups;

            // ...

            public function __construct() {
                $this->groups = new \Doctrine\Common\Collections\ArrayCollection();
            }
        }

        /** @Entity */
        class Group
        {
            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="groups" target-entity="Group">
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
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

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE users_groups (
        user_id INT NOT NULL,
        group_id INT NOT NULL,
        PRIMARY KEY(user_id, group_id)
    ) ENGINE = InnoDB;
    CREATE TABLE Group (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    ALTER TABLE users_groups ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE users_groups ADD FOREIGN KEY (group_id) REFERENCES Group(id);

.. note::

    Why are many-to-many associations less common? Because
    frequently you want to associate additional attributes with an
    association, in which case you introduce an association class.
    Consequently, the direct many-to-many association disappears and is
    replaced by one-to-many/many-to-one associations between the 3
    participating classes.

    为何 many-to-many 不常见？因为经常你想用一个关联来关联额外属性，这种情况你采用一个关联类。
    因此，直接的 many-to-many 关联不存在了，取而代之的是3个类之间参与的 one-to-many/many-to-one 关联。

Many-To-Many, Bidirectional // Many-To-Many, 双向的
--------------------------------------------------------

Here is a similar many-to-many relationship as above except this
one is bidirectional.

这是一个类似 many-to-many 关联，和上面不同的是它是双向的。

.. configuration-block::

    .. code-block:: php

        <?php
        /** @Entity */
        class User
        {
            // ...

            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group", inversedBy="users")
             * @JoinTable(name="users_groups")
             */
            private $groups;

            public function __construct() {
                $this->groups = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

        /** @Entity */
        class Group
        {
            // ...
            /**
             * Many Groups have Many Users.
             * @ManyToMany(targetEntity="User", mappedBy="groups")
             */
            private $users;

            public function __construct() {
                $this->users = new \Doctrine\Common\Collections\ArrayCollection();
            }

            // ...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity name="User">
                <many-to-many field="groups" inversed-by="users" target-entity="Group">
                    <join-table name="users_groups">
                        <join-columns>
                            <join-column name="user_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column name="group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>

            <entity name="Group">
                <many-to-many field="users" mapped-by="groups" target-entity="User"/>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group
              inversedBy: users
              joinTable:
                name: users_groups
                joinColumns:
                  user_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  group_id:
                    referencedColumnName: id

        Group:
          type: entity
          manyToMany:
            users:
              targetEntity: User
              mappedBy: groups

The MySQL schema is exactly the same as for the Many-To-Many
uni-directional case above.

这个 MySQL Schema 完全地与上面 Many-To-Many 单向的例子一样。

Owning and Inverse Side on a ManyToMany association // ManyToMany 关联上的 Owning and Inverse 侧
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For Many-To-Many associations you can chose which entity is the
owning and which the inverse side. There is a very simple semantic
rule to decide which side is more suitable to be the owning side
from a developers perspective. You only have to ask yourself, which
entity is responsible for the connection management and pick that
as the owning side.

对于 Many-To-Many 关联你可以选择那个实体是 owning 一侧，另一个实体是 inverse 一侧。
从一个开发者的角度，有一个很简单的语义化规则来确定那一侧更为合适作为 owning 一侧。
你只需问自己，那个实体是负责连接管理，那么这一侧就是 owning 一侧。

Take an example of two entities ``Article`` and ``Tag``. Whenever
you want to connect an Article to a Tag and vice-versa, it is
mostly the Article that is responsible for this relation. Whenever
you add a new article, you want to connect it with existing or new
tags. Your create Article form will probably support this notion
and allow to specify the tags directly. This is why you should pick
the Article as owning side, as it makes the code more
understandable:

举个例子有两个实体 ``Article`` 和 ``Tag``。
每当你想要连接 Article 至 Tag，反之亦然，通常 Article 负责此关联。
每当你添加一篇新的 Article，你想要用现有的或新的 tags 连接它。
你创建的 Article 的表单将很可能支持这个功能并允许直接指定 tags。
这就是为何你应该选择 Article 作为 owning 一侧，因为它使代码更容易理解：


.. code-block:: php

    <?php
    class Article
    {
        private $tags;

        public function addTag(Tag $tag)
        {
            $tag->addArticle($this); // synchronously updating inverse side
            $this->tags[] = $tag;
        }
    }

    class Tag
    {
        private $articles;

        public function addArticle(Article $article)
        {
            $this->articles[] = $article;
        }
    }

This allows to group the tag adding on the ``Article`` side of the
association:

这允许在关联的 ``Article`` 一侧上分组 tag 的添加。

.. code-block:: php

    <?php
    $article = new Article();
    $article->addTag($tagA);
    $article->addTag($tagB);

Many-To-Many, Self-referencing // Many-To-Many, 自引用
-----------------------------------------------------------

You can even have a self-referencing many-to-many association. A
common scenario is where a ``User`` has friends and the target
entity of that relationship is a ``User`` so it is self
referencing. In this example it is bidirectional so ``User`` has a
field named ``$friendsWithMe`` and ``$myFriends``.

你甚至可以有一个自引用的 many-to-many 关联。
一个常见的场景，一个 ``User`` 有多个朋友，并且关联的目标实体也是一个 ``User``，所以它是自引用。
在此例中，它是双向的，所以 ``User`` 有一个命名的字段 ``$friendsWithMe`` 及 ``$myFriends``。

.. code-block:: php

    <?php
    /** @Entity */
    class User
    {
        // ...

        /**
         * Many Users have Many Users.
         * @ManyToMany(targetEntity="User", mappedBy="myFriends")
         */
        private $friendsWithMe;

        /**
         * Many Users have many Users.
         * @ManyToMany(targetEntity="User", inversedBy="friendsWithMe")
         * @JoinTable(name="friends",
         *      joinColumns={@JoinColumn(name="user_id", referencedColumnName="id")},
         *      inverseJoinColumns={@JoinColumn(name="friend_user_id", referencedColumnName="id")}
         *      )
         */
        private $myFriends;

        public function __construct() {
            $this->friendsWithMe = new \Doctrine\Common\Collections\ArrayCollection();
            $this->myFriends = new \Doctrine\Common\Collections\ArrayCollection();
        }

        // ...
    }

Generated MySQL Schema:

生成的 MySQL Schema：

.. code-block:: sql

    CREATE TABLE User (
        id INT AUTO_INCREMENT NOT NULL,
        PRIMARY KEY(id)
    ) ENGINE = InnoDB;
    CREATE TABLE friends (
        user_id INT NOT NULL,
        friend_user_id INT NOT NULL,
        PRIMARY KEY(user_id, friend_user_id)
    ) ENGINE = InnoDB;
    ALTER TABLE friends ADD FOREIGN KEY (user_id) REFERENCES User(id);
    ALTER TABLE friends ADD FOREIGN KEY (friend_user_id) REFERENCES User(id);

.. _association_mapping_defaults:

Mapping Defaults // 映射默认值
----------------------------------

The ``@JoinColumn`` and ``@JoinTable`` definitions are usually optional and have
sensible default values. The defaults for a join column in a
one-to-one/many-to-one association is as follows:

``@JoinColumn`` 和 ``@JoinTable`` 定义通常是可选的且有一个合理的默认值。
在 one-to-one/many-to-one 关联中 join 列的默认值如下：

::

    name: "<fieldname>_id"
    referencedColumnName: "id"

As an example, consider this mapping:

作为例子，思考这个映射：

.. configuration-block::

    .. code-block:: php

        <?php
        /** @OneToOne(targetEntity="Shipping") */
        private $shipping;

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipping" target-entity="Shipping" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipping:
              targetEntity: Shipping

This is essentially the same as the following, more verbose,
mapping:

这本质上与下面相同，更详细的映射：

.. configuration-block::

    .. code-block:: php

        <?php
        /**
         * One Product has One Shipping.
         * @OneToOne(targetEntity="Shipping")
         * @JoinColumn(name="shipping_id", referencedColumnName="id")
         */
        private $shipping;

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="Product">
                <one-to-one field="shipping" target-entity="Shipping">
                    <join-column name="shipping_id" referenced-column-name="id" />
                </one-to-one>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        Product:
          type: entity
          oneToOne:
            shipping:
              targetEntity: Shipping
              joinColumn:
                name: shipping_id
                referencedColumnName: id

The @JoinTable definition used for many-to-many mappings has
similar defaults. As an example, consider this mapping:

被用于 many-to-many 映射的 @JoinTable 定义有类似的默认值。
作为例子，思考这个映射：

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            //...
            /** @ManyToMany(targetEntity="Group") */
            private $groups;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="User">
                <many-to-many field="groups" target-entity="Group" />
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group

This is essentially the same as the following, more verbose, mapping:

这本质上与下面相同，更详细的映射：

.. configuration-block::

    .. code-block:: php

        <?php
        class User
        {
            //...
            /**
             * Many Users have Many Groups.
             * @ManyToMany(targetEntity="Group")
             * @JoinTable(name="User_Group",
             *      joinColumns={@JoinColumn(name="User_id", referencedColumnName="id")},
             *      inverseJoinColumns={@JoinColumn(name="Group_id", referencedColumnName="id")}
             *      )
             */
            private $groups;
            //...
        }

    .. code-block:: xml

        <doctrine-mapping>
            <entity class="User">
                <many-to-many field="groups" target-entity="Group">
                    <join-table name="User_Group">
                        <join-columns>
                            <join-column id="User_id" referenced-column-name="id" />
                        </join-columns>
                        <inverse-join-columns>
                            <join-column id="Group_id" referenced-column-name="id" />
                        </inverse-join-columns>
                    </join-table>
                </many-to-many>
            </entity>
        </doctrine-mapping>

    .. code-block:: yaml

        User:
          type: entity
          manyToMany:
            groups:
              targetEntity: Group
              joinTable:
                name: User_Group
                joinColumns:
                  User_id:
                    referencedColumnName: id
                inverseJoinColumns:
                  Group_id:
                    referencedColumnName: id

In that case, the name of the join table defaults to a combination
of the simple, unqualified class names of the participating
classes, separated by an underscore character. The names of the
join columns default to the simple, unqualified class name of the
targeted class followed by "\_id". The referencedColumnName always
defaults to "id", just as in one-to-one or many-to-one mappings.

在那个例子中，join 表名称默认为简单的组合参与类的绝对类名，由下划线字符分隔。
join 列名称默认为简单的默认为目标类的绝对类名后跟随“\_id”。
referencedColumnName 总是默认为“id”，如同在 one-to-one 或 many-to-one 映射中那样。

If you accept these defaults, you can reduce the mapping code to a
minimum.

如果你接受这些默认值，你可以减少映射的代码至最小。

.. _collections:

Collections // 集合
------------------------

Unfortunately, PHP arrays, while being great for many things, are missing
features that make them suitable for lazy loading in the context of an ORM.
This is why in all the examples of many-valued associations in this manual we
will make use of a ``Collection`` interface and its
default implementation ``ArrayCollection`` that are both defined in the
``Doctrine\Common\Collections`` namespace. A collection implements
the PHP interfaces ``ArrayAccess``, ``Traversable`` and ``Countable``.

不幸的是，PHP 数组缺乏一些在 ORM 上下文中使它们适合懒加载的特性，尽管它多数情况下表现良好。
这就是为何本手册中所有多值（many-valued）关联的例子中我们将使用 ``Collection`` 接口及其
默认实现 ``ArrayCollection``，这俩都被定义在 ``Doctrine\Common\Collections`` 命名空间中。
一个集合实现了 PHP 的 ``ArrayAccess``、``Traversable`` 和 ``Countable`` 接口。

.. note::

    The Collection interface and ArrayCollection class,
    like everything else in the Doctrine namespace, are neither part of
    the ORM, nor the DBAL, it is a plain PHP class that has no outside
    dependencies apart from dependencies on PHP itself (and the SPL).
    Therefore using this class in your model and elsewhere
    does not introduce a coupling to the ORM.

    该 Collection 接口和 ArrayCollection 类与其他在 Doctrine 命名空间中的类类似，
    并不是 ORM 或 DBAL 的一部分，它是一个无外部依赖的纯 PHP 类，除了依赖 PHP 本身（和 SPL）。
    因此，在你的模型中或其他地方使用这个类并不会引入与该 ORM 的耦合。

Initializing Collections // 初始化集合
------------------------------------------

You should always initialize the collections of your ``@OneToMany``
and ``@ManyToMany`` associations in the constructor of your entities:

在实体的构造器中，你应当总是初始化你的 ``@OneToMany`` 和 ``@ManyToMany`` 关联的集合：

.. code-block:: php

    <?php
    use Doctrine\Common\Collections\Collection;
    use Doctrine\Common\Collections\ArrayCollection;

    /** @Entity */
    class User
    {
        /**
         * Many Users have Many Groups.
         * @var Collection
         * @ManyToMany(targetEntity="Group")
         */
        private $groups;

        public function __construct()
        {
            $this->groups = new ArrayCollection();
        }

        public function getGroups()
        {
            return $this->groups;
        }
    }

The following code will then work even if the Entity hasn't
been associated with an EntityManager yet:

以下代码甚至于不用与一个 EntityManager 关联也能正常工作：

.. code-block:: php

    <?php
    $group = new Group();
    $user = new User();
    $user->getGroups()->add($group);
