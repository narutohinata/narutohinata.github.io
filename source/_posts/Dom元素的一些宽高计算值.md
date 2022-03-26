---
title: Dom元素的一些宽高计算值
date: 2019-03-03 14:45:41
desc:
tags:
  - css
  - dom
---
# Dom元素的一些宽高计算值
## HTMLElement.offsetWidth/Height
`HTMLElement.offsetWidth/Hight` 是一个只读属性,  返回一个元素的布局宽度/高度。一般来说(各个浏览器的实现可能不一致) offsetWidth/Height是测量包含元素的边框(border)、内边距(padding)、滚动条(scrollbar)、以及CSS设置的size值。

*  offsetWidth = content.width + paddingLeft + paddingRight + borderLeft + borderRight + verticalScrollbarWidth
* offsetHeight = content.width + paddingTop + paddingBottom + borderTop + borderBottom + horizontalScrollbarWidth

## Element.clientWidth/Height
`Element.clientWidth/Element.clientHeight` 属性表示一个元素的内部宽度/高度。该属性包含内边距padding,但是不包含边框border、外边距margin以及滚动条。 也就是说:
* Element.clientWidth = contentWidth + paddingLeft + paddingRight
* Element.clientHeight = contentHeight + paddingTop + paddingBottom

值得注意的是内联元素(inline element)以及没有css样式的原始的clientWidth/clientHeight的属性值为0

## Element.scrollWidth/Height
