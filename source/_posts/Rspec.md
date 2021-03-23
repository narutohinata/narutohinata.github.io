---
title: RSpec Expectations
date: 2016-11-1 11:11:11
---


# RSpec Expectations 3.6
`rspec expectations` 是被用来定义预期的结果的。
```ruby
Rspec.describe Account do
  it "has a balance of zero when first creare" do
    expect(Account.new.balance).to eq(Money.new(0))
  end
end
```

<!-- more -->

## 基本结构
`rspec expectation` 的基本结构是:
```ruby
expect(actual).to matcher(expected)
expect(actual).no_to matcher(expected)
```
注意：你同样也可以使用`expect(..).to_not` 代替`expect(..).not_to`。这个就是它的别名，所以你可以使用你认为读起来更好的那一个。
### Examples
```ruby
expect(5).to eq(5)
expect(5).not_to eq(4)
```
### matcher 是什么
`matcher`是可以响应下面方法的任何一个对象：
```ruby
matcher?(actual)
failure_message
```
这些方法们也是matcher协议的一部分，但是是可选的：
```ruby
does_not_match?(actual)
failure_message_when_negated
description
supports_block_expectations?
```
`Rspec`附带的一些内建的`matchers`和用来写定制化`mathcers`的`DLS`