---
layout:     post
title:      "Angular Dynamic Template"
subtitle:   "Show dynamic view according configuration"
date:       2016-04-12
author:     "Felix Xi"
header-img: "img/banner-9.jpg"
tags:
    - Angular
    - Directive
---

### Background

`Agnualr` 将 `html` 拆分为 `template` 跟 `controller` 的一个重要的目的就是将数据逻辑与显示分开，而且不建议用户直接操作 `DOM`。如果要控制部分 `DOM Node` 显示不显示，一般我们会用 `ng-show` 或 `ng-hide` (`css`的语法糖) 来控制。这样做的好处就是方便简单，当然缺点就是会添加一些没有用的 `DOM Node` 在页面上。有时候我们确实需要将不显示的 `DOM Node` 从 `DOM Tree` 中移除，比如，用户可以根据自己的喜好配置一个 `config` 文件，页面将跟据他的配置文件来动态的添加相应的组件，而将不显示的组件去除。那么如何实现呢？

我们知道，`Directive` 的 `link` 方法可以提供给开发人员操作 `DOM` 的功能。看下面的代码：

```
"use strict";

var app = require('angular')
    .module('app',[])
    .directive('customizedComponent', ['$compile', function ($compile) {
        var components = {
            a: '<div>a</div>',
            b: '<div>b</div>',
            c: '<div>c</div>'
        };

        function getTemplate(components){
            var template = '';
            components.forEach(function(item) {
                template = template + components[item];
            });
            return template;
        }

        var linker = function(scope, element) {
            element.html(getTemplate(scope.components));
            $compile(element.contents())(scope);
        };

        var controller = function($scope, customerService) {
            var components = $scope.components;

            components.forEach(function(item) {
                switch(item) {
                    case "a":
                        $scope.a = customerService.getA();
                        break;
                    case "b":
                        $scope.b = customerService.getB();
                        break;
                    case "c":
                        $scope.a = customerService.getC();
                        break;
                    default:
                        break;
                }
            });
        };

        return {
            restrict: "E",
            link: linker,
            controller: ['$scope', 'customerService', controller],
            scope: {
                components:'<'
            }
        };
    });


/************ html ******************/

<customized-component components='{{["a", "b"]}}'></customized-component>

```

这样就可以实现动态加载相应的组件的需求，同时也可以减少不必要的数据请求。我们还可以进一步将配置文件放在 server 端，根据不同的用户配置不同的 components，在页面加载前 load 用户配置。例如：

```
angular
    .module('app', ['ui.router'])
    .config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {
        $urlRouterProvider.otherwise("/page1");
        $stateProvider
            .state('page1', {
                url: "/page1",
                template: "<customized-component components='components'></customized-component>",
                resolve: {
                    components: function($http) {
                        return $http({
                            method: 'GET',
                            url: '/components.json'
                        }).then(function success(response) {
                            return response.data;
                        });
                    }
                },
                controller: ['$scope', 'components', function($scope, components) {
                    $scope.components = components;
                }]
            })
            .state('page2', {
                url: "/page2",
                template: "<div> page2 </div>"
            });
        }]);
```

本文参考：[AngularJS Dynamic Templates – Yes We Can!](http://onehungrymind.com/angularjs-dynamic-templates/)
