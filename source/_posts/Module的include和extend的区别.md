---
title: Module的include和extend的区别
date: 2016-11-06 08:57:05
desc:
tags:
---
在`ruby`中我们经常用`Module`来扩展类的功能，我们常常看到这样的代码：
```ruby
module Sayable
  def say
    puts "hello"
  end
end

class People
  extend  Sayable
  include Sayable
end
```
我们看到我们在这里分别使用了`extend`和`include`,那么`extend`和`include`有什么区别呢？我们接下来实验一下：
我们先定义一个`module`,然后定义两个类分别使用`extend`和`include`，然后看看这个两个类的扩展有啥区别。

<!-- more -->

```ruby
module Testable
  def test
    puts "test"
  end
end

class A
  extend Testable
end

class B
  include Testable
end

A.test # test
A.new.test # NoMethodError: private method `test' called for #<A:0x007f9528d33b40>

B.test # NoMethodError: private method `test' called for B:Class
B.new.test  # test

```
看到上面的实验结果我们知道当我们`extend`一个模块的时候，这个模块的所有方法会变成`扩展类`的`类方法`，而当我们`include`一个模块的时候，这个模块的所有方法会变成`扩展类`的`实例方法`。
