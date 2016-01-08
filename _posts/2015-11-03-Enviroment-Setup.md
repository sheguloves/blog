---
layout: post
title: Environment Setup
description: "This is a notes for development environment setup"
modified: 2015-11-08
tags: [environment]
---

### Config git proxy

{% highlight Bash shell scripts %}
git config --global http.proxy http://<host>:<port>
git config --global https.proxy http://<host>:<port>
{% endhighlight %}

<!--more-->

### npm proxy

{% highlight Bash shell scripts %}
npm config set proxy http://<host>:<port>,
npm config set https-proxy http://<host>:<port>
{% endhighlight %}

### bower proxy

edit .bowerrc file and add the following code
{% highlight Bash shell scripts %}
{
  ...
  "proxy":"http://<user>:<password>@<host>:<port>", // "proxy":"http://<host>:<port>",
  "https-proxy":"http://<user>:<password>@<host>:<port>" // "https-proxy":"http://<host>:<port>"
}
{% endhighlight %}
