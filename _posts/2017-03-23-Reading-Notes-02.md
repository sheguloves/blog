---
layout:     post
title:      "读书笔记 - 02"
subtitle:   "读Javascript经典面试笔记"
date:       2017-03-08
author:     "Felix Xi"
header-img: "img/night.jpg"
catalog: true
tags:
    - 读书笔记
    - 前端面试
---


## 读书笔记 - 02

#### instanceof
instanceof是检查对象的原型链中的constructor，所以一个对象可以是多种类型的对象实例
```
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}
var mycar = new Car('Honda', 'Accord', 1998);
var a = mycar instanceof Car;    // returns true
var b = mycar instanceof Object; // returns true
```

#### typeof, Hoisting
Why
```
 var y = 1;
  if (function f(){}) {
    y += typeof f;
  }
  console.log(y); // 1undefined
```
> The confusion here is that the syntax function fName() {} is ambiguous and depends on context.
>
> Alone, it is a function ***declaration*** and is therefore hoisted. But in certain contexts it is a (named) function ***expression*** and therefore isn't.
>
> This is why you see IIFEs declared as (function() {..})() or !function() {..}() to force them into being function ***expressions*** rather than ***declarations***.
>
> The important thing about named function expressions is that their name is essentially local to the function itself, and is not accessible outside that function. This is why += f appends undefined in your code.
>
> 以上答案来自 [Stack Overflow](http://stackoverflow.com/questions/38159258/function-hoisting-in-javascript-from-condition-statement)


### 未完待续
https://github.com/nishant8BITS/123-Essential-JavaScript-Interview-Question
