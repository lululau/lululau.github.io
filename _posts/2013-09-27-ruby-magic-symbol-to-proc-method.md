---
layout: post
title:  "Ruby 魔法之 Symbol#to_proc() 方法"
date:   2013-09-27 21:40:00
categories: 
  - Ruby
---

*摘自 “Ruby 元编程” 附录A.5*

这种有点诡异的法术在 Ruby 程序猿中很流行。当第一次接触这种法术时，笔者也很难理解其背后的道理。下面来一步一步地分析它，这样理解它会更容易。

请看下面的代码：

```ruby
names = ['bob', 'bill', 'heather']
names.map {|name| name.capitalize }   # => ["Bob", "Bill", "Heather"]
```
下面关注这个块——这是一个简单的“一次调用块（one-call block）”，它只有一个参数，并且对这个参数只调用一个方法。一次调用块在 Ruby 中十分常见，尤其在（但并非只是在）处理数组的时候。

在像 Ruby 这样以简明扼要闻名的语言中，一次调用块甚至都显得有点啰嗦。为什么必须费力创建一个带有花括号的完整块来让 Ruby 仅仅调用一个方法呢？Symbol#to_proc() 的目的就是用一种更简单的方式来替代一次调用块。下面从必须的最小信息开始，也就是你希望调用的方法的名字，用一个符号表示为：

```ruby
:capitalize
```
你希望把这个符号转换为如下的一次调用块：
```ruby
{|x| x.capitalize}
```
作为第一步，你可以给 Symbol 类增加一个方法，用来把符号转换为一个 Proc 对象：

```ruby
Class Symbol
  def to_proc
    Proc.new {|x| x.send(self) }
  end
end
```
看到这个方法是怎么工作的了么？如果调用它（比如 `:capitalize` 符号），则它会返回一个带有参数的 proc，并且对这个参数调用 `capitalize()` 方法。现在可以使用 `to_proc()` 方法和 `&` 操作符，首先把符号转换为 Proc，然后再转换为块：

```ruby
names = ['bon', 'bill', 'heather']
names.map(&:capitalize.to_proc)   # => ["Bob", "Bill", "Heather"]
```
甚至还可以把这段代码变得更短。由于`&`可以作用于任何对象，所以它会调用该对象的 `to_proc` 方法来把这个对象转换为 `Proc`。（你不会认为是随机选择 `to_proc()` 作为方法名了吧？）因此，可以简单使用下面的方式：

```ruby
names = ['bob', 'bill', 'heather']
names.map(&:capitalize)   # => ["Bob", "Bill", "Heather"]
```
这就是被称为**符号到 Proc** 的法术，漂亮吧？

如果是使用 Ruby 1.9, 你甚至无需自己为 `Symbol` 定义 `to_proc` 方法，因为它已经存在了。事实上，Ruby 1.9 中实现的 `to_proc()` 方法甚至支持对于一个参数的块，可以支持像 inject() 这样的方法：

```ruby
# 不用 Symbol#to_proc:
[1, 2, 5].inject(0) {|memo, obj| memo + obj}  # => 8

# 使用 Symbol$to_proc:
[1, 2, 5].inject(0, &:+)
```