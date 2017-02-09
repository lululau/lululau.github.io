---
layout: post
title:  "《Ruby 元编程》中的那些干货"
date:   2013-10-03 19:55:00
categories: 
  - Ruby
---

## 1. 对象模型

### 1.1 打开类

从某种意义上说，Ruby 的 `class` 关键字更像是一个作用域操作符而不是类声明语句。它的确可以创建一个还不存在的类，不过也可以把这看成一种副作用。对于 `class` 关键字，其核心任务是把你带到类的上下文中，让你可以在其中定义方法。

**示例1:**

你的代码中需要定义一个函数去掉字符串中的标点符号和特殊字符，只保留字母、数字和空格。

在像 Java 这样的编程语言中，你必须这样定义这个方法：

```java
public class StringUtil {
    public static String to_alphanumeric(s) {
        return s.replaceAll("[^\\w\\s]", "");
    }
}
```

而在 Ruby 中你可以将这个 to_alphanumeric 方法直接定义为 String 类的实例方法：

```ruby
class String
    def to_alphanumeric
        self.gsub /[^\w\s]/, ''
    end
end
```

**示例2:**

```ruby
class D
    def x; 'x'; end
end

class D
    def y; 'y'; end
end

obj = D.new
obj.x        # => 'x'
obj.y        # => 'y'
```

*使用“打开类”技术时，需要注意的问题：在打开一个类向其中添加一个新的方法时，需要注意这个类中是否已经存在一个同名的方法，以免不小心覆盖了类中原有的同名方法。*

### 1.2 几个 Reflection 方法

#### 1.2.1 `Object#instance_variables()`

   Returns an array of instance variable names for the receiver. Note that simply defining an accessor does not create the corresponding instance variable.
   
#### 1.2.2 `Object#methods(all=true)`   
   Returns a list of the names of public and protected methods of obj. This will include all the methods accessible in obj’s ancestors. If the all parameter is set to false, only those methods in the receiver will be listed.

#### 1.2.3 `Module#instance_methods(include_super=true)`

   Returns an array containing the names of the public and protected instance methods in the receiver. For a module, these are the public and protected methods; for a class, they are the instance (not singleton) methods. With no argument, or with an argument that is false, the instance methods in mod are returned, otherwise the methods in mod and mod’s superclasses are returned.
   
#### 1.2.4 `Module#ancesotrs()`

   Returns a list of modules included in mod (including mod itself).
   
```ruby
class MyClass
    def my_methods; 'my_method()'; end    
end

class MySubclass < MyClass
end

MySubclass.ancestors()        # => [MySubclass, MyClass, Object, Kernel, BasicObject]
```

### 1.3 常量的作用域和路径

常量的作用域不同于变量，它有自己独特的规则。例如：

```ruby
module MyModule
    MyConstant = 'Outer Constant'
    class MyClass
        MyConstant = 'Inner Constant'
    end
end
```

这段代码中的所有常量像文件系统一样组织成树形结构：

```
MyModule
├── MyClass
│   └── MyConstant
└── MyConstant

```

### 1.4 几个和常量有关的方法

#### 1.4.1 `Module#constants(inherit=true)`

   Returns an array of the names of the constants accessible in mod. This includes the names of constants in any included modules (example at start of section), unless the inherit parameter is set to false.



#### 1.4.2 `Module.constants()` 和 `Module.constants(inherit)`

   In the first form, returns an array of the names of all constants accessible from the point of call. This list includes the names of all modules and classes defined in the global scope.

   The second form calls the instance method constants.
   
#### 1.4.3 `Module.nesting()`   
   获取当前常量的路径。例如：
   
```ruby
moudle M
    class C
        module M2
            Module.nesting        # => [M::C::M2, M::C, M]
        end
    end
end        
```

### 1.5 修剪常量树

若想在网上找到一个 `motd.rb` 文件用来在控制台上显示“当天的消息”，且想把这段代码集成到最新的程序中去，那么使用`load`执行该文件来显示消息：

```ruby
load('motd.rb')
```

