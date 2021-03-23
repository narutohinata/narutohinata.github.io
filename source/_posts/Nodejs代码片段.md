---
title: Nodejs代码片段
date: 2018-05-08 17:38:24
desc:
tags: Nodejs 代码片段 Tips
---

# Nodejs `Base64`编码和解码
1. Base64编码
```js
var str = "我爱js";
var decode_str = Buffer.from(str).toString('base64') // > '5oiR54ixanM='
// 或者这样：new Buffer(str).toString('base64') 【已弃用】
```

2. Base64解码
```js
var decode_str = '5oiR54ixanM=';
var str = Buffer.from(decode_str, 'base64').toString() // > 我爱js
```
