---
layout: post
title: Angular with gulp and browserify
description: "Build Angularjs app with gulp and browserify"
modified: 2016-02-18
tags: [Angular, Angularjs, gulp, browserify]
---

### Background
一直在关注`Angular`, `gulp`, `browserify`,正好公司项目更新,就用这些新的东西来做更新我们的项目.项目地址：[https://github.com/sheguloves/monitor](https://github.com/sheguloves/monitor)
下面是在构建项目的过程中遇到的一些问题,记录下来,以免忘掉.

<!--more-->

### ng-route | angular-route
the folder structure like:

{% highlight yaml %}
example
    public
    src
        app
            controller
                login.js
                dashboard.js
                index.js
            partials
                login.html
                dashboard.html
            service
                index.js
                service.js
            main.js
        assets
            fonts
                Arial.ttf
            images
                1.png
                2.png
            styles
                styles.css
        data
            model.js
        env
            prod.js
            dev.js
    index.html
{% endhighlight %}