不过，使用 `load()` 方法有一个副作用。`motd.rb` 文件很可能定义了变量和类。尽管变量在加载完成后会落在当前作用域之外，但常量不会。这样，`motd.rb`可能会通过它的常量（尤其是类名）污染当前程序的命名空间。

可以通过使用第二个可选参数来控制其常量仅在自身范围内有效：

```ruby
load('motd.rb', true)
```

通过这种方式加载的文件，Ruby 会创建一个匿名模块，使用它作为命名空间来容纳 `motd.rb` 中定义的所有常量，加载完成后，该模块会被销毁。

`require()`方法与`load()`方法颇为相似，但是它的目的不同。通过`load()`方法可以执行代码，而`require()`则是用来导入类库。这就是`require()`方法没有第二个可选参数的原因。在这些类库中的类名通常是你导入这些库时所希望得到的，因此没有理由在加载后销毁它们。

### 1.6 关于 Kernel 模块

如果需要定义一个工具函数，这个函数的作用很广泛，以至于它使用起来更像语言内核的一部分，那么这个方法最好定义 Kernel 模块的方法。

### 1.7 `private` 究竟意味着什么

私有方法服从一个简单的规则：不能明确指定一个接受者来调用一个私有方法。换言之，每次调用一个私有方法时，只能调用于隐含的接受者 —— `self` 上。下面看一个极端例子：

```ruby
class C
    def public_method
        self.private_method
    end
    
    private
    
        def private_method; end
end

C.new.public_method

 # => NoMethodError: private method `private_method' called […]
```

在这段代码中，如果去掉 `self` 关键字，它就可以正常运行。

这个人为制造的例子演示了私有方法是由两条规则一起控制的：第一条：如果调用方法的接受者不是你自己，则必须明确指明一个接受者；第二条，私有方法只能被隐含接受者调用。把这两条规则糅合在一起，你会发现只能在自身中调用一个私有方法。你可以把这个糅合后的规则成为“似有规则”。

### 1.8 混乱的模块

有如下代码：

```ruby
module Printable
    def print; end
end

module Document
    def print; end
end

class Book
    include Document
    include Printable
end

book = Book.new
b.print
```

问：`b.print` 调用的是 `Printable` 还是 `Document` 中定义的 `print` 方法？

答：`Book.ancestors` 的输出：`[Book, Printable, Document, Object, .. ..]`，因此调用的是 `Printable` 中定义的 `print` 方法。

### 1.9 完整的类和对象模型图

下列代码：

```ruby
class MyClass
    def method_a; end
end

obj1 = MyClass.new
obj2 = MyClass.new
obj3 = MyClass.new

class << obj3
    def method_b; end
end
```

对应的模型图如：

![ObjectModel.png](/assets/images/2013-10-03-the-ruby-metaprogramming-in-the-dry/1.png)

## 2. 方法

### 2.1 动态方法

#### 2.1.1 动态方法所依赖的基础

* 动态派发：`Object#send(symbol_or_string [, args…])`

  Invokes the method identified by symbol, passing it any arguments specified. You can use `__send__` if the name send clashes with an existing method in obj. When the method is identified by a string, the string is converted to a symbol.

* 动态定义：`Module#define_method(symbol, method)` 或 `Module#define_method(symbol){ block }`

  Defines an instance method in the receiver. The method parameter can be a Proc, a Method or an UnboundMethod object. If a block is specified, it is used as the method body. This block is evaluated using instance_eval. `define_method` is a **private** method.
  
#### 2.1.2 示例

有这样一个程序：

