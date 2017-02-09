---
layout: post
title:  "Ruby 的 Time 类和 DateTime 类的区别"
date:   2013-11-20 21:26:00
categories: 
  - Ruby
---

`Time` 是 Ruby 对 `POSIX` 的原始数据类型 `time_t` 的封装，换句话说，每一个 `Time` 对象都是表示距离 UNIX 纪元（即1970-01-01 00:00:00 UTC）的秒数。它可以用一个正的或者负的整数来表示，并且是有界限的：

```ruby
Time.at(0x7FFF_FFFF)     # => 2038-01-19 11:14:07 +0800
Time.at(-0x7FFF_FFFF)    # => 1901-12-14 05:12:37 +0826
```

在 Ruby 1.8.7 中超出此边界的值会产生一个错误。（1.9.2 貌似可以处理更大一些的值）

相比 `Time` 类，`DateTime` 就没有边界的限制。而 Rails 更是依据 SQL 标准中的 `DATETIME` 类型扩展了 `DateTime`，它可以表示任何一个日期。所以如果是亚里士多德在他的那个年代在网上发表一个帖子，用 `DateTime` 类的实例是可以记录他的这个帖子的发表时间的。