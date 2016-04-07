---
layout:     post
title:      "Angular Interpolation and Data-Binding"
subtitle:   "Angular expression binding to arbitrary attributes"
date:       2016-04-07
author:     "Felix Xi"
header-img: "img/banner-6.jpg"
tags:
    - Angular
    - Data Binding
---

### Background

Agnualr 给我们提供了非常方便的数据双向绑定，这也是很多人使用 Angular 感到很方便的地方。

最简单的方式：

```
/* template html */
<div ng-controller="mainCtrl as ctrl">{{ctrl.description}}</div>

/* controller.js */
angular.module('app', []).controller('mainCtrl', function() {
    this.description = "This is a description";
});

```

通过数据绑定，我们将所有的数据放在 `controller` 中, 而 template 只负责显示，数据的更新都交给数据绑定来实现。

### Binding to boolean attributes

我们可以将数据绑定给我们的自定义组件 `directive` 或 `component` 中自定义的属性，而如果要将数据绑定给原生的 `html` 的属性的时候，为了避免 `html` 预编译(自己理解)和其他原因造成的错误，`Angular` 提供了 `ng` 前缀来避免这类的错误，比如，如果我们要给 `button` 绑定 `disabled` 属性

```
Disabled: <input type="checkbox" ng-model="isDisabled" />
<button disabled="{{isDisabled}}">Disabled</button>
```

在这种情况下，`button` will always disabled. 类似情况的属性还有： `required`, `selected`, `checked`, `readOnly` , and `open`。
为了让 `html` 按照我们预想的那样，我们可以加上 `ng` 前缀。

```
Disabled: <input type="checkbox" ng-model="isDisabled" />
<button ng-disabled="isDisabled">Disabled</button>
```

### `ngAttr` for binding to arbitrary attributes

对于有些 `svg` 属性，browser 会对某些属性的合法值挑剔。例如：

```
<svg>
  <circle cx="{{cx}}"></circle>
</svg>
```

运行，我们会得到错误： `Error: Invalid value for attribute cx="{{cx}}"`，如何处理这种错误呢， Angular 提供了前缀 `ngAttr` 或 `ng-attr`.

```
<svg>
  <circle ng-attr-cx="{{cx}}"></circle>
</svg>
```

如果 `svg` 的属性存在驼峰式命名结构，可以用下划线来表示绑定。例如属性 `viewBox`：

```
<svg ng-attr-view_box="{{viewBox}}">
</svg>
```

以上内容都是 Angular 官方网站上的介绍，可以参考[链接](https://docs.angularjs.org/guide/interpolation)

### Key point

不经意间在在搜索的时候发现一个比较有意思的解决以上问题的方法：

```
// javascript
angular.module('yourmodule.directives', [])
    .directive('ngX', function() {
        return function(scope, element, attrs) {
            scope.$watch(attrs.ngX, function(value) {
                element.attr('x', value);
            });
        };
    });

// html
<circle ng-x="{{ x }}" y="0" r="5"></circle>

```

话说这才是本文的精华所在，虽然很短。
这种解决方法给我们一个很好的启发，脑洞大开的我们可以很好的借鉴一下。[参考链接](https://github.com/angular/angular.js/issues/1050)