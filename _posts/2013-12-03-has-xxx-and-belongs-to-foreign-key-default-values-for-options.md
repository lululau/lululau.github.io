---
layout: post
title:  "has_xxx 和 belongs_to 的 :foreign_key 选项的默认值"
date:   2013-12-03 23:10:00
categories: 
  - Ruby
  - Rails
---

#### 1. `has_one`、`has_many`、`has_and_belongs_to_many` 的 `:foreign_key` 的默认值

`has_one`、`has_many`、`has_and_belongs_to_many` 的 `:foreign_key` 的默认值为以声明了此关联关系的模型类的类名的小写加上"_id"。例如: 一篇文章(model class: `Article`)有一个作者(model class: `User`)以及多个贡献者(model class: `User`)，对应的这个数据模型的 migration 为：

```ruby
class CreateUsers < ActiveRecord::Migration
  def change
    create_table :users |t|
      t.string :name
      ......
    end
  end
end
```

```ruby
class CreateArticles < ActiveRecord::Migration
  def change
    create_table :articles |t|
      t.string :subject
      ......
      t.references :author
    end
  end
end
```

```ruby
class CreateJoinTableArticlesContributors
  def change
    create_join_table :articles, :users, :table_name => :articles_contributors
  end
end
```

那么若要表示一个用户所主编的所有文章，需要在 `User` 模型类里添加以下的关联声明：

```ruby
class User < ActiveRecord::Base
  has_many :authoring_articles, :class_name => :Article, :foreign_key => :author_id
end
```

#### 2. `belongs_to` 的 `:foreign_key` 的默认值

`belongs_to` 的默认值为关联关系的名称加上"_id"。例如，在上面的例子中，需要表示一篇文章有一个作者，那么需要在 `Article` 模型里添加以下的关联声明：

```ruby
class Article < ActiveRecord::Base
  belongs_to :author, :class_name => :User, :foreign_key => :author_id
end
```

因为 `belongs_to` 的 `:foreign_key` 选项的默认值为此关联关系的名称加上`_id`，即 `author`加上`_id`，因此 article.rb 中的 `belongs_to :author, :class_name => :User, :foreign_key => :author_id` 可以简化为 `belongs_to :author, :class_name => :User`