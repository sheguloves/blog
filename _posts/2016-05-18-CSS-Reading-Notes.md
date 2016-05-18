---
layout:     post
title:      "CSS Reading Notes"
subtitle:   "Blog Reading Notes - css相对定位relative绝对定位absolute系列"
date:       2016-05-18
author:     "Felix Xi"
header-img: "img/all-2.jpg"
tags:
    - CSS
---

### Background
最近在读张鑫旭大神的blog，受益良多，将我认为非常独特的观点记录下来，在以后使用的过程中参考。原文系列：[css相对定位relative绝对定位absolute系列](http://www.zhangxinxu.com/wordpress/2011/08/css%E7%9B%B8%E5%AF%B9%E5%AE%9A%E4%BD%8Drelative%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8Dabsolute%E7%B3%BB%E5%88%97%EF%BC%88%E5%9B%9B%EF%BC%89/)

### Notes

* 想重构高质量的页面，少用绝对定位布局！
* 控制元素显示与隐藏才是 `absolute` 属性的正业所在.
* `position:absolute` 与 `float:left` 是近亲，两者有两大共性：包裹性，破坏性。

### Thinking

* 文章中说到 `absolute` 的正业为控制元素显示与隐藏，定位相关的实现更推荐兼容性更好的 `margin`， 于是我就想到 `margin` 的使用方法，同时验证一下 `margin` 造成的 `DOM` 变化。

  > `position: static` 的元素设置负的 `margin-top` 会将整个元素上移，同时缩小 `static` 定位下占用的空间，父元素高度减小。同理，如果为正的 `margin-top` 值， 就会使父元素高度撑开至完全容得下子元素。而`position: relative` 元素设置负的 `top` 也会使元素上移，但是仍然会保留 `static` 情况下的占用空间，父元素的高度不会减小，同理，`top` 为正值的情况下，元素下移，但父元素的高度不变
