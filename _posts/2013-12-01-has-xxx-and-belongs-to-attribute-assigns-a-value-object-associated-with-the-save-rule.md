---
layout: post
title:  "给 belongs_to 和 has_xxx 关联属性赋值时对象的保存规则"
date:   2013-12-01 19:06:00
categories: 
  - Ruby
  - Rails
---

#### 1. When are objects saved when assigning an object to a `belongs_to` association

将一个对象赋值给一个 `belongs_to` 关联属性时，这个对象并不会被自动保存，赋值的左值属性所属的对象也不会被保存。

#### 2. When are objects saved when assigning objects to `has_xxx` association

当将一个或多个对象赋值给一个 `has_one`、`has_many` 或者 `has_and_belongs_to_many` 关联属性时，这个（些）对象将被自动保存（因为需要更新这些对象所对应的数据库记录的 foreign key）。另外任何从该关联关系中剔除的对象也会被自动保存，因为他们的 foreign key 也需要更新。

如果在上述提及这些对象的自动保存过程中，由于 validation 出错而保存失败，那么赋值语句会返回 `false`，并此赋值操作将被取消。

另外，如果父对象（声明 `has_one`、`has_many`、`has_and_belongs_to_many`的那一方）尚未保存（就是说在此对象上调用 `new_record?` 将返回 `true`），那么这些孩子对象也不会被保存，孩子对象将会在父对象被保存时被自动保存。