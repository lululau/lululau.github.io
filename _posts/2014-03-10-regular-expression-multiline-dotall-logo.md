---
layout: post
title:  "正则表达式中的 MULTILINE 和 DOTALL 标识"
date:   2014-03-10 15:34:00
categories: 
  - Regexp
  - Ruby
  - Perl
  - Python
---

#### 1. `MULTILINE` 标识

在 Perl 5 正则中，用于表示字符串开头和字符串结尾的两个表示边界的元字符 `^` 和 `&`, 默认分别只匹配整个字符串的开头和结尾。举例来说，对于字符字符串 `hello\nwordl\n` 来说，虽然整个字符串实际上包含了两行文本，但是使用正则 `/^./g` 只会匹配到字符 `h`，而不会匹配到第二行开头的字符： `w`。

在 Perl 5 正则中，若要使 `^` 和 `$` 匹配字符串中的每一行的开头和结尾，那么需要在正则表达式中加入 `m` 选项。比较下面两个示例的不同：

```perl
# 示例1.pl
my $x = "hello\nworld\n";
$x =~ s#^.#_#g;
$x   # => "_ello\nworld\n"
```

```perl
# 示例2.pl
my $x = "hello\nworld\n";
$x =~ s#^.#_#mg;
$x   # => "_ello\n_orld\n";
```

在 Python 中，使用 `re.M` 或 `re.MULTILINE` 来获得示例2所演示的特性。比较下面两个示例的不同：

```python
# 示例3.py
import re
x = "hello\nworld\n"
re.sub(r"^.", "_", x)   # => "_ello\nworld\n"
```

```python
# 示例4.py
import re
x = "hello\nworld\n"
re.sub(r"^.", "_", x, flags=re.M)   # => "_ello\n_orld\n"
```

Ruby 的正则表达式引擎默认即支持 MULTILINE 模式，因此不需要像 Perl 和 Python 中那样指定一个标记：

```ruby
# 示例5.rb
x = "hello\nworld\n"
x.gsub /^./, '_'  # => "_ello\n_orld\n"
```

#### 2.  `DOTALL` 标记

在 Perl 5 正则中，元字符 `.` 虽然被认为可以匹配任何字符，但实际上默认情况下 `.` 并不匹配换行符，例如：

```perl
# 示例6.pl
my $x = "hello\nworld\n";
$x =~ s#.*#_#;
$x    # => "_\nworld\n"
```

如果想使 `.` 也匹配换行符，需要在正则表达式的后面加上 `s` 选项：

```perl
# 示例7.pl
my $x = "hello\nworld\n";
$x =~ s#.*#_#s;
$x    # => "_"
```

在 Python 中，使用 `re.S` 或 `re.DOTALL` 选项。比较下面两个例子的区别

```python
# 示例8.py
import re
x = "hello\nworld\n"
re.sub(r".*", "_", x, count=1)  # => "_\nworld\n"
```

```python
# 示例9.py
import re
x = "hello\nworld\n"
re.sub(r".*", "_", x, count=1, flags=re.S)  # => "_"
```

需要注意的是在 Ruby 中，需要使用 `m` 标记来达成同样的效果。比较下面两个例子的区别：

```ruby
# 示例10.rb
x = "hello\nworld\n"
x.sub /.*/, "_"    # => "_\nworld\n"
```

```ruby
# 示例11.rb
x = "hello\nworld\n"
x.sub /.*/m, "_"    # => "_"
```