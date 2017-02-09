---
layout: post
title:  "在 Rails 项目中应用 Rspec/FactoryGirl/Capybara 的配置设置"
date:   2013-11-29 22:47:00
categories: 
  - Ruby
  - Rails
  - Test
---

#### 1. 向 Gemfile 中添加需要的 gems:
  
```ruby
group :development, :test do
  gem "rspec-rails"
  gem "factory_girl_rails"
end

group :test do
  gem "faker"
  gem "capybara"
  gem "database_cleaner"
  gem "launchy"
  gem "selenium-webdriver"
end
```

  其中：
+ `rspec-rails` 封装 Rspec 的程序，还包含了一些专为 Rails 提供的功能;
+ `factory_girl_rails` 把 Rails 生成测试数据默认使用的固件换成更好用的预构件
+ `faker` 为预构件生成名字、Email 地址以及其他的示例数据;
+ `capybara` 便于模拟用户和程序的交互操作;
+ `database_cleaner` 清理“测试数据库”,确保 RSpec 中的测试用例运行于一块净土之上;
+ `launchy` 这个 gem 的功能只有一个,但做的很好,如果需要,它会打开系统的默认浏览器，显示应用程序当前渲染的页面。调试测试时特别有用;
+ `selenium-webdriver` 结合 Capybara 测试基于 JavaScript 的交互操作。

#### 2. 准备数据库

```sh
rake db:create:all
rake db:migrate
rake db:test:clone
```

#### 3. 在项目中安装 Rspec

```sh
rails generate rspec:install
```

#### 4. 修改 Rspec Generator 配置

得益于强大的 Railties,安装 `rspec-rails` 和 `factory_girl_rails` 这两个 gem 之后,一切就都 搞定了。Rails 的生成器将不会在 `test` 文件夹中生成 Test::Unit 测试文件,而是在 `spec` 文件夹中生 成 RSpec 测试文件(生成的预构件存在 `spec/factories` 文件夹中)。不过,如果需要,还可以对生 成器做些设置。如果使用 `scaffold` 生成器生成程序代码的话,可能就想自定义一下设置,因为默认的生成器会生成很多测试文件。打开 config/application.rb 文件,在 Application 类中加入下面的代码:

```ruby
config.generators do|g|
  g.test_framework :rspec,
  fixtures: true,
  view_specs: false,
  helper_specs: false,
  routing_specs: false,
  controller_specs: true,
  request_specs: false
  g.fixture_replacement :factory_girl, dir: "spec/factories"
end
```
 其中：
 
 + `fixtures: true` 的意思是为各模型生成测试固件(使用 Factory Girl 创建的预构件,而不是默认的固件)
 + `view_specs: false` 的意思是不生成“视图测试”。
 + `helper_specs: false` 的意思是生成控制器时不生成对应的帮助方法测试文件。
 + `routing_specs: false` 的意思是不生成针对 config/routes.rb 的测试文件。
 + `g.fixture_replacement :factory_girl` 告知 Rails 使用预构件代替固件,把预构件存放在 `spec/factories` 文件夹中
