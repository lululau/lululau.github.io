---
layout: post
title:  "Ruby 1.9+ 的字符编码"
date:   2014-01-18 01:46:00
categories: 
  - Ruby
---


从 1.9 开始，Ruby 增加了对字符编码的支持。这篇文章基本上是看了 Ruby 2.0 镐头书第 17 章 `Character Encoding` 做的笔记，并补充了一些自己通过实验得到的结论。

## Ruby 代码文件的编码

(1). Ruby 源文件的默认编码：  
    你需要告诉 Ruby 你的 Ruby 代码文件使用的是什么编码，因为 Ruby 中的字符串字面量、Symbol 字面量以及正则表达式字面量的字符编码在多数时候取决于定义他们的源文件的字符编码。

   - Ruby 1.9 默认 Ruby 源文件的编码为 `US-ASCII`     

```ruby
#!/usr/bin/env ruby
# default_source_encoding.rb
puts__ENCODING__  # 通过__ENCODING__来查询当前文件的字符编码
```

```sh
rvm use 1.9 
ruby default_source_encoding.rb
# 输出：
US-ASCII
```
  - Ruby 2.0 默认 Ruby 源文件的编码为 `UTF-8`    

```sh
rvm use 2.0
ruby default_source_encoding.rb
# 输出：
UTF-8
```

(2). 指定 Ruby 源文件的字符编码  

  在 Ruby 1.9 中，如果代码文件中包含了非 ASCII 字符，或者在 Ruby 2.0 中代码文件中包含了非 UTF-8 字符，那么就需要在代码文件中声明该代码文件的字符编码：

```ruby
#!/usr/bin/env ruby
#  non_ascii.rb
puts '中文'
```

```sh
rvm use 1.9
ruby non_ascii.rb
# 输出：
non_ascii.rb:2: invalid multibyte char (US-ASCII)
non_ascii.rb:2: invalid multibyte char (US-ASCII)
```

   Ruby 使用一个看似神奇实则很简单的标记规则来指定代码文件的字符编码：如果一个文件的第一行（如果第一行是 UNIX shebang `#!`，那么就是第二行）是注释行，Ruby 会使用 `coding:\s*(\S+)` 这个正则表达式来对这个注释行进行匹配，如果匹配成功那么该文件的字符编码就被设置为 `$1`的值。所以，可以这样将一个 Ruby 代码文件的字符编码设置为 UTF-8：

```ruby
# coding: utf-8
```

   因为 Ruby 只是检索字符串中是否包含 `coding:` 这个子字符串，所以实际上也可以这样写：

```ruby
# encoding: utf-8
```

   Emacs 用户可能会更喜欢这样写：

```ruby
# -*- encoding: utf-8 -*-
```

   另外，如果 Ruby 代码文件包含了 UTF-8 BOM，也就是说代码文件的头三个字节是 `\xEF\xBB\xBF`，那么 Ruby 认为这个代码文件的字符编码是 UTF-8，而不管上述的标记行：  

```ruby
#!/usr/bin/env ruby
# coding: GBK
# gbk.rb
puts__ENCODING__
```
```sh
rvm use 2.0
ruby gbk.rb
# 输出:
GBK
ruby -e 'print [0xEF, 0xBB, 0xBF].pack("c*")' > bom.rb
cat gbk.rb >> bom.rb
ruby bom.rb
# 输出：
UTF-8
```    

(3).  查询代码文件的编码：  

   特殊常量 `__ENCODING__` 存储了文件的字符编码
     
## 字符串（还有Symbol和Regexp）字面量的字符编码

在 Ruby 1.9+ 中，每一个字符串对象、Symbol 对象和正则表达式对象都有自己的字符编码。

```ruby
#!/usr/bin/env ruby
# coding: utf-8
# show_encoding.rb
str = "中文"
sym = :name
regex = Regexp.new(str.encode("GBK"))

puts str.encoding
puts sym.encoding
puts regex.encoding
```
```sh
rvm use 2.0
ruby show_encoding.rb
# 输出：
UTF-8
US-ASCII
GBK
```

字符串对象、Symbol 对象和正则表达式对象的字面量的编码是这样确定的：

(1). 字符串字面量总是以定义它的源代码文件的字符编码来编码的。  

