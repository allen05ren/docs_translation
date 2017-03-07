Association Updates: Owning Side and Inverse Side // 关联更新：Owning 侧和 Inverse 侧
===========================================================================================

When mapping bidirectional associations it is important to
understand the concept of the owning and inverse sides. The
following general rules apply:

当映射双向关联时，重要的是理解 owning 侧和 inverse 侧的概念。适用以下常规规则：

-  Relationships may be bidirectional or unidirectional.
-  关联可能是单向的或双向的。
-  A bidirectional relationship has both an owning side and an inverse side
-  双向的关联拥有 owning 和 inverse 两侧。
-  A unidirectional relationship only has an owning side.
-  单向的关联仅拥有 owning 侧。
-  Doctrine will **only** check the owning side of an association for changes.
-  Doctrine 将**仅**检查关联的 owning 侧的变更。

Bidirectional Associations // 双向的关联
----------------------------------------------

The following rules apply to **bidirectional** associations:

**双向的** 关联适用于以下规则：

- The inverse side has to use the ``mappedBy`` attribute of the OneToOne,
  OneToMany, or ManyToMany mapping declaration. The mappedBy
  attribute contains the name of the association-field on the owning side.
- inverse 侧必须使用 OneToOne、OneToMany、ManyToMany 映射声明的 ``mappedBy`` 属性。
  ``mappedBy`` 属性包含 owning 侧上关联字段的名称。
- The owning side has to use the ``inversedBy`` attribute of the
  OneToOne, ManyToOne, or ManyToMany mapping declaration. 
  The inversedBy attribute contains the name of the association-field
  on the inverse-side.
- owning 侧必须使用 OneToOne、ManyToOne、ManyToMany 映射声明的 ``inversedBy`` 属性。
  ``inversedBy`` 属性包含 inverse 侧上关联字段的名称。
- ManyToOne is always the owning side of a bidirectional association.
- ManyToOne 总是双向的关联的 owning 侧。
- OneToMany is always the inverse side of a bidirectional association.
- OneToMany 总是双向的关联的 inverse 侧。
- The owning side of a OneToOne association is the entity with the table
  containing the foreign key.
- OneToOne 关联的 owning 侧是带有包含外键的表的实体。
- You can pick the owning side of a many-to-many association yourself.
- 你可以自己选择 many-to-many 关联的 owning 侧。

Important concepts // 重要概念
------------------------------------

**Doctrine will only check the owning side of an association for changes.**

**Doctrine 将仅检查关联的 owning 侧的变更。**

To fully understand this, remember how bidirectional associations
are maintained in the object world. There are 2 references on each
side of the association and these 2 references both represent the
same association but can change independently of one another. Of
course, in a correct application the semantics of the bidirectional
association are properly maintained by the application developer
(that's his responsibility). Doctrine needs to know which of these
two in-memory references is the one that should be persisted and
which not. This is what the owning/inverse concept is mainly used
for.

为了完全理解它，记住双向的关联在对象的世界中如何被维护。有2个引用在关联的每一侧且这2个引用都代表同一个关联
但是可以独立于彼此修改。当然，在一个正确的应用程序中双向的关联的含义是由应用程序开发者正确地被维护（这是开发者的责任）。
Doctrine 需要知道两个在内存中的引用哪个是应该被持久和哪一个是不应该被持久。这是 owning/inverse 概念的
主要用途。

**Changes made only to the inverse side of an association are ignored. Make sure to update both sides of a bidirectional association (or at least the owning side, from Doctrine's point of view)**

**仅关联的 inverse 侧所做的变更被忽略。确保双向的关联两侧都更新（或至少 owning 侧，从 Doctrine 的角度来看）**

The owning side of a bidirectional association is the side Doctrine
"looks at" when determining the state of the association, and
consequently whether there is anything to do to update the
association in the database.

双向的关联的 owning 侧是确定关联的状态从而是否有该关联任何的更新需在数据库中更新时 Doctrine 关注的那一侧。

.. note::

    "Owning side" and "inverse side" are technical concepts of
    the ORM technology, not concepts of your domain model. What you
    consider as the owning side in your domain model can be different
    from what the owning side is for Doctrine. These are unrelated.

    “owning 侧”和“inverse 侧”是 ORM 技术的技术上的概念，而不是你的领域模型的概念。
    在领域模型中，你考虑什么作为 owning 侧可以不同于 Doctrine。这是不相关的。

