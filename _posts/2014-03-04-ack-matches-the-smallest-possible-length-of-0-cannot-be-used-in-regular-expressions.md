---
layout: post
title:  "ack 中不能使用最小可能匹配长度为0的正则表达式"
date:   2014-03-04 14:07:00
categories: 
  - Shell
---

[ack](http://beyondgrep.com) 是一个使用 Perl 编写的类似于 grep 的文本过滤工具，可以使用 Perl 5 正则表达式。

在使用 ack 的过程中，发现一个值得注意的问题，即：使用最小可能匹配长度为0的正则表达式，可能会使 ack 陷入死循环。

例如：

```sh
echo hello world | ack '.*'
```

将使 ack 陷入死循环。

注意匹配长度为0的匹配和不匹配的区别：

```sh
echo hello world | ack 'x'
```

上面这个例子中，`x` 不匹配 `hello world`，因此 ack 没有任何输出并退出。
而在下面的例子中，`x*` 可以匹配空字符串，匹配长度为0，因此`x*` 会使 ack 进入死循环:

```sh
echo hello world | ack 'x*'
```