```ruby
#!/usr/bin/env ruby
# coding: utf-8
# utf8.rb
puts "abc".encoding
puts "中文".encoding
```

```ruby
#!/usr/bin/env ruby
# coding: GBK
# gbk.rb
puts "abc".encoding
puts "中文".encoding
```

```sh
rvm use 2.0
ruby utf8.rb
ruby gbk.rb
# 输出:
UTF-8
UTF-8
GBK
GBK
```

(2). Symbol 和正则表达式有点特别（我猜测可能出于性能方面的考量）：如果它们只包含 ASCII 字符（即所有字节的最高位都为0），那么它们就以 US-ASCII 编码；否则它们就以定义它们的源代码文件的字符编码来编码。  

```ruby
#!/usr/bin/env ruby
# coding: UTF-8
# sym_regex_encoding.rb
a = :name
b = :名字
x = /hello/
y = /你好/
puts a.encoding
puts b.encoding
puts x.encoding
puts y.encoding
```

```sh
rvm use 2.0
ruby sym_regex_encoding.rb
# 输出:
US-ASCII
UTF-8
US-ASCII
UTF-8
```

(3). 一个例外：  

在字符串和正则表达式中，可以使用 `\uxxxx` 或 `\u{x... x... x...}` 来创建任意的 UNICODE 字符，如果一个字符串字面量或者正则表达式字面量中包含了 `\uxxxx` 或 `\u{x... x... x...}` 标记，且此标记所表示的字符不是 ASCII 字符，那么它的编码将设置为 UTF-8，而不管定义它的源代码文件的字符编码是什么。  

```ruby
#!/usr/bin/env ruby
# coding: GBK
# unicode_notation.rb
a = "a"
b = "中"
x = "\u0061"
y = "\u2d4e"
puts a.encoding
puts b.encoding
puts x.encoding
puts y.encoding
```

```sh
rvm use 2.0
ruby unicode_notation.rb
# 输出:
GBK
GBK
GBK
UTF-8
```

## 虚拟编码 ASCII-8BIT

Ruby 支持一个叫做 `ASCII-8BIT` 的虚拟字符编码。这个虚拟编码更多地是用来处理二进制数据，或者在不确定 Ruby 代码文件编码时也可以将其指定为 `ASCII-8BIT`。
    
## 编码转换

(1). 可以将字符串从一个编码转换为另外一个编码  

```ruby
#!/usr/bin/env ruby
# coding: UTF-8
# transcoding.rb
a = "中"
puts a.encoding
p a.bytes.map { |e| e.to_s(16) }
b = a.encode("GBK")
puts b.encoding
p b.bytes.map { |e| e.to_s(16) }
```

```sh
rvm use 2.0
ruby transcoding.rb
# 输出:
UTF-8
["e4", "b8", "ad"]
GBK
["d6", "d0"]
LANG=zh_CN.UTF-8 echo -n 中 | od -An -tx1
# 输出：
e4  b8  ad
LANG=zh_CN.UTF-8 echo -n 中 | iconv -t GBK | od -An -tx1
# 输出：
d6  d0
```

(2). 改变一个对象的编码  

 `encode` 方法实际上是返回一个新的对象，而要改变一个对象的编码，则使用 `force_encoding` 方法：    

```ruby
#!/usr/bin/env ruby
# coding: ASCII-8BIT
# force_encoding.rb
a = "中"
puts a.encoding
p a.bytes.map { |e| e.to_s(16) }
a.force_encoding("UTF-8")
puts a.encoding
p a.bytes.map { |e| e.to_s(16) }
a.force_encoding("GBK")
puts a.encoding
p a.bytes.map { |e| e.to_s(16) }
```

```sh
rvm use 2.0
ruby force_encoding.rb
# 输出:
ASCII-8BIT
["e4", "b8", "ad"]
UTF-8
["e4", "b8", "ad"]
GBK
["e4", "b8", "ad"]
```

**可以看到 `force_encoding`只是改变了对象的字符编码，并没有改变存储字符的实际字节。** 

## IO 的字符编码

