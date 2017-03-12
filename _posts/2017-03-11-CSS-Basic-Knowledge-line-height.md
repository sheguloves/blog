---
layout:     post
title:      "CSS基础知识"
subtitle:   "CSS基础知识 - line-height"
date:       2017-03-12
author:     "Felix Xi"
catalog: true
header-img: "img/frogs.jpg"
tags:
    - CSS
---

### Background

CSS属性中，line-height一直困扰着我，终于可以抽出时间做一次line-height值的基础研究，本文都是一些基本的知识，记下来方便自己以后查看，不涉及任何深入的内容，可能会有一些差错，先记下来再说。

### line-height基本知识

line-height可以接受合法值为：`px/em`、`normal`、`inherit`、`百分比`、`数值`。

* px/em：限值line-height为固定值，当然，如果设置为`em`时，line-height的值为`font-size`的值 * em的值
* normal：为浏览器设定的值，一般为 1～1.2 （不是很确定）
* inherit：继承父节点的line-height的值
* 百分比：line-height的值为 `font-size`的值 * 百分比
* 数值：line-height的值为 `font-size`的值 * 数值

### 百分比与数值的区别

先看下图：

![line-height](/img/line-height.jpg "line-height")

是否看出了区别？由图中我们可以看到，当line-height的值为百分比时，子元素的font-size不管怎么变化，line-height的值都是父元素的font-size * 百分比，而当line-height的值为数值时，子元素的line-height的值则是子元素的font-size * 数值。

### 更多深入研究参考

下面是ling-height的一些深入研究参考：

* [css行高line-height的一些深入理解及应用](http://www.zhangxinxu.com/wordpress/2009/11/css%E8%A1%8C%E9%AB%98line-height%E7%9A%84%E4%B8%80%E4%BA%9B%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3%E5%8F%8A%E5%BA%94%E7%94%A8/)
* [CSS深入理解vertical-align和line-height的基友关系](http://www.zhangxinxu.com/wordpress/2015/08/css-deep-understand-vertical-align-and-line-height/)