```ruby
 # data_source.rb:
class DS
  def initialzie # connect to data source…
  def get_mouse_info(workstation_id)  # …
  def get_mouse_price(workstation_id)  # …
  def get_keyboard_info(workstation_id)  # …
  def get_keyboard_price(workstation_id)  # …
  def get_cpu_info(workstatiob_id)  #…
  def get_cpu_price(workstation_id)  #…
  def get_display_info(workstation_id)  # …
  def get_display_price(workstation_id)  #...
end

 # computer.rb
class Computer
  def initialzie(id, data_source)
    @id = id
    @data_source = data_source
  end
  
  def mouse
    info = @data_souce.get_mouse_info(@id)
    price = @data_source.get_mouse_price(@id)
    result = "Mouse: #{info} ($#{price})"
    return "* #{result}" if price > 100
    result
  end
  
  def cpu
    info = @data_souce.get_cpu_info(@id)
    price = @data_source.get_cpu_price(@id)
    result = "Mouse: #{info} ($#{price})"
    return "* #{result}" if price > 100
    result  
  end
  … … 
end  
```

##### 2.1.2.1 使用动态派发重构 `computer.rb`

```ruby
class Computer
  def initialize(id, data_source)
    @id = id
    @data_source = data_source
  end
  
  def mouse
    component :mouse
  end
  
  def cpu
    component :cpu
  end
  … …
  
  def component(name)
    info = @data_source.send("get_#{name}_info", @id)
    price = @data_source.send("get_#{name}_price", @id)
    result = "#{name.to_s.capitalize}: #{info} ($#{price})"
    return "* #{result}" if price > 100
    result
  end
end
```

##### 2.1.2.2 使用动态定义重构 `computer.rb`

```ruby
class Computer
  def initialzie(id, data_source)
    @id = id
    @data_source = data_source
  end

  def self.component(name)
    define_method name do 
      info = @data_source.send "get_#{name}_info", @id
      price = @data_source.send "get_#{name}_price", @id
      result = "#{name.to_s.capitalize}: #{info} ($#{price})"
      return "* #{result}" if price > 100
      result
    end
  end
  
  component :mouse
  component :cpu
  … …
end
```

##### 2.1.2.3 使用内省重构 `computer.rb`

```ruby
class Computer
  def initialzie(id, data_source)
    @id = id
    @data_source = data_source
    @data_source.methods.grep(/^get_(\w+)_info/) {self.class.component $1}
  end

  def self.component(name)
    define_method name do 
      info = @data_source.send "get_#{name}_info", @id
      price = @data_source.send "get_#{name}_price", @id
      result = "#{name.to_s.capitalize}: #{info} ($#{price})"
      return "* #{result}" if price > 100
      result
    end
  end 
end
```

### 2.2 `method_missing` 幽灵方法(Ghost Method)

#### 2.2.1 `method_missing(symbol [, *args] )` 方法

Invoked by Ruby when obj is sent a message it cannot handle. symbol is the symbol for the method called, and args are any arguments that were passed to it. By default, the interpreter raises an error when this method is called. However, it is possible to override the method to provide more dynamic behavior. If it is decided that a particular method should not be handled, then super should be called, so that ancestors can pick up the missing method.

#### 2.2.2 使用 `method_missing` 重构 `computer.rb`

```ruby
class Computer
  def initialzie(id, data_source)
    @id = id
    @data_source = data_source
  end
  def method_missing(name, *args)
    super unless @data_source.respond_to? "get_#{name}_info"
    info = @data_source.send "get_#{name}_info", @id
    price = @data_source.send "get_#{name}_price", @id
    result "#{name.to_s.capitalzie}: #{info} ($#{price})"
    return "* #{result}" if price > 100
    result    
  end
end
```

#### 2.2.3 覆盖 `respond_to?`

```ruby
class Computer
  def initialzie(id, data_source)
    @id = id
    @data_source = data_source
  end
  def method_missing(name, *args)
    super unless @data_source.respond_to? "get_#{name}_info"
    info = @data_source.send "get_#{name}_info", @id
    price = @data_source.send "get_#{name}_price", @id
    result "#{name.to_s.capitalzie}: #{info} ($#{price})"
    return "* #{result}" if price > 100
    result    
  end
  def respond_to?(name)
    @data_source.respond_to?("get_#{name}_info") || super
  end
end
```

#### 2.2.4 白板类(BlankSlate)

有时候所要代理的方法，已经存在于类或者祖先类种，这种情况下，调用那个方法时，就不会经过 `method_missing` 代理，因此这时，需要将从父类中继承来的那些不需要的方法都删除掉：

