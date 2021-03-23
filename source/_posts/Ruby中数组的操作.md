---
title: Ruby中数组的操作
date: 2016-09-21 09:26:15
desc: 小记
tags:
---

```ruby
  # 创建并打印数组
  array = ['one', 'two', 'three']
  p array # => ["one", "two", "three"]
  p array[1] # => "two"

  # 增加数组元素
  # 我们可以这样的
  array << ["four"]
  p array # => ["one", "two", "three", "four"]
  # 还可以这样
  array.concat(["five"])
  p array # =>  ["one", "two", "three", "four", "five"]

  # 获取数组的数量
  p array.count # => 5
  # 获取数组特定元素的数量
  array << ["two"] # =>  ["one", "two", "three", "four", "five", "two"]
  p array("two").cout # => 2

  

  # 清空数组
  array.clear
  p array # => []
```