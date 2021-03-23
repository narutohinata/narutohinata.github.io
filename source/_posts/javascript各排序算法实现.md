---
title: javascript各排序算法实现
date: 2018-06-06 11:14:34
desc: javascript各排序算法的实现
tags: 算法
categories: javascript
---
1. 冒泡排序(普通版)
```js
function bubble_sort(data) {
  for (let i = data.length - 1; i > 0 ; i-- ){
    for (let j = 0; j < i; j++) {
      if (data[j] > data[j+1]) {
        let tmp = data[j+1];
        data[j+1] = data[j];
        data[j] = tmp;
      }      
    }
  }
}
```
2. 冒泡排序（改良版1)
```js
function bubble_sort1(data) {
  for (let i = data.length - 1; i > 0 ; i-- ){
    let tag = true;
    for (let j = 0; j < i; j++) {
      if (data[j] > data[j+1]) {
        let tmp = data[j+1];
        data[j+1] = data[j];
        data[j] = tmp;
        tag = false;
      }
    }
    if (tag) {
      return;
    }
  }
}
```
这里说下改良版的思路，上一个版本没有记录上一趟冒泡的排序状态，导致必须执行`n-1`趟排序。考虑最好的情况，如果一开始序列就是有序的，如果使用第一种排序的话，导致可以`O(n)`的解决的问题花了`O(n^2)`的时间了，这显然是不合适的，于是我们在每一趟排序开始前，设置一个表示序列是否有序的状态量(`let tag`)，并认为一开始是有序的(`true`)，然后在交换排序过程中一旦发生了交换就把状态量设置为无序状态（`false`)。然后在进行下一趟排序的时候检查这个状态量，如果是`true`则说明序列此时已经有序无需进行下一趟了直接返回就行了，如果为`false`则表示序列此时还是无序状态则需要进行下一趟排序。