```ruby
class Computer
  instance_methods.each do |method|
    undef_method method unless method.to_s =~ /method_missing|respond_to\?/
  end
  … … 
end
```

用于从一个类中删除实例方法的方法：

* `Module#undef_method(symbol)`

  Prevents the current class from responding to calls to the named method. Contrast this with remove_method, which deletes the method from the particular class; Ruby will still search superclasses and mixed-in modules for a possible receiver. String arguments are converted to symbols.


* `Module#remove_method(symbol)`

  Removes the method identified by symbol from the current class. For an example, see Module.undef_method. String arguments are converted to symbols.

> **BasicObject**

>
从 Ruby 1.9 开始，白板技术被集成到语言自身中，在过去的版本中，`Object`是类体系结构的根节点。在 Ruby 1.9 中，`Object`类有一个名叫 `BasicObject` 的超类，它只提供几个很基本的方法：

```ruby
p BasicObject.instance_methods

=> [:==, :requals?, :!, :!=, :instance_eval, :instance_exec, :__sned__]
```

> 默认情况下，类还是会从 `Object` 继承，从 `BasicObject` 继承来的类会自动成为白板类。


## 3. 代码块

### 3.1 当前块

在一个方法中，可以向Ruby询问当前的方法调用是否包含块。这可以通过 `Kernel#block_given?()` 方法来做到：

```ruby
def a_method
  return yield if block_given?
  'no block'
end

a_method
a_method { "here's a block!" }
```

### 3.2 使用 Ruby 实现 C# 的 `using` 关键字

设想在写一个 C# 程序，这个程序会连接一个远程服务器，并有一个对象表示这个连接：

```ruby
RemoteConnection conn = new RemoteConnection("my_server");
String stuff = conn.readStuff();
conn.dispose();
```

这段代码在使用了连接后，会正确的释放连接。然而，它并没有处理异常。如果 `readStuff()` 方法抛出一个异常，那么 `conn` 对象将永远不会得到释放。代码需要将异常管理起来，以便不管是否发生异常都能正确地释放连接。幸运的是，C# 提供了一个叫做 `using` 的关键字，它能帮助你处理整个过程：

```ruby
RemoteConnection conn = new RemoteConnection("some_remote_server");
using (conn)
{
  conn.readSomeData();
  doSomeMoreStuff();
}
```

这个 `using` 关键字期望 `conn` 对象有一个名为 `dispose()` 的方法，当大括号中的代码执行完后，不管有没有异常抛出，这个方法都会被自动调用。

OK，下面这个是用Ruby实现的 `using` 关键字：

```ruby
module Kernel
  def using(obj)
    begin
      yield
    rescue
      obj.dispose
    end 
  end
end
```

### 3.3 闭包


![binding.png](/assets/images/2013-10-03-the-ruby-metaprogramming-in-the-dry/2.png)

正如上图所示，块不仅仅是一段浮动的代码。你不可能在真空中运行代码。当代码运行时，它需要一个执行环境：局部变量、实例变量、self …… 既然这些东西是绑定在对象上的名字，就可以把它们简称为**绑定(binding)**。块的要点在于它们是完整的，可以立即运行。它们既包含代码，也包含一组绑定。

那么块是从哪里获得它的绑定的呢？当定义一个块时，它会获取当时环境中的绑定，并且把它传给一个方法时，它会带着这些绑定一起进入该方法：

```ruby
def my_method
  x = "Goodbye"
  yield "cruel"
end

x = "Hello"
my_method {|y| "#{x}, #{y} world" }  # => "Hello, cruel world"
```

当创建块时会获取到局部绑定（比如上面的`x`），然后把块连同它自己的绑定传给一个方法。在上面的例子中，块的绑定中包括一个名为 `x` 的变量。虽然在方法中定义了一个变量 `x`，块看到的 `x` 也是在块定义时绑定的 `x`，但是方法中的 `x` 对这个块来说是不可见的。基于这样的特性，计算机科学家喜欢把块称为 **闭包(Closure)**。

