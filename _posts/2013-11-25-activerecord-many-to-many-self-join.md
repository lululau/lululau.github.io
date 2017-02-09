---
layout: post
title:  "ActiveRecord Many-To-Many Self Join"
date:   2013-11-25 00:35:00
categories: 
  - Ruby
  - Rails
---

系统中有一个 `User` 模型，要求用户之间需要存在一个朋友关系，即：每个人有可以拥有多个朋友:

```ruby
current_user.friends.push << user1 << user2
```
步骤：

(1). 首先创建一个用于存储朋友关系的表：

```ruby
class CreateFriendships < ActiveRecord::Migration
  def change
    create_table :friendships do |t|
      t.references :friend, index: true
      t.references :person, index: true

      t.timestamps
    end
  end
end
```
(2). 修改 `User` 模型:

```ruby
class User < ActiveRecord::Base
   has_many :friendships
   has_many :friends, :through => :friendships
end
```

(3). 添加 `Friendship` 模型:

```ruby
class Friendship < ActiveRecord::Base
   belongs_to :user
   belongs_to :friend, :foreign_key => :friend_id, :class_name => :User
end
```

实际上，步骤 `(2)` 和 `(3)` 还可以简化为:

```ruby
class User < ActiveRecord::Base
  has_and_belongs_to_many :friends, :class_name => :User, :join_table => "friendships",
                  :association_foreign_key => :friend_id
end
```

到目前为止已经可以实现了在用户之间建立朋友关系，但是上面实现的这个关联是单方向的：当把张三设置为李四的朋友(lisi.friends << zhangsan)之后，李四并不会出现在张三的朋友集合(zhangsan.friends)中，我没有找到能很好地实现此种需求的关联设置方案，但是从 RailsCasts(见下面参考部分的链接)里找到一个替代方案。实施方法如下：

修改 `User` 模型如下即可：

```ruby
class User < ActiveRecord::Base
  has_many :friendships
  has_many :friends, :through => :firendships
  has_many :inverse_friendships, :class_name => :Friendships, :foreign_key => :friend_id
  has_many :inverse_friends, :through => :inverse_friendships, :source => :user
end
```
此时，当将张三添加到李四的朋友集合中(`lisi.friends << zhangsan`)后，那么李四就会出现 `zhangsan.inverse_friends` 集合中

参考： 

+ [Self-Referential Association](http://railscasts.com/episodes/163-self-referential-association)
+ [rails many to many self join](http://stackoverflow.com/questions/3396831/rails-many-to-many-self-join)