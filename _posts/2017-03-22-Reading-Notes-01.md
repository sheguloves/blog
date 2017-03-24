---
layout:     post
title:      "读书笔记 - 01"
subtitle:   "由WebSocket引发的探索"
date:       2017-03-08
author:     "Felix Xi"
header-img: "img/night.jpg"
catalog: true
tags:
    - WebSocket
    - 读书笔记
    - 前端面试
---

### WebSocket
* WebSocket是协议
* WebSocket既依赖于浏览器支持，也依赖于服务器和代理（如果需要经过代理的话）支持

> The WebSocket Protocol is an independent TCP-based protocol.  Its
   only relationship to HTTP is that its handshake is interpreted by
   HTTP servers as an Upgrade request.
> #### The handshake from the client looks as follows:
> ```
> GET /chat HTTP/1.1
> Host: server.example.com
> Upgrade: websocket
> Connection: Upgrade
> Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
> Sec-WebSocket-Protocol: chat, superchat
> Sec-WebSocket-Version: 13
> Origin: http://example.com
> ```
> The request MUST include a header field with the name ***Origin*** if the request is coming from a browser client
>
> The opening handshake is intended to be compatible with HTTP-based
   server-side software and intermediaries, so that a single port can be
   used by both HTTP clients talking to that server and WebSocket
   clients talking to that server.  To this end, the WebSocket client's
   handshake is an HTTP Upgrade request
>
> #### The handshake from the server looks as follows:
> ```
> HTTP/1.1 101 Switching Protocols
> Upgrade: websocket
> Connection: Upgrade
> Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
> Sec-WebSocket-Protocol: chat
> ```
> These fields are checked by the WebSocket client for scripted pages.
   If the |Sec-WebSocket-Accept| value does not match the expected
   value, if the header field is missing, or if the HTTP status code is
   not 101, the connection will not be established, and WebSocket frames
   will not be sent.
>
> #### Http problems:
> * The server is forced to use a number of different underlying TCP connections for each client: one for sending information to the client and a new one for each incoming message
> * The wire protocol has a high overhead, with each client-to-server message having a HTTP header
> * The client-side script is forced to maintain a mapping from the outgoing connections to the incoming connection to track replies

以上文档取自[[RFC6455](https://tools.ietf.org/html/rfc6455)]

### (短)轮询、长轮询、长连接
* (短)轮询就是以一定的时间间隔不断的发请求去query server是否数据有更新。所以这种轮询比较耗客户端与服务端的资源，同时占带宽。
* 长轮询是客户端发请求，server端检查如果没有数据更新，就不返回，hold住当前的请求，只有当有数据更新了再返回，客户端收到更新的数据。如果timeout，客户端会收到请求过期的错误，再接着发请求。好处就是没有数据更新的情况下，减少发送HTTP请求的个数，减少带宽占用，减少建立TCP连接的资源消耗。通过前端后端一起实现。
* 长连接是指客户端与服务器建立TCP连接后，在一段时间内不会关闭这个TCP连接，其他的HTTP请求会复用这个TCP连接，从而减少建立、关闭TCP连接的资源消耗。由Framework来实现，HTTP1.1中支持，需要客户端、服务器同时设置，request head中需要设置Connection: keep-alive。

### 新想到的问题及未解决的问题
* websocket如何处理成千上万的连接同时发生，并发量到底多高
* 代理跨域如何解决登陆、跳转问题
* 浏览器并发http请求的个数，例如chrome

