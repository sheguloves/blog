### 正则表达式

#### 贪婪匹配
正则表达式默认是贪婪匹配的，也就是匹配尽可能多的字符。举例如下，匹配出数字后面的`0`：

```
var re = /^(\d+)(0*)$/;
re.exec('102300'); // ['102300', '102300', '']
```

由于`\d+`采用贪婪匹配，直接把后面的`0`全部匹配了，结果`0*`只能匹配空字符串了。

必须让`\d+`采用非贪婪匹配(也就是尽可能少匹配)，才能把后面的`0`匹配出来，加个`?`就可以让`\d+`采用非贪婪匹配：

```
var re = /^(\d+?)(0*)$/;
re.exec('102300'); // ['102300', '1023', '00']
```

#### 全局搜索

Javascript的正则表达式还有几个特殊的标志，比如`g`，表示全局匹配：

```
var r1 = /test/g;
// 等价于:
var r2 = new RegExp('test', 'g');
```

全局匹配可以多次执行`exec()`方法来搜索一个匹配的字符串。当我们指定`g`标志后，每次运行`exec()`，正则表达式本身会更新`lastIndex`属性，表示上次匹配到的最后索引：

```
var s = 'JavaScript, VBScript, JScript and ECMAScript';
var re=/[a-zA-Z]+Script/g;

// 使用全局匹配:
re.exec(s); // ['JavaScript']
re.lastIndex; // 10

re.exec(s); // ['VBScript']
re.lastIndex; // 20

re.exec(s); // ['JScript']
re.lastIndex; // 29

re.exec(s); // ['ECMAScript']
re.lastIndex; // 44

re.exec(s); // null，直到结束仍没有匹配到
```

全局匹配类似搜索，因此不能使用`/^...$/`，那样只会匹配一次。


### CORS

##### Options

W3C的CORS规范将跨域资源请求划分为两种类型：简单请求
  CORS规范将`GET`、`HEAD`、`POST`这三个HTTP方法视为**简单HTTP方法**，而将请求报头`Accept`、`Accept-Language`、`Content-Language`以及采用如下三种媒体类型的报头Content-type称为**简单请求报头**
  * application/x-www-form-urlencoded
  * multipart/form-data
  * text/plain

请求的报头包含两种类型，一类是通过浏览器自动生成的报头，另一种则是由JavaScript程序自行添加的报头（比如调用XMLHttpReuqest的setRequestHeader方法可以为生成的Ajax请求添加任意报头），后者被称为自定义报头。

CORS规范将符合如下条件的跨域资源请求划分为简单请求：**请求采用简单HTTP方法，并且其自定义请求报头为空活着所有自定义报头均为简单报头**。之所以作如此划分是因为具有这些特征的请求不是以更新（添加、修改和删除）资源为目的，服务端对请求的处理不会导致自身维护资源的改变。

对于简单跨域资源请求来说，浏览器将两个步骤（取得授权和获取资源）合二为一，由于不涉及资源的改变，所以不会带来任何副作用（Side Effect）。如果针对请求的处理过程会涉及对资源的改变，这样做就会有问题了。按照CORS规范的规定，浏览器应该采用一种被称为**预检（Preflight Request）**的机制来完成非简单跨域资源请求。

所谓预检机制就是说浏览器在发送真正的跨域资源请求前，先发一个预检请求（Preflight Request）。预检请求为一个采用`HTTP-OPTIONS`方法的请求，这是一个不包含主体的请求，同时用户凭证相关的报头也会被剔除。基于真正资源请求的一些辅助授权的信息会包含在此预检请求的相应报头中。除了代表请求页面所在站点的`Origin`报头之外，如下所示的是两个典型的请求报头。
* Access-Control-Request-Method: 真正跨域资源请求采用的HTTP方法
* Access-Control-Request-Headers: 真正跨域资源请求所携带的自定义报头
资源的提供者在接收到预检请求之后，根据其提供的相关报头进行授权检验，具体的检验逻辑即包括确定请求站点是否值得信任，以及请求采用HTTP方法和自定义报头是否被允许。如果预检请求没有通过授权检验，资源提供者一般会返回一个状态为
`400， Bad Reuqest`的响应。反之则会返回一个状态为`200， OK`的响应，授权相关信息会包含在响应报头中。除了上面介绍的`Access-Control-Allow-Origin`和`Access-Control-Expose-Headers`报头之外，预检请求的响应还具有如下3个典型的报头:
* Access-Control-Allow-Methods：跨域资源请求允许采用的HTTP方法列表。
* Access-Control-Allow-Headers：跨域资源请求允许携带的自定义报头列表。
* Access-Control-Max-Age：浏览器可以将响应结果进行缓存的时间（单位为秒），这样可以让浏览器避免频繁地发送预检请求。

