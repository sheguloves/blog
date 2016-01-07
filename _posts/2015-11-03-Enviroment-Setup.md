---
layout: post
title: Environment Setup
description: "This is a notes for development environment setup"
modified: 2015-11-08
tags: [CSS]
---

### Config git proxy
```
git config --global http.proxy http://<host>:<port>
git config --global https.proxy http://<host>:<port>
```

### npm proxy
```
npm config set proxy http://<host>:<port>,
npm config set https-proxy http://<host>:<port>
```
### bower proxy
edit .bowerrc file and add the following code
```
{
  ...
  "proxy":"http://<user>:<password>@<host>:<port>", // "proxy":"http://<host>:<port>",
  "https-proxy":"http://<user>:<password>@<host>:<port>" // "https-proxy":"http://<host>:<port>"
}
```