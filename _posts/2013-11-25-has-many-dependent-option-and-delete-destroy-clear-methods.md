---
layout: post
title:  "has_many 的 :dependent 选项和 delete/destroy/clear 方法"
date:   2013-11-25 16:00:00
categories: 
  - Ruby
  - Rails
---

`has_many` 的 `:dependent` 选项有两个方面的作用：

## 1. Controls what happens to the associated objects when their owner is destroyed:

* `:destroy`：使被删除对象的所有关联对象也被 destroy。
* `:delete_all`：直接从数据库中删除所有的关联记录，因此各种 callbacks 都不会调用。
* `:nullify`：直接将数据库中所有的关联记录的对应外键设置为 `NULL`，同样不会调用 callbacks。
* `restrict_with_exception`：删除时若存在任何关联记录，则抛出异常。
* `restrict_with_error`：删除时若存在任何关联记录，则向被删除对象的 errors 数组中追加一个 error。

## 2. `:dependent` 选项的取值影响 `delete` / `clear` 方法的行为：

* `owner.collections.destroy(obj)` 方法的行为不受 `:dependent` 选项的取值的影响，`obj` 所对应的记录会从数据库中删除。

* `owner.collections.delete(obj)` 和 `owner.collections.clear()` 方法的行为都会受 `:dependent` 选项的值的影响：

  - `:dependent` 设为 `:destroy` 时：`delete` 和 `clear` 都会使得相关的关联对象对 destroy。
  - `:dependent` 设为 `:delete_all` 时：`delete` 和 `clear` 会从数据库中直接删除相关的关联记录，不会调用 callbacks。
  - 其他情况：相关的关联对象的外键被设置为 `NULL`。
