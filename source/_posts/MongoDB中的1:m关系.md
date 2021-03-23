---
title: MongonDB中的1:m关系
date: 2018-04-28 13:15:05
desc:
tags:
---

在博客系统中用户(user)和博文(post)是1对多关系,在mongodb中我们怎么表述这种关系呢？在mongo中我们可以利用在多端保存1端的`ObjectId`来维持这种`1:m`关系,下面分别列出`user`和`post`的结构:
1. `user`的结构
``` json
{
    "_id" : ObjectId("5ae3caa35ece85125d2429f7"),
    "username" : "百思可乐1",
    "password" : "123456",
    "age" : 21,
    "updateAt" : ISODate("2018-04-28T01:11:54.698Z"),
    "createdAt" : ISODate("2018-04-28T01:11:54.698Z"),
    "__v" : 0
}
```

<!-- more -->



2. `post`的结构
```json
{
    "_id" : ObjectId("5ae3384594aa810f8d17f073"),
    "title" : "ruby",
    "body" : "sdasda",
    "author" : ObjectId("5ae3caa35ece85125d2429f7"),
    "__v" : 0
}

```


上面我们可以看到我们在`post`document里添加个一个`author`的字段里面保存了`user`的`ObjectId`.

于是我们就可以执行下面这些操作了:
1. 查询一个用户发布的所有文章
```js
  var user = db.user.findOne();
  var posts = db.post.find({author: user._id});
```

2. 查询一篇文章的作者
```js
  var post = db.post.findOne();
  var user = db.user.findOne({author: post.author });
```

3. 用户创建文章
```js
  var user = db.user.findOne();
  var posts = db.post.insert(
    {
      title: "ruby",
      body: "sdasda",
      author: user._id
    });
```