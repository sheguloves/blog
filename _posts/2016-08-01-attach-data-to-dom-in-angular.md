---
layout:     post
title:      "Attach Data to DOM in Angular"
subtitle:   "Attach Data to DOM and Remove Attribute in Angular"
date:       2016-08-01
author:     "Felix Xi"
header-img: "img/code-4.jpg"
tags:
    - Angular
---

#### Background

在开发中，有时候我们需要对`DOM`进行一些操作，比如拖拽，在监听拖拽事件的`callback`中，我们为了让 drag 组件与 drop 组件解耦，往往需要被拖拽的`DOM`携带一些数据，这样我们就可以将拖拽的`DOM`做成一个单独的组件，drop 组件也只需要根据`callback`中`DOM`携带的数据打交道，从而实现解耦。

#### Implement

在 Angular 中，我们经常使用双向绑定，那么我们就可以将要传送的数据绑定在`DOM`上，但是我们又不希望所有的数据都显示在`DOM`中，那么我们就在`link`函数中将这个数据属性删除。

```
<div class="list-view">
    <ul ng-repeat="node in list">
        <li draggable drag-option="dragOption"
            attached-data="{{configureTableInfo(node.name)}}">
            <a >
                <span>{{node.name}}</span>
            </a>
        </li>
    </ul>
</div>

<script type="text/javascript">
    $scope.list = [{...}];

    $scope.configureTableInfo = function(name) {
        return {
            name: name,
            data1: data1,
            data2: data2
        };
    };
</script>

```

```
"use strict";

module.exports = function() {
    var angular = require("angular");
    return {
        restrict: 'A',
        link: function($scope, $element, $attrs) {
            $attrs.$observe("attachedData", function() {
                var model = angular.fromJson($attrs.attachedData);
                $element.removeAttr("attached-data");
                delete $attrs.attachedData;
                angular.element($element).data("attachedData", model);
            });
        }
    };
};
```

**注意**：删除`DOM`属性的操作放在了`$attrs.$observe()`里面了，为什么呢？这是因为`configureTableInfo()`是`$scope`中的方法，当用户触发脏检查时，这个函数都会执行，如果不把删除属性的操作放在`$attrs.$observe()`里，实际`render`完成后，你会发现那个属性并没有被删除，这才是本文的精华所在。
