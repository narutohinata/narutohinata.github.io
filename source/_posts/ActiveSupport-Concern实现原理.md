---
title: 'ActiveSupport::Concern实现原理'
date: 2017-05-11 18:12:26
desc:
tags:
  - activeSupport
  - rails
---
在`Rails`中我们经常将可复用的代码放在`Concern`中以防止`fat model`具体用法如下(取自rubychina源码)：
```ruby
# 开启关闭帖子功能
module Closeable
  extend ActiveSupport::Concern

  included do
  end

  def closed?
    closed_at.present?
  end

  def close!
    transaction do
      Reply.create_system_event(action: 'close', topic_id: self.id)
      update!(closed_at: Time.now)
    end
  end

  def open!
    transaction do
      update!(closed_at: nil)
      Reply.create_system_event(action: 'reopen', topic_id: self.id)
    end
  end
end


class Topic < ApplicationRecord
  include Closeable
end
```

我们今天就来看看`ActiveSupport::Concern`是如何实现的，以及它解决了以前的那些痛点?

<!-- more -->


## 解决了那些痛点？
  假设我们有2个模块`module A`和`module B`,其中`module B`包含了`module A`，然后我们在类`Test`中`include B`这个时候类`Test`的行为是我们预想的不一样：
  ```ruby
  module A
    def self.included(base)

    # 这个base是包含这个模块的module/class
    #  例如：
    #  class Demo
    #    include A
    #  end
    #  此时的base就是Demo
    # 
      base.extend ClassMethods  #扩展类方法
    end

    def a_instance_method
      "a_instance_method"
    end

    module ClassMethods
      def a_class_method
        "a_class_method"
      end
    end
  end

  module B
    def self.included(base)
      base.extend ClassMethods
    end

    def b_instance_method
      "b_instance_method"
    end
    
    module ClassMethods
      def b_class_method
        "b_class_method"
      end
    end

    include A # 注意这里包含了模块A
  end

  class Test
    include B
  end

  Test.new.a_instance_method # => "a_instance_method"
  Test.new.b_instance_method # => "b_instance_method"

  Test.a_class_method # => NoMethodError: undefined method `a_class_method' for Test:Class
  Test.b_class_method # => "b_class_method"
  ```
  我们发现当Test调用`a_class_method`方法时候抛出了`NoMethodError`错误。

  其实我们仔细思考下就知道问题出在哪里了：
  > 我们在模块`B`中 `include`了模块`A`的时候,`B`充当了`base`导致`A::ClassMethods`模块中定义的方法变成了`module B`的类方法

  而我们的`ActiveSupport::Concern`就完美的解决了链式包含的问题。接下来我们看下到底是怎么实现的吧。

  ## 实现原理
  ```ruby
  module ActiveSupport
    module Concern
      class MultipleIncludedBlocks < StandardError #:nodoc:
        def initialize
          super "Cannot define multiple 'included' blocks for a Concern"
        end
      end

      # 这里设置了一个变量用来标识是否是一个`Concern`模块
      def self.extended(base) #:nodoc:
        base.instance_variable_set(:@_dependencies, [])
      end

      # append_features方法是在模块被包含的时候被调用的，里面包含了一个默认实现
      # 用来检查被包含模块是否已经在包含类的祖先链上，如果不在则将该模块加入其祖先链
      def append_features(base)
        if base.instance_variable_defined?(:@_dependencies)
          base.instance_variable_get(:@_dependencies) << self
          return false
        else
          return false if base < self
          @_dependencies.each { |dep| base.include(dep) }
          super
          base.extend const_get(:ClassMethods) if const_defined?(:ClassMethods)
          base.class_eval(&@_included_block) if instance_variable_defined?(:@_included_block)
        end
      end


      def included(base = nil, &block)
        if base.nil?
          raise MultipleIncludedBlocks if instance_variable_defined?(:@_included_block)

          @_included_block = block
        else
          super
        end
      end

      # 生成模块ClassMethod用来扩展类方法
      def class_methods(&class_methods_module_definition)
        mod = const_defined?(:ClassMethods, false) ?
          const_get(:ClassMethods) :
          const_set(:ClassMethods, Module.new)

        mod.module_eval(&class_methods_module_definition)
      end
    end
  end
```
我们发现`Concern`通过复写了`append_features`改变了默认的包含行为，我们包含一个模块时`Concern`会通过`@_dependencies`检测`base`是否是一个`Concern`如果是一个`Concern`我们就把它加到`@_dependencies`变量中，同时返回`false`以指明该模块没有被真正被包含。如果不是一个`Concern`此时分两种情况：
1. 当`Concern`已经出现在包含类的祖先链中(`if base < self`)我们返回`false`
2. 当`Concern`没有出现在包含类的祖先链中，我们将`@_dependencies`存储的依赖递归去包含(` @_dependencies.each { |dep| base.include(dep) }`)

接下来我们也要把自身也加入祖先链中(`super`)。
然后`extend`方法`class_methods`所定义的内容，以及使用`class_eval`在`base`类中执行`included`方法中所定义的块
