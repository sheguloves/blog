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

缓存主要分为服务端缓存和客户端缓存两大类。常见的CND、代理服务器缓存、反向代理服务器缓存都是指服务端缓存，原理就是让客户端请求走捷径，从离得最近的地方或者道路最畅通的地方获取请求资源；客户端缓存一般是指浏览器缓存。

#### 私有缓存

私有缓存只针对专有用户，所以不需要很大空间。Web浏览器中有内建的私有缓存--大多数浏览器都会将常用资源缓存在个人电脑的磁盘和内存中，如Chrome的缓存存放位置在`C:\Users\Your_Account\AppData\Local\Google\Chrome\User Data\Default`中的Cache文件夹和Media Cache文件夹，在chrome地址栏中输入`chrome:cache`即可查看当前的缓存内容

#### 公有缓存

公有缓存是特殊的共享代理服务器，被称为`缓存代理服务器`或`代理缓存`(反向代理的一种用途)。公有缓存会接受来自多个用户的访问，所以通过它能够更好的减少冗余流量。看下图

![share cache](/img/browser_cache/share_cache.png "browser first request")

#### 缓存处理流程
![cache progress](/img/browser_cache/cache_progress.png "cache progress")

对于前端开发人员来说，我们主要根浏览器打交道，所以可以将上图流程简化为：
![frontend cache progress](/img/browser_cache/cache_progress.png "frontend cache progress")

下面主要是讲浏览器缓存，也就是所谓的前端开发人员必须要非常了解的内容

### 浏览器请求流程

* 浏览器第一次请求
![browser first request](/img/browser_cache/http-request-first-time.png "browser first request")

* 浏览器再次请求
![browser first request](/img/browser_cache/http-request-same-site.png "browser first request")

### 浏览器缓存
#### HTML Meta标签控制缓存

浏览器缓存机制，其实主要就是HTTP协议定义的缓存机制（如： Expires、Cache-control等）。但是也有非HTTP协议定义的缓存机制，如使用HTML Meta 标签，Web开发者可以在HTML页面的<head>节点中加入<meta>标签，代码如下：

{% highlight html %}
<META HTTP-EQUIV="Pragma" CONTENT="no-cache">
{% endhighlight %}

上述代码的作用是告诉浏览器当前页面不被缓存，每次访问都需要去服务器拉取。使用上很简单，但只有部分浏览器可以支持，而且所有缓存代理服务器都不支持，因为代理不解析HTML内容本身。

#### HTTP头信息控制缓存

##### 新鲜度限值
浏览器通过缓存将服务器资源的副本保留一段时间，这段时间称为`新鲜度限值`。在这段时间内请求相同的资源不会再通过服务器。HTTP协议中`Cache-Control` 和 `Expires` 可以用来设置新鲜度的限值。`Cache-Control` 是HTTP1.1中新增的响应头，`Expires` 是HTTP1.0中的响应头。两者的作用是相同的，都是服务端用来约定和客户端的有效时间的，更推荐使用 `Cache-Control: max-age=xxx(秒)`。

#####  Cache-Control

值可以是public、private、no-cache、no-store、must-revalidate、proxy-revalidate、max-age
各个消息中的指令含义如下：
* public: 指示响应可被任何缓存区缓存。
* private: 指示对于单个用户的整个或部分响应消息，不能被共享缓存处理。这允许服务器仅仅描述当用户的部分响应消息，此响应消息对于其他用户的请求无效。
* no-cache: 表示必须先与服务器确认返回的响应是否被更改，然后才能使用该响应来满足后续对同一个网址的请求。因此，如果存在合适的验证令牌 (ETag)，no-cache 会发起往返通信来验证缓存的响应，如果资源未被更改，可以避免下载
* no-store: 用于防止重要的信息被无意的发布。在请求消息中发送将使得请求和响应消息都不使用缓存，完全不存下來。
* max-age: 指示客户机可以接收生存期不大于指定时间（以秒为单位）的响应。

##### Last-Modified/If-Modified-Since

`Last-Modified/If-Modified-Since`是指当有效期过后，check服务端文件是否更新的第一种方式，要配合Cache-Control使用。在HTTP请求响应头中，存在字段`Last-Modified`，根据名字就知道它表达的意思，当缓存有效期过后，HTTP请求时会添加字段`If-Modified-Since`发往服务器，`If-Modified-Since`的值就是`Last-Modified`的值。如果服务器资源没有更新，则相应HTTP304，从缓存中读取数据，如果资源更新，则返回HTTP200，同时通过响应头更新`Last-Modified`的值。

##### ETag/If-None-Match

既然已经有了`Last-Modified/If-Modified-Since`，为何还会出现`ETag/If-None-Match`？这是因为`Last-Modified/If-Modified-Since`有以下缺点：

* Last-Modified标注的最后修改只能精确到秒级，如果某些文件在1秒钟以内，被修改多次的话，它将不能准确标注文件的修改时间
* 如果某些文件会被定期生成，但有时内容并没有任何变化（仅仅改变了时间），但Last-Modified却改变了，导致文件没法使用缓存
* 有可能存在服务器没有准确获取文件修改时间，或者与代理服务器时间不一致等情形

`Etag`是服务器根据资源索引、大小和最后修改时间等进行Hash后得到的一个字符串，在请求中发送`If-None-Match`选项，值为上次请求得到的`Etag`的值，服务端会用最新的资源`Etag`值与`If-None-Match`值做比对，如果相同，则响应HTTP304，如果不相同，则响应HTTP200，同时更新`Etag`值。

### 用户行为与缓存

浏览器缓存行为还有用户的行为有关，请参考下图

![user action](/img/browser_cache/user_action.png "user action")

### 参考阅读
[透过浏览器看HTTP缓存](http://www.cnblogs.com/skylar/p/browser-http-caching.html)

[浏览器缓存机制浅析](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551769&idx=1&sn=3a422455b5cc240f8625842d31d81ab8&chksm=8025afd8b75226cec68e1e0e4b36334bb1d88e51a7fdba0cf7884d9f1d6597e4e4475cffaa38&scene=0&key=14dedca505cc3f43c6a4d233713db6622f46ef4cb8b38560a2d86c3125344594841f722c72f1c9440770400abeecd9f41d6f3ed4289683ee11923ba5ab54c5fedcd124869585d18c38bf21a28c6c8e71&ascene=0&uin=MzA3NDk0NjAw&devicetype=iMac+MacBookPro12%2C1+OSX+OSX+10.12.2+build(16C67)&version=12020010&nettype=WIFI&fontScale=100&pass_ticket=op6p%2FRIGPnVS0DbauBZRlcVPy964Xj6glTzdnemU7TgmVLrasHaK4vQt4OJECOMB)

[浏览器 HTTP 协议缓存机制详解](http://www.cnblogs.com/520yang/articles/4807408.html)

[浅谈Web缓存](http://web.jobbole.com/85243/)

[作为前端应当了解的Web缓存知识](http://www.cnblogs.com/dojo-lzz/p/5515839.html)