#### 3.3.1 块局部作用域

块会在定义时获取周围的绑定。你可以在块的背部定义额外的绑定，但是这些绑定在快结束时就会消失：

```ruby
def my_method
  yield
end

top_level_variable = 1
my_method do
  top_level_variable += 1
  local_to_block = 1
end

top_level_variable  # => 2
local_to_block      # => Error!
```

**警告：** 在 Ruby 1.8及之前的版本中，块参数对粗心者而言有一个陷阱。跟你期望的相反，块会覆盖具有相同名字的局部变量：

```ruby
def my_method
  yield(2)
end

x = 1
my_method do |x|
  # 不错什么特殊的操作
end
x  # => 2
```

当把块参数命名为 `x` 时，块发现当前上下文中已存在一个 `x` 变量，于是它把这个值传给了块。这种让人吃惊的行为在过去常常是引发 bug 的根源。有条好消息是，在 Ruby 1.9 中这个问题已经被修正了。

### 3.4 作用域门

Ruby 程序会在三个地方关闭前一个作用域，同时打开一个新的作用域：

* 类定义
* 模块定义
* 方法定义

只要程序进入类或魔窟开及方法的定义，就会发生作用域切换。这三个边界分别用 **class、module** 和 **def**关键字作为标志。每一个关键字都充当了一个**作用域门（Scope Gate）**。

示例：

```ruby
v1 = 1
class MyClass   # => 作用域门：进入 class
  v2 = 2
  local_variables   # => ["v2"]
  
  def my_method # => 作用域门：进入 def
    v3 = 3
    local_variables
  end         # => 作用域门：离开 def
  
  local_variables   # => ["v2"]
end           # => 作用域门：离开 class
obj = MyClass.new
obj.my_method     # => [:v3]
obj.my_method     # => [:v3]
local_variables     # => [:v1, :obj]
```

`Kernel#local_variables`

Returns the names of the current local variables.

### 3.5 全局变量和顶级实例变量

全局变量可以在任何作用域中访问：

```ruby
def a_scope
  $var = "some value"
end

def another_scope
  $var
end

a_scope
another_scope # => "some value"
```

全局变量的问题在于系统的任何部分都可以修改它们。因此，你会立即发现几乎没法追踪谁把它们改成了什么。正因为如此，基本的原则是：如非必要，尽可能少使用全局变量：

```ruby
@var = "The top-level @var"

def my_method
  @var
end

my_method  # => "The top-level @var"
```

如上面的代码所示，只要 `main` 对象在扮演 `self` 的角色，就可以访问一个顶级实例变量。但当其他对象成为 `self` 时，顶级实例变量就退出作用域了：

```ruby
class MyClass
  def my_method
    @var = "This is not the top-level @var!"
  end
end
```

由于不像全局变量那么有全局性，一般认为顶级示例变量比全局变量更安全。

### 3.6 扁平化作用域

* 当想要使变量穿越过类定义时，使用 `Class.new`

* 当想要使变量穿越过方法定义时，使用 `Module#define_method`

示例：

```ruby
my_var = "Success"  
class MyClass
  # 希望在这里打印 my_var
  
  def my_method
    # 还有这里 ……
  end 
end
```

**----------->**

```ruby
my_var = "Success"
MyClass = Class.new do
  p my_var
  define_method :my_method do
    p my_var
  end
end
```


### 3.7 上下文探针 `Object#instant_eval()`

* `instance_eval(string [, filename [, lineno]] )`
* `instance_eval {| | block }`

Evaluates a string containing Ruby source code, or the given block, within the context of the receiver (obj). In order to set the context, the variable self is set to obj while the code is executing, giving the code access to obj’s instance variables. In the version of instance_eval that takes a String, the optional second and third parameters supply a filename and starting line number that are used when reporting compilation errors.

示例：

```ruby
class MyClass
  def initialize
    @v = 1
  end
end

obj = MyClass.new
obj.instance_eval do
  self      # => #<MyClass:0x3340dc @v=1>
  @v        # => 1
  @v += 1
  @v        # => 2
end
```