如果将一个某种特定字符编码的字符串输出到外部 IO 对象时，Ruby 将会使用什么编码输出这个字符串呢？答案取决于这个 IO 对象的编码是什么。  
每个IO对象都有两个和字符编码相关的属性：外部编码 `external_encoding` 和 内部编码 `internal_encoding`。  

(1). 输出过程中的编码转换  

   与输出数据到一个 IO 对象这个过程相关的是 `external_encoding`， 输出过程中的字符编码转换规则为：若此 IO 对象的 `external_encoding` 为 `nil` ，则被输出的对象将不会被转换字符编码而直接输出其内存中的实际字节；否则，被输出的对象将使用 `external_encoding` 进行编码，编码过程中所使用的源编码为被输出对象的 `encoding` 属性。  

```ruby
#!/usr/bin/env ruby
# coding: UTF-8
# output_transcoding.rb
s_utf8 = "中"
p s_utf8.bytes.map { |e| e.to_s(16) }   # => ["e4", "b8", "ad"]
s_gbk = s_utf8.encode("GBK")
p s_gbk.bytes.map { |e| e.to_s(16) }    # => ["d6", "d0"]
p s_gbk.encoding    # =>  #<Encoding:GBK>
p STDOUT.external_encoding    # => nil
p STDOUT.internal_encoding    # => nil
puts s_gbk      # => 0xd6  0xd0 ， 说明 s_gbk 的字节没有经过编码转换而直接输出
STDOUT.set_encoding("UTF-8:Windows-31J")
p STDOUT.external_encoding     # => #<Encoding:UTF-8>
p STDOUT.internal_encoding     # => #<Encoding:Windows-31J>
puts s_gbk      # => 0xe4  0xb8  0xad ，被正常转换为 UTF-8，说明数据输出过程中的编码转换和 internal_encoding <Windows-31J> 无关
```

(2). 输入过程中的编码转换  
   当从一个 IO 对象读取数据时，读取的数据的编码和此 IO 对象的 `external_encoding` 和 `internal_encoding` 两个属性都有关系，具体的规则为：若 `internal_encoding`  为 nil，那么外部数据将被不经任何转换地读进内存，在内存中存储此块数据的对象的 `encoding` 属性被设置为此 IO 对象的 `external_encoding`；否则，外部数据被读进内存时将被转换为 `internal_encoding` 所标识的字符编码，且存储此块数据的对象的 `encoding` 属性被设置为 `internal_encoding`，编码转换所使用的源编码为 `external_encoding`。  

```sh
echo -n $'\xe4\xb8\xad' > tmp  # 向 tmp 文件中输出“中”字经 UTF-8 编码的字节序列
cat tmp
#输出：
中
```

```ruby
#!/usr/bin/env ruby
# input_transcoding.rb
File.open "tmp", "r:UTF-8" do |f|
    p f.external_encoding   # => #<Encoding:UTF-8>
    p f.internal_encoding   # => nil
    l = f.gets
    p l.encoding   # => #<Encoding:UTF-8>
    p l.bytes.map { |e| e.to_s(16) }  # => ["e4", "b8", "ad"]
end
File.open "tmp", "r:UTF-8:GBK" do |f|
    p f.external_encoding  # => #<Encoding:UTF-8>
    p f.internal_encoding  # => #<Encoding:GBK>
    l = f.gets
    p l.encoding  # => #<Encoding:GBK>
    p l.bytes.map { |e| e.to_s(16) }  # => ["d6", "d0"]
end
```
    
(3). 设置 IO 对象的字符编码  
   在使用 `IO.new()`创建一个 IO 对象时，可以指定这个对象的 `external_encoding` 和 `internal_encoding`：  

```ruby
#!/usr/bin/env ruby
# open_file.rb
hello = File.new "hello", "w:gbk"
world = File.new "world", "w:ISO8859-1:sjis"
p hello.external_encoding
p hello.internal_encoding
p world.external_encoding
p world.internal_encoding
```

```sh
rvm use 2.0
ruby open_file.rb
# 输出
#<Encoding:GBK>
nil
#<Encoding:ISO-8859-1>
#<Encoding:Windows-31J>
```

   若要修改一个 IO 对象的 `external_encoding` 和 `internal_encoding`，使用 `IO#set_encoding()` 方法：  

