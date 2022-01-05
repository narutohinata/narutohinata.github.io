---
title: CSS选择器
date: 2015-09-010 09:26:15
desc:  CSS里的那些选择器
tags:
  - css
---

#### 类选择器

```html
  <div class="wrapper"></div>
```

```css
 .wrapper{
   color: red;
 }
```
给类名为`wrapper`的元素的字体颜色设置为红色。

<!-- more -->

#### ID选择器

```html
  <div id="banner"></div>
```

```css
 #banner{
   background: url("http://chromer.com/images/banner.jpg")
 }
```
给ID为`banner`的元素设置背景图片

#### 标签选择器
```html
 <p> Hello World </p> 
```

```css
 p {
   color: red;
 }
```
给标签为`p`的元素的字体颜色设置为红色

#### 属性选择器

```html
  <h1 title="title"></h1>
```
```css
  h1[title]{
    font-size: 48px;
  }
```
给所有包含`title`属性的`h1`标签的字体大小设置为`48px`
除此之外属性选择器还有精确匹配和模糊匹配:

选择器                   | 功能                                    |
:-----:                 | :-----:                                |
[attribute=value]       | 匹配attribute的值等于value的元素          |
[attribute~=value]      | 匹配attribute的值至少有一个等于value的元素  |
<!-- [attribute&#124;=value] | 匹配attribute的值为“value”或是以“value-”为前缀的元素| -->
[attribute^=value]      | 匹配attribute的值是以"value"开头的元素|
[attribute$=value]      | 匹配attribute的值是以“value”结尾的元素 |
[arribute*=value]       | 匹配attribute的值是包含“value”的元素 |

#### 群组选择器
```html
  <h1>H1</h1>
  <h2>H2</h1>
  <h3>H3</h1>
  <h4>H5</h1>
```

```css
  h1, h2, h3, h4{
    color: #FFFFFF;
  }
```
设置标签`h1`、`h2`、`h3`、`h4`的颜色为`#FFFFFF`

#### 相邻兄弟选择器

```html
  <div>
    <h1>H1</h1>
    <p>这是内容</p>
    <p>哈哈</p>
  </div>
```

```css
  h1 + p {
    color: red;
  }
```
设置紧跟着`h1`标签后面的`p`元素的文字颜色为`red`

#### 通用兄弟选择器

和上面类似但是匹配的是和指定元素后面的兄弟元素,不一定是紧跟着指定元素之后的

```html
  <div>
    <h1>H1</h1>
    <p>这是内容</p>
    <p>哈哈</p>
  </div>
```

```css
  h1 ~ p {
    color: red;
  }
```
设置`h1`标签后面的所有`p`元素的文字颜色为`red`

#### 子选择器

```html
  <div>
    <h1> hello </h1>
  </div>
```
```css
  div > h1 {
    color: blue;
  }
```