Ruby 1.9 中引入了一个名为 `instance_exec()` 的方法，它跟 `instance_eval()` 的功能相似，但它允许对块传入参数：

```ruby
class C
  def initialize
    @x, @y = 1, 2
  end
end

C.new.instance_exec(3) {|arg| (@x + @y) * arg}     # => 9
```

### 3.8 可调用对象

可调用对象分为三种：

* proc
* lambda
* 方法

#### 3.8.1 proc 和 lambda 的区别

使用 `Kernel#lambda()` 生成的 Proc 对象就是 lambda，通过其他方式生成的 Proc 对象都简称为 proc。 lambda 和 proc 主要有两个方面的区别：

* 在 lambda 中调用 `return` 是从该 Proc 对象中返回，而不是从调用的方法中返回。proc 中调用 `return` 是从该 Proc 对象中返回
* lambda 调用中对参数的个数会有严格的检查，而 proc 没有。

综上所述，lambda 的行为更像是普通的方法/函数，因此一般都使用 lambda 对象，除非是想使用 proc 的某种特殊功能。

另外，在 Ruby 1.8 中 `Kernel#proc()` 是 `Kernel#lambda()` 的别名，而在 Ruby 1.9 中 `Kernel#proc()` 是 `Proc.new()` 的别名。

#### 3.8.2 方法

可以通过 `Object#method()` 获取一个 Method 对象：

```ruby
class MyClass
  def initialize(value)
    @x = value
  end
  
  def my_method
    @x
  end 
end

obj = MyClass.new(1)
m = obj.method :my_method
m.call()            # => 1
```

可以用 `Method#unbind()` 把一个方法跟它所绑定的对象相分离，该方法再返回一个 `UnboundMethod` 对象。你不能执行 `UnboundMethod` 对象，但能把它绑定到一个对象上，使之再次成为一个 `Method` 对象：

```ruby
unbound = m.unbind
another_obj = MyClass.new(2)  
m = unbound.bind(another_obj)
m.call()              # => 2
```

### 3.9 RedFlag

**初版：**

```ruby
def event(msg)
  puts msg if yield
end

Dir.glob("*events.rb").each {|file| load file}
```

**第2版：**

```ruby
def event(name, &block)
    @events[name] = block
end

def setup(&block)
    @setups << block
end

Dir.glob("*events.rb").each do |file|

    @events = {}
    @setups = []
    load file
    @events.each_pair do |k, v|
        env = Object.new
        @setups.each do |su|
            env.instance_eval &su
        end
        puts k if env.instance_eval &v
    end
end
```

**第3版：**

```ruby
->{

    events = {}
    setups = []

    Kernel.send :define_method, :event do |name, &block|
        events[name] = block
    end

    Kernel.send :define_method, :setup do |&block|
        setups << block
    end

    Kernel.send :define_method, :each_event do |&block|
        events.each_pair do |k, v|
            block.call(k, v)
        end
    end

    Kernel.send :define_method, :each_setup do |&block|
        setups.each do |su|
            block.call(su)
        end
    end

}.call

Dir.glob("*events.rb").each do |file|
    load file
    each_event do |name, event|
        env = Object.new
        each_setup do |su|
            env.instance_eval &su
        end
        puts name if env.instance_eval &event
    end
end
```

## 4. 类定义

### 4.1 `Module#class_eval()` 方法

`class_eval` 用于在一个类中定义实例方法：

```ruby
class Hello; end

Hello.class_eval do
  def hello
    puts "hello world"
  end
end

h = Hello.new
h.hello   # => hello world
```

`instance_eval` 用于在一个类中定义类方法：

```ruby
class Hello; end

Hello.instance_eval do
  def hello
    puts "hello world"
  end
end

Hello.hello   # => hello world
```

### 4.2 类实例变量和类变量

#### 4.2.1 类实例变量

Ruby 解释器假定所有的实例变量都属于当前对象 `self`。在类定义时也是如此：

```ruby
class MyClass
  @my_var = 1
end
```

