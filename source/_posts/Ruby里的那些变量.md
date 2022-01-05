---
title: Ruby里的那些变量
date: 2016-9-1 11:11:11
desc: 介绍ruby里的那些变量
tags: ruby
---

## 实例变量(instance variable)
顾名思义`实例变量`就是ruby类的实例的变量,是不是听得有些拗口？没关系我们来看代码:

``` ruby
class People
  def initialize(name)
    @name = name
  end
end

p1 = People.new("小明")
```
这里我们可以看到我在`People`类的初始化方法中传入name参数,然后把参数的值赋给`@name`变量，这个`@name`变量就是我们所称的`实例变量`。而且这些变量是存在于实例的本身的,如下图所示：
![](http://o9dxmww0x.bkt.clouddn.com/ruby_instance_variables.png)
#### 获取实例变量
我们上面初`new`了一个`People`的对象`p1`,这个对象中含有一个`@name`的变量。那我们如何获取`@name`这个变量的值呢？同时我们如何修改这个变量的值呢？

<!-- more -->

有两种方法：
1. 我们给类定义两个方法,一个写方法、一个读方法。这种写法类似`java`里的`sets`和`gets`的写法。
   ``` ruby
    class People
      def initialize(name)
        @name = name
      end

      def name
        @name
      end

      def name=(name)
        @name = name
      end
    end

    p1 = People.new("小明")
    p1.name # => 小明
    p1.name= "小刘"
    p1.name # => 小刘
 ```
2. 我们可以使用`ruby`提供的内置的类宏，`attr_reader`生成写方法、 `attr_writer`生成读方法、`attr_accessor`生成这两种方法。
  ```ruby
    class People
      attr_accessor :name
      def initialize(name)
        @name = name
      end
    end

    p = People.new("小明")
    p.name # => 小明
    p.name= "小花"
    p.name # => 小花
  ```
## 类实例变量(Class instance variable)
我们都知道`ruby`世界里一切都是对象,类也不例外。我们这里的`People`类也是一个对象，生成它的是`Class`类。当类作为一个对象的时候，那些适用于对象的规则也适用于类。也就是说当`People`类当做一个对象的时候，它也有自己的实例变量。这个变量是存在于`People`类里的。

  ```ruby
  class People
    @name = "哈哈"
    
    def self.name
      @name
    end

  end

  People.name # => "哈哈"
  ```
  这里类定义的时候`self`有类本身充当，因此这个实例变量`@name`属于这个类。
  ![](http://o9dxmww0x.bkt.clouddn.com/QQ20170703-210740@2x.png)

##  类变量(Class Variable)
`ruby`类中还有一种以`@@`打头的类变量。类变量和类实例变量不同，它可以被子类和类的实例所访问。(这种行为有点类似`java`里的`静态成员变量`)

```ruby
class A
  @@name = "A"

  def self.name
    @@name
  end

  def name
    @@name
  end
end

class B < A
  def self.name
    @@name
  end

  def name
    @@name
  end
end

A.name # => "A"
A.new.name # => "A"
B.name # => "A"
B.new.name # => "A"
```


