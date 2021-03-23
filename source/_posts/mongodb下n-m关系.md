---
title: 'mongodb下n:m关系'
date: 2018-04-29 10:46:39
desc:
tags:
---

在用户关注系统中关注人和被关注人是`多对多关系`,在关系型数据库中我们很容易通过中间表关联实现这个功能。关注和取消关注无非就是对这个中间表的记录的增加和删除。那么在`mongodb`中如何实现这种多对多关系呢？我们知道`关注操作`就是`被关注操作`的逆操作，当用户`A`关注用户`B`的时候,也就是用户`B`被用户`A`关注了。那么我们可以在用户文档中维护一个用户关注的用户集合`followings`:
```js
{
  username: 'A',
  followings: [ObjectId]
},
{
  username: 'B',
  followings: [ObjectId]
}
```

<!-- more -->

1. 那么用户`A`关注的`B`的操作我们就可以这么写了:
```js
var b = db.users.findOne({ username: 'B' });
db.users.updateOne(
  { username: 'A' },
  { 
    $push: { followings: b._id }
  })
```
2. 那么用户`A`取消关注的`B`的操作我们就可以这么写了:
```js
var b = db.users.findOne({ username: 'B' });
db.users.updateOne(
  { username: 'A' },
  {
    $pull: { followings: b._id }
  })
```