在类定义的时候，self 的角色由类本身担任，因此实例变量 `@my_var` 属于这个类。类的实例变量不同于类的对象的实例变量。另外一个例子：

```ruby
class MyClass
  @my_var = 1
  def self.read; @my_var; end
  def write; @my_var = 2; end
  def read; @my_var; end
end

obj = MyClass.new
obj.write
obj.read      # => 2
MyClass.read    # => 1
```
#### 4.2.2 类变量

类变量与类示例变量不同，它们可以被自雷或类的实例所使用（在这个意义上，它们更像是 Java 的静态成员）。

```ruby
class D < C
  @@v = 1
  def my_method; @@v; end
end

D.new.my_method     # => 1
```

不幸的是，类变量有一个很不好的怪癖。下面是一个例子：

```ruby
@@v = 1

class MyClass
  @@v = 2
end

@@v     # => 2
```

的搭配这样的结果是因为类变量并不真正属于类————它们属于类体系结构。由于 `@@v` 定义于 `main` 的上下文，它属于 `main` 的类 `Object`，所以也属于 `Object` 的所有后台。 `MyClass` 继承自 `Object`，因此它也共享了这个类变量。

从技术上讲，尽管这种行为可以理解，但它还是很容易把你绊倒，因为可能会遇到上面所示的意外事件，现在绝大多数 Ruby 主义者都避免使用类变量，而尽量使用类实例变量。


### 4.3 `Class.new`

`Class.new(super_class=Object) { |mod| … }`

Creates a new anonymous (unnamed) class with the given superclass (or Object if no parameter is given). You can give a class a name by assigning the class object to a constant.
If a block is given, it is passed the class object, and the block is evaluated in the context of this class using class_eval.

示例：

```ruby
Employee = Class.new(Person) do
  def hello
    ...
  end
end

e = Employee.new
e.hello
```

### 4.4 类方法的写法

1.

```ruby
def MyClass.my_method; end
```

2.

```ruby
class MyClass
  def self.my_method; end
end
```

3.

```ruby
class MyClass
  class << self
    def my_method; end
  end
end
```

在日常编程中应该使用哪种语法，这在很大程度上取决于个人喜好。大多数人认为 `self` 形式的语法可读性更高，而一些人则明确指出 `eigenclass` 才是方法的真正所用之处。专家级的 Ruby 程序猿都会不屑于使用“类名”方法的语法，因为这种方式重复了类的名字，给重构带来了不便。


### 4.5 类方法和`Module#include()` 以及 `Object#extend()`

使用 `Module#inculde()` 方法可以混入一个模块，使得在该模块中定义的方法变为调用类的示例方法：

```ruby
module Hello
  def hello
    "hello world"
  end
end

class Person
  include Hello
end

p = Person.new
p.hello     # => "hello world"  
```

结合 `eigenclass` 和 `Module#include()` 可以将模块中定义的方法变为调用类的类方法：

```ruby
module Hello
  def hello
    "hello world"
  end
end

class Person
  class << self
    include Hello
  end
end

Person.hello      # => "hello world"
```

为了简化上面的操作，Ruby 提供了 `Object#extend` 方法，例如：

```ruby
module Hello
  def hello
    "hello world"
  end
end

class Person; end
Person.extend Hello
Person.hello      # => "hello world"

p = Person.new
p.extend Hello
p.hello         # => "hello world"
```

### 4.6 环绕别名

编写环绕别名的步骤：

1. 给方法定义一个别名
2. 重新定义这个方法
3. 在新的方法中调用老的方法

示例：

```ruby
class String
  alias old_length length
  def length
    if old_length > 5
      "long"
    else
      "short"
    end
  end
end 
```
**警告：** 永远不要把一个环绕别名加载两次！

## 5. 编写代码的代码

### 5.1 Binding 对象

Binding 就是一个用对象表示的完整作用域。你可以通过创建 Binding 对象来捕获并带走当前的作用域。接下来，你还可以通过 `eval()` 方法、`instance_eval()` 方法或 `class_eval()` 方法，在 Binding 对象所携带的作用域中执行代码。

