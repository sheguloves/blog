---
layout: post
title: Javascript use byte images from java servlet
description: "Javascript use byte images from java servlet"
modified: 2016-01-27
tags: [Javascript, Java, Image Byte]
---

## How to get the correct "src" value of ```<img>``` by ajax call from java servlet

### Javascript

{% highlight javascript %}

var $ = require("jquery");

$.ajax({
    url: logoUrl,
    type: "GET",
    contentType: "image/jpg",
    dataType: "text",
    success: function success(response) {
        var logo = "data:image/png;base64," + response;
        $("#img").attr("src", logo);
    }
});

{% endhighlight %}

<!--more-->

### Java servlet

{% highlight java %}

import org.apache.commons.codec.binary.Base64;

httpResponse.setContentType("image/jpg");
byte[] encodedBytes = Base64.encodeBase64(/* image.getByteArray */byteArray);
httpResponse.getWriter().println(new String(encodedBytes));
httpResponse.getWriter().flush();
httpResponse.getWriter().close();

{% endhighlight %}
