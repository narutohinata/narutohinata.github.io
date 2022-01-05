---
title: ActiveRecord里的Scope实现原理
date: 2017-03-05 11:52:37
tags: 
  - Rails
  - ActiveRecord
categories: Ruby
---
# scope的用法
`scope`是我们在`rails`应用中简化代码的一个重要的工具。比如我们有个`Post`模型，这个模型有个`published`的布尔类型的字段用来区分文章是否发布。当我们需要查询所有的已发布文章的集合时候，我们首先想到的是这样写：
<!-- more -->

```ruby
 class PostsController < ApplicationController
  def index
    @posts = Post.where(published: true)
  end
 end
```

但是这样子写导致我们把数据查询的逻辑搬到了`Controller`层,随着业务的复杂我们`Controller`会充斥着大量这样的代码导致可读性很低。这个时候我们可以借助`ActiveRecord`给我们提供的`scope`方法把这些数据查询逻辑封装到`Model`层中。代码如下：

```ruby
  # model/post.rb
  class Post < ApplicationRecord
    scope :published, where(published: true)
  end

  # controller/posts_controller.rb
   class PostsController < ApplicationController
  def index
    @posts = Post.published
  end
 end
```
看这样一整理我们的`Controller`是不是变得很干净了，而且可读性大大增加。

# scope的实现原理
今天我们来看看`ActiveRecord`给我们提供的这个`scope`到底是怎么实现的。在看源码之前我们先看看上面我们写的`scope`的用法，不难发现这个`scope`是个类宏，并且执行了这个类宏后它会给我们在当前类中定义一个类方法，方法名就是我们给`scope`传递的第一个参数，并且这个方法执行的是我们给`scope`传递的第二个参数。例如，我们这么定义一个`scope`: `scope :locked, where(locked: true)`,这会给当前类定义一个名为`locked`的类方法,并且这个类方法执行`where(locked: true)`这个语句。带这个这个猜想我们现在来看看源码是怎么实现这个功能的吧。

```ruby
def scope(name, body, &block)
  unless body.respond_to?(:call)
    raise ArgumentError, "The scope body needs to be callable."
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but Active Record already defined " \
      "a class method with the same name."
  end

  valid_scope_name?(name)
  extension = Module.new(&block) if block

  if body.respond_to?(:to_proc)
    singleton_class.send(:define_method, name) do |*args|
      scope = all.scoping { instance_exec(*args, &body) }
      scope = scope.extending(extension) if extension

      scope || all
    end
  else
    singleton_class.send(:define_method, name) do |*args|
      scope = all.scoping { body.call(*args) }
      scope = scope.extending(extension) if extension

      scope || all
    end
  end

  private

    def valid_scope_name?(name)
      if respond_to?(name, true) && logger
        logger.warn "Creating scope :#{name}. " \
          "Overwriting existing method #{self.name}.#{name}."
      end
    end
end
```
我们可以看到`scope`接收3个参数，比我们猜想的多了个`block`。先不管我们继续向下看，
```ruby
unless body.respond_to?(:call)
    raise ArgumentError, "The scope body needs to be callable."
  end

  if dangerous_class_method?(name)
    raise ArgumentError, "You tried to define a scope named \"#{name}\" " \
      "on the model \"#{self.name}\", but Active Record already defined " \
      "a class method with the same name."
  end
```
这两段语句是检查传入参数的合法性。
`valid_scope_name?(name)`这句是当传入的`name`和类中已经存在的类方法名称一样的时候会提示`Overwriting existing method`

下面这段代码才是`scope`实现的核心代码
```ruby
if body.respond_to?(:to_proc)
  singleton_class.send(:define_method, name) do |*args|
    scope = all.scoping { instance_exec(*args, &body) }
    scope = scope.extending(extension) if extension

    scope || all
  end
else
  singleton_class.send(:define_method, name) do |*args|
    scope = all.scoping { body.call(*args) }
    scope = scope.extending(extension) if extension

    scope || all
  end
end
```
最外层`if`把`body`参数两种类型的情况分开处理，第一种是`body`参数可以响应`to_proc`方法,这里使用了`singleton_class`方法获取了当前类的`单例类`。并通过`动态派发`调用`define_methd`方法定义了一个单例方法(这里也就是类方法),同时使用了`instance_exec`执行了方法体,并且如果存在`extension`就通过`extending`方法增强`scope`，`if`的另一个分支解决了无法响应`to_proc`方法的情况(~~我没有明白为何这里要处理这种情况，ruby里面的callable对象不是都可以响应`to_proc`方法吗?~~)，使用了`body.call(*args)`调用了方法体。两个分支最后都返回了`scope`以保证`链式调用`。