可以使用 `Kernel#binding()` 方法来创建 Binding 对象：

```ruby
class MyClass
  def my_method
    @x = 1
    binding
  end
end

b = MyClass.new.my_method
```

对 `*eval()` 方法家族，可以给它们传递一个 Binding 对象作为额外的参数，代码就可以在这个 Binding 对象所携带的作用域中执行：

```ruby
eval "@x", b      # => 1
```

Ruby 还提供了一个名为 `TOPLEVEL_BINDING` 的预定义常量，它表示顶级作用域的 Binding 对象。你可以在程序的任何地方访问这个顶级作用域：

```ruby
class AnotherClass
  def my_method
    eval "self", TOPLEVEL_BINDING
  end
end

AnotherClass.new.my_method      # => main
```

从某种意义上说，你可以把 Binding 对象看成是一个比块更“纯净”的闭包，因为它们只包含作用域而不包含代码。

### 5.2 代码字符串

代码字符串可以像块一样访问局部变量：

```ruby
array = ['a', 'b', 'c']
x = 'd'
array.instance_eval "self[0] = x"

array   # => ['d', 'b', 'c']
```

### 5.3 `Kernel#eval` 与 Ruby 安全级别

使用 `Kernel#eval()` 的问题在于，这存在遭受代码注入攻击的危险。

Ruby 会自动把不安全的对象——尤其是从外部传入的对象——标记为被污染的。污染对象包括程序从 Web 表单、文件和命令行读入的字符串，甚至包括系统变量。每次从污染字符串运算而来的新字符串，也是被污染的。下面的例子通过调用 `tainted?()` 方法来判断类是不是被污染了：

```ruby
user_input = "User input: #{gets()}"
puts user_input.tainted?
```

每次都要检查字符串是否被污染很麻烦，Ruby 提供了一种叫做安全级别的概念，它能很好地弥补污染对象的不足，当设置一个安全你级别（可以通过给 `$SAFE` 全局变量赋值来实现）时，就禁止了某些特定的潜在危险操作。

有五个安全级别可供选择，从默认的 0（这里是一个“嬉皮士公社”，在这儿你可以不受约束，也可以格式化硬盘）到 4（这里是“军事管辖区”，在这儿你甚至不能自由地退出程序）。例如，在安全级别 2 上，会禁止绝大多数文件相关工作。值得注意的是，在任何大于 0 的安全级别上，Ruby 都会拒绝执行污染的字符串：

```ruby
$SAFE = 1
user_input = "User input: #{gets()}"
eval user_input
```

为了调节安全性，可以在执行代码字符串之前显式去除它的污染小（通过调用 `Object#untaint()` 方法），然后可以依赖安全级别来禁止注入文件操作这样的危险动作。

### 5.4 类扩展混入（Class Extension Mixins）

编写类混入扩展的步骤：

1. 定义一个模块，姑且叫做 `MyMixin`
2. 在 `MyMixin` 中定义一个内部模块（通常把它叫做 `ClassMethods`），并给它定义一些方法。这些方法最终会成为类方法。
3. 覆盖 `MyMixin#included()` 方法来用 `ClassMethods` 扩展包含者（使用 `extend()` 方法）。

示例：

```ruby
module MyMixin
  module ClassMethods
    def hello
      "hello world"
    end
  end
  
  def self.included(base)
    base.extend ClassMethods
  end
end

class Person
  include MyMixin
end

Person.hello      # => "hello world"
```
### 5.5 CheckedAttributes

```ruby
module CheckedAttributes
  module ClassMethods
    def attr_checked name, &block
      define_method name do
        instance_variable_get "@#{name}"
      end
      
      define_method "#{name}=", value do
        raise "Invalid Attribute Value" unless block.call(value)
        instance_variable_set "@#{name}", value
      end
    end
  end
  def self.included(base)
    base.extend ClassMethods
  end
end

class Person
  attr_checked :age do |age|
    age > 18
  end
end

p = Person.new
p.name = 20
p.name        # => 20
p.name = 10     # => raise "Invalid Attribute Value"
```
