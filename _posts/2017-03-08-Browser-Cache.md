---
layout:     post
title:      "Browser Cache"
subtitle:   "Browser Cache Notes"
date:       2017-03-08
author:     "Felix Xi"
header-img: "img/night.jpg"
catalog: true
tags:
    - Html
---

### Background

一直以来做的都是面向企业的web前端，所以对首页加载及performance问题不是特别关注，更多的时间是放在功能实现上。随着工作年限的增长，对知识面的要求越来越广，所以抽时间补了一下这方面的知识。但是，毕竟年纪大了，不在实际工作中应用，看过很快就忘记了。网络上的好文章很多，怕日子长了，源文章不在了，就想自己写下来，方便以后查阅，同时，也增强一下记忆。

### 缓存的种类

缓存主要分为服务端缓存和客户端缓存两大类。常见的CND、代理服务器缓存、反向代理服务器缓存都是指服务端缓存，原理就是让客户端请求走捷径，从离得最近的地方或者道路最畅通的地方获取请求资源；客户端缓存一般是指浏览器缓存。下面主要是讲浏览器缓存，也就是所谓的前端开发人员必须要非常了解的内容

### 浏览器缓存
#### HTML Meta标签控制缓存

浏览器缓存机制，其实主要就是HTTP协议定义的缓存机制（如： Expires； Cache-control等）。但是也有非HTTP协议定义的缓存机制，如使用HTML Meta 标签，Web开发者可以在HTML页面的<head>节点中加入<meta>标签，代码如下：

{% highlight html %}
<META HTTP-EQUIV="Pragma" CONTENT="no-cache">
{% endhighlight %}

上述代码的作用是告诉浏览器当前页面不被缓存，每次访问都需要去服务器拉取。使用上很简单，但只有部分浏览器可以支持，而且所有缓存代理服务器都不支持，因为代理不解析HTML内容本身。

#### HTTP头信息控制缓存



