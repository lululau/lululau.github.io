---
layout: post
title:  "ActiveRecord 的乐观锁字段需要将默认值设置为 0"
date:   2013-11-26 11:09:00
categories: 
  - Ruby
  - Rails
---

在使用乐观锁时，如果乐观锁字段（默认为 `lock_version`）的默认值为 `NULL`，那么这条记录被加载到 Rails 中后，将被初始化为 `0`。当下一次调用 `save` 方法更新这条记录时，Rails 会使用这样的 SQL 语句来查询所更新记录是否已经被更新过：

```sql
UPDATE "clients" SET "name" = 'xxx', "lock_version" = 1 WHERE ("clients"."id" = 1 AND "clients"."lock_version" = 0)
```

因为该记录的 `lock_version` 字段在数据库中的值为默认的 `NULL`，因此必然会查询失败，抛出 `ctiveRecord::StaleObjectError` 异常。

为了解决这个问题，需要在创建库表或者添加 `lock_version` 字段时，指定其默认是为 `0` :

```ruby
class AddLockVersionToClients < ActiveRecord::Migration
  def change
    add_column :clients, :lock_version, :integer, :default => 0
  end
end
```

另外修改一个 Model 的乐观锁字段：
```ruby
class Client < ActiveRecord::Base
  self.locking_column = :lock
end
```