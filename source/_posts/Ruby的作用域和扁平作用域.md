---
title: Ruby的作用域和扁平作用域
date: 2016-09-010 09:26:15
desc: 
tags:
---
在`Ruby`中的作用域和`java`不太一样,它没类似`java`的内部作用域和嵌套作用域,在`java`中我们可能写出这样的代码:
```java
  // outter.java
  class Outter {
    private String name = "我是Outter的name";

    class Inner{
      public void say(){
        System.out.println(name);
      }
    }
  }
  // main.java
  public class main {
    public static void main(String[] args){
      Outter outter = new Outter();
      outter.new Inner().say(); // => 我是Outter的name
    }
  }
```

<!-- more -->


我们可以看到上面的`Inner`的内部类可以访问包裹类`Outter`的成员变量`name`,也就说在`java`中一个作用域可以看到外围作用域的变量。但在`ruby`这样写显然是不行的，不信？我们可以试试：
```ruby
  class Outter
    def self.name
      p @name
    end
    @name = "我是Outter的name"

    class Inner
      def say_outter
        p name
      end
    end
  end

  Outter::Inner.new.say_outter 
  # => NameError: undefined local variable or method `name' for #<Outter::Inner:0x007fcf111e3a10
```
我们可以看到我们调用`Inner`的实例方法`say_outter`报`undefind local variable or method`的错误。
这是因为在`ruby`中一个作用域和作用域内的之间是完全分开的(`井水不犯河水`)。当我们从一个作用域切换到另一个作用域,原先的绑定会被替换成一组新的绑定，我们可以利用`Kernel#local_variables`方法来验证这一问题。(`local_variable`方法会打印出当前绑定的名称)
```ruby
v1 = 1
class MyClass
  v2 = 2
  local_variables # => [:v2]

  def my_method
    v3 = 3
    local_variables 
  end

  local_variables # => [:v2]
end

obj = MyClass.new
obj.my_method  # => [:v3]
obj.my_method  # => [:v3]
local_variables # => [:v1, :obj]

```
在`ruby`中在`类定义`、`模块定义`、`方法定义`的时候会切换作用域。

