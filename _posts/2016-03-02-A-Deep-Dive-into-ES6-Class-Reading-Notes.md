---
layout:     post
title:      "A Deep Dive into ES6 Class Reading Notes"
subtitle:   "A Deep Dive into ES6 Class Reading Notes"
date:       2016-02-18
author:     "Felix Xi"
header-img: "img/banner-3.jpg"
tags:
    - Javascript
    - Class
    - ES6
---

### Background
最近一直忙于项目的setup，今天基础框架基本搭好，终于有时间来弥补一下这段时间的Reading。此文就是今天阅读过程中看到的一些不太了解得地方，记下来，供将来参考。

[原文地址](http://www.sitepoint.com/object-oriented-javascript-deep-dive-es6-classes/)

### Notes

  > * When a function is associated with a class or object, we call it a “method”.
  > * When an object is created from a class, that object is said to be an “instance” of the class.

### Subclasses (子类)

#### Inherit to Avoid Duplication

没有类的情况下，我们的代码是这样写的
{% highlight javascript %}
class Employee {
  constructor(firstName, familyName) {
    this._firstName = firstName;
    this._familyName = familyName;
  }

  getFullName() {
    return `${this._firstName} ${this._familyName}`;
  }
}

class Manager {
  constructor(firstName, familyName) {
    this._firstName = firstName;
    this._familyName = familyName;
    this._managedEmployees = [];
  }

  getFullName() {
    return `${this._firstName} ${this._familyName}`;
  }

  addEmployee(employee) {
    this._managedEmployees.push(employee);
  }
}
{% endhighlight %}

有了类，我们可以用Class的继承去除重复的代码

{% highlight javascript %}
class Employee {
  constructor(firstName, familyName) {
    this._firstName = firstName;
    this._familyName = familyName;
  }

  getFullName() {
    return `${this._firstName} ${this._familyName}`;
  }
}

class Manager extends Employee {
  constructor(firstName, familyName) {
    super(firstName, familyName);
    this._managedEmployees = [];
  }

  addEmployee(employee) {
    this._managedEmployees.push(employee);
  }
}
{% endhighlight %}

#### IS-A and WORKS-LIKE-A

  > 有一个设计法则来帮助我们决定是否要使用继承。继承应该总是模拟`IS-A`和`WORKS-LIKE-A`这样的关系。正如上面我们看到的，manager `is a`并且`works like a`特殊的employee，那么我们在任何使用superclass实例的地方都可以用subclass的实例代替，并且一切都应该work，但是在某些情况下，也会违背这个法则，比如`Rectangle`和`Square`.

{% highlight javascript %}
class Rectangle {
  set width(w) {
    this._width = w;
  }

  get width() {
    return this._width;
  }

  set height(h) {
    this._height = h;
  }

  get height() {
    return this._height;
  }
}

// A function that operates on an instance of Rectangle
function f(rectangle) {
  rectangle.width = 5;
  rectangle.height = 4;

  // Verify expected result
  if (rectangle.width * rectangle.height !== 20) {
    throw new Error("Expected the rectangle's area (width * height) to be 20");
  }
}

// A square IS-A rectangle... right?
class Square extends Rectangle {
  set width(w) {
    super.width = w;

    // Maintain square-ness
    super.height = w;
  }

  set height(h) {
    super.height = h;

    // Maintain square-ness
    super.width = h;
  }
}

// But can a rectangle be substituted by a square?
f(new Square()); // error
{% endhighlight %}

当然，上诉的情况毕竟是比较特别的情况，所以Unit Test是很重要的

#### Beware Overuse (注意不要过度使用)

  > 我们可以发现，Class和继承带来的这些便利及功能是如此的诱人，即使作为一个有经验的开发者亦是如此。但是同样，继承也带来一些缺陷。回想当初，我们确保数据或者状态（state）的合法是通过操作很少的特定的functions来实现，但是当我们用到继承，我们增加了很多可以直接操作数据或者状态的functions，那么这些额外的functions也当然也要对数据合法性负责。如果太多的functions可以直接操作数据，那么这些数据将变得跟global变量一样差劲。太多的继承削弱了数据封装性并且会很难保证数据正确并切会难以重用。
  > 然而我们有另外一种方式来去除重复代码，我们称之为“composition”。下面的代码使用了组合而非继承来实现去除重复代码：

{% highlight javascript %}
class Employee {
  constructor(firstName, familyName) {
    this._firstName = firstName;
    this._familyName = familyName;
  }

  getFullName() {
    return `${this._firstName} ${this._familyName}`;
  }
}

class Group {
  constructor(manager /* : Employee */ ) {
    this._manager = manager;
    this._managedEmployees = [];
  }

  addEmployee(employee) {
    this._managedEmployees.push(employee);
  }
}
{% endhighlight %}

### 为什么说ES6的class more than sugar

#### Static Properties Are Inherited

{% highlight javascript %}
// ES5
function B() {}
B.f = function () {};

function D() {}
D.prototype = Object.create(B.prototype);

D.f(); // error
{% endhighlight %}

{% highlight javascript %}
// ES6
class B {
  static f() {}
}

class D extends B {}

D.f(); // ok
{% endhighlight %}

#### Built-in Constructors Can Be Subclassed

{% highlight javascript %}
// ES5
function D() {
  Array.apply(this, arguments);
}
D.prototype = Object.create(Array.prototype);

var d = new D();
d[0] = 42;

d.length; // 0 - bad, no array exotic behavior
{% endhighlight %}

{% highlight javascript %}
// ES6
class D extends Array {}

let d = new D();
d[0] = 42;

d.length; // 1 - good, array exotic behavior
{% endhighlight %}

#### Miscellaneous

  > There’s a small assortment of other, probably less significant differences. Class constructors can’t be function-called. This protects against forgetting to invoke constructors with `new`. Also, a class constructor’s `prototype` property can’t be reassigned. This may help JavaScript engines optimize class objects. And finally, class methods don’t have a `prototype` property. This may save memory by eliminating unnecessary objects.