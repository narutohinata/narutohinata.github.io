---
title: CommonJS是如何处理循环依赖的
date: 2020-01-01 13:21:32
desc:
tags:
  - commonjs
---

## Commonjs是如何解决循环依赖

要回答这个问题我们先了解下`cjs` 的模块加载流程。当加载一个`cjs` 模块的时候会先缓存模块对象 `Module._cache[filename] = module`  然后再加载模块 `module.load(filename)` 。基于这个模块加载流程我们分析下面这个循环依赖的代码:

```javascript
// A.js
exports.name = "A"
const B = require("B")
console.log(B)
exports.data = "Data-A"
```



```javascript
// B.js
exports.name = "B"
const A = require("A")
console.log(A)
exports.data = "Data-B"

```

当我们执行`node A.js` 会输出什么？

首先系统会先缓存A模块然后在加载A模块（也就是执行A里面的代码）， 然后当A代码执行到第二行 系统会去require B模块，同A模块一样,系统也是先缓存B模块 然后加载B模块，当B执行到第二行`const A = require("A")`的时候 系统会去加载A模块，但此次我们发现没有？A模块已经在缓存中了，只不过这个A模块还没加载完成 处于一种部分加载的状态。也就是B这里得到的A是个部分值(也就是A.js里在require('B')上方exports出来的值 `{ name: "A" } ` )。然后B打印A的部分值`{ name: "A" }` 然后执行最后一行代码后，module B模块加载完毕。此时执行权回到模块A，A这里得到的B是个完全加载的模块`{ name: "B", data: "Data-B" }` 



于是我们可以这样回答这个问题，cjs的模块加载流程是先缓存再加载。基于这个原理我们很容易发现，当出现循环依赖，我们拿到的是一个部分加载模块的部分值。