```ruby
#!/usr/bin/env ruby
# set_enc.rb
p STDOUT.external_encoding
p STDOUT.internal_encoding
p STDERR.external_encoding
p STDERR.internal_encoding
STDOUT.set_encoding('gbk')
STDERR.set_encoding('sjis:utf-8')
p STDOUT.external_encoding
p STDOUT.internal_encoding
p STDERR.external_encoding
p STDERR.internal_encoding
```

```sh
rvm use 2.0
ruby set_enc.rb
# 输出
nil
nil
nil
nil
#<Encoding:GBK>
nil
#<Encoding:Windows-31J>
#<Encoding:UTF-8>
```

### IO 默认编码

Ruby 1.9+ 还有一个 IO 默认外部编码 `Encoding.default_external` 和 IO 默认内部编码 `Encoding.default_internal`的概念，不过通过我在 ruby-2.0.0-p247 上的实践，发现这个概念真是一团糟。总的来说， 当你创建一个 IO 对象时，如果没有在 mode 参数里指定内部编码和外部编码，那么这个 IO 对象的内部编码和外部编码会分别设置为这两个默认编码，但是这需要满足以下规则：

(1). 如果 `Encoding.default_internal` 为 nil，那么用户创建的 IO 对象的内部编码和外部编码，与这两个默认编码没有关系，也就是说在这种情况下，即便是创建 IO 对象时没有指定内部编码和外部编码，Ruby 也不会用这两个默认编码的值去设置这个 IO 对象的内部和外部编码。  

   **例外： 如果 IO 对象是以 readonly (如`File.new filename, "r"`)模式打开的，且没有指定内部编码和外部编码，那么不管`default_internal`是否为 nil，那么该对象的外部编码都将被设置为 `default_external` 的值。**

(2). 如果 `Encoding.default_internal` 和 `Encoding.default_external` 的值相同（顺便提一下，`default_external` 的值永远不会是 nil），那么如果创建 IO 对象时没有指定内部编码和外部编码，那么这个 IO 对象的外部编码将被设置为 `default_external` 的值，而 IO 对象的内部编码不会被设置。

(3). 如果 `default_internal` 值不为 nil，且与 `default_external` 不相等，创建 IO 对象时没有指定内部编码和外部编码，那么这个 IO 对象的外部编码将被设置为 `default_external` 的值，内部编码被设置为 `default_internal`。

另外，当从一个 IO 对象读取数据时，如果该 IO 对象的 `external_encoding` 和 `internal_encoding` 都为 nil，那么外部数据将被不经任何转换地读进内存，在内存中存储此块数据的对象的 `encoding` 属性被设置为`Encoding.default_external`；

#### 设置默认编码

(1). 可以通过 `Encoding.default_external=()` 和 `Encoding.default_internal=()` 来设置默认外部编码和默认内部编码。  
(2). 也可以通过 ruby 解释器的 `-E` 选项来指定默认外部编码和默认内部编码。  
(3). 另外，Ruby 也会从 `LANG` 环境变量推断默认的外部编码  
(4). 如果没有设置 `LANG` 环境变量，也没有指定 `-E` 选项，那么默认的外部编码就被设置为 `US-ASCII`  

#### 标准 IO 对象的默认编码

`STDIN`、`STDOUT`和`STDERR`这三个标准 IO 对象的外部编码和内部编码的默认值受 `Encoding.default_external` 和 `Encoding.default_internal` 控制，其规则和前文所述的 `default_external`、`default_internal` 对新建为指定编码的IO对象的控制规则一致。

一个典型的例子：系统的 LANG 环境变量为 `zh_CN.UTF-8`，且没有指定 ruby 解释器的 `-E` 选项，那么这 3 个标准 IO 对象的内部编码和外部编码分别为：

```ruby
#!/usr/bin/env ruby
# stdio_encoding.rb

puts Encoding.default_external  # => UTF-8
puts Encoding.default_internal  # => nil

puts STDIN.external_encoding  # => UTF-8
puts STDIN.internal_encoding  # => nil

puts STDOUT.external_encoding  # => nil
puts STDOUT.internal_encoding  # => nil

puts STDERR.external_encoding  # => nil
puts STDERR.internal_encoding  # => nil
```
