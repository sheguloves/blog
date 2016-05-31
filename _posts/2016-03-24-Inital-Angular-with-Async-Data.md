---
layout:     post
title:      "Inital Angular App with Async Data"
subtitle:   "Inital angular router with async data"
date:       2016-03-24
author:     "Felix Xi"
header-img: "img/banner-5.jpg"
tags:
    - Angular
---

### Background

在开发过程中，我们经常会遇到需要在界面出来之前需要提前做一些配置，而配置信息有些则是从后端拿来，那么在 `Angular` 中如何实现呢？请看下文。

### Content

正如我们所了解的，配置 Angular Router 需要在 Angular APP 初始化的时候去完成

```
var angular = require('angular');
require('angular-ui-router');
var app = angular.module('monitor', ['ui.router']);
app.config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {
    $urlRouterProvider.otherwise("/login");
    $stateProvider
        .state('login', {
            url: "/login",
            templateUrl: 'app/component/login/login.html',
            controller: 'LoginCtrl'
        })
        .state('dashboard', {
            url: "/dashboard",
            templateUrl: 'app/component/dashboard/dashboard.html',
            controller: 'DashboardCtrl'
        });
}]);
```
OK, 一切都还不错，挺简单的。需求来了： `我们想根据用户的状态来决定他的 default url 应该是什么(也就是$urlRouterProvider.otherwise("")接收的值是动态的)`，比如，如果用户是一个超级用户，我们应该将他的 default url 设置为 `/dashboard` 而不是 `/login` 。OK，可以呀，在 login.html 里面去判断呀？不是一样可以解决问题吗

```
<!-- login.html -->
<div id="loginBody" ng-controller="LoginCtrl">
    <form id="loginForm">
        <fieldset id="inputs">
            <input type="text" class="usernameInput" ng-model="user.username"
                   placeholder="Username" autofocus required/>
            <input type="password" class="passwordInput" ng-model="user.password"
                   placeholder="Password" required/>
        </fieldset>
        <fieldset>
            <input id="remember_me" type="checkbox" ng-model="user.rememberMe" class="rememberMe" />
            <label htmlFor="remember_me" class="rememberMe">Remember my account</label>
        </fieldset>
        <input id="submit" type="submit" value="" ng-click="login(user)"/>
    </form>
</div>
<script type="text/javascript">
    app.controller('LoginCtrl', ['$http', '$state', function($scope, $state) {
        $http({
            method: 'GET',
            url: '/userstate'
        }).then(function successCallback(response) {
            $state.go('dashboard');
        }, function errorCallback(response) {

        });
    }]);
</script>
```
这样不是也阔以的吗？哈哈哈。

你以为这样就结束了？too young too simple！用户为什么每次进来都要先跳到 login ？即使他是超级用户。这你能忍？我不能忍。

那我们把获取 user state 的代码移到初始化的地方

```
var angular = require('angular');
require('angular-ui-router');
var app = angular.module('monitor', ['ui.router']);
app.config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {

    // 我擦，不能用$http, 我们可以用 Ajax call，在这里我就不写这部分代码了，用 setTimeout 模拟异步调用
    setTimeout(function() {
        $urlRouterProvider.otherwise("/login");
        $stateProvider
            .state('login', {
                url: "/login",
                templateUrl: 'app/component/login/login.html',
                controller: 'LoginCtrl'
            })
            .state('dashboard', {
                url: "/dashboard",
                templateUrl: 'app/component/dashboard/dashboard.html',
                controller: 'DashboardCtrl'
            });
        }, 1);
}]);

```

好了，我们运行一下。界面上什么都木有，不 work。接着改改

```
var angular = require('angular');
require('angular-ui-router');
setTimeout(function() {
    var app = angular.module('monitor', ['ui.router']);
    app.config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {
        $urlRouterProvider.otherwise("/login");
        $stateProvider
            .state('login', {
                url: "/login",
                templateUrl: 'app/component/login/login.html',
                controller: 'LoginCtrl'
            })
            .state('dashboard', {
                url: "/dashboard",
                templateUrl: 'app/component/dashboard/dashboard.html',
                controller: 'DashboardCtrl'
            });
    }]);
}, 1);
```
再运行一下，我擦，还是不 work。

来，我们好好研究一下为啥倪。

我们都知道，使用 Angular bootstrap 我们的 APP 的时候可以直接在 DOM 上添加 `ng-app` 标签来实现，还有一种方式就是手动的 bootstrap。

```
<!-- Automatic Initialization -->
<!doctype html>
<html ng-app="app">
    <body>
    I can add: {{ 1+2 }}.
    <script src="angular.js"></script>
    </body>
</html>

```

```
<!-- Manual Initialization -->
<!doctype html>
<html>
<body>
    <div ng-controller="MyController">
        Hello {{greetMe}}!
    </div>
    <script src="http://code.angularjs.org/snapshot/angular.js"></script>

    <script>
        angular.module('myApp', [])
            .controller('MyController', ['$scope', function ($scope) {
                $scope.greetMe = 'World';
        }]);

        angular.element(document).ready(function() {
            angular.bootstrap(document, ['myApp']);
        });
    </script>
</body>
</html>

```
在我们上面的例子中，我们使用的是 Automatic Initialization 这种方式，所以在一开始就会执行 Angular 的初始化，而我们的 config 函数作为异步调用的 callback，实际上已经发生在其他代码之后了，比如，注册 service，directive 等等。所以我们要把注册 service，directive等的代码放在 Angular 的主 APP 初始化之后执行。那么我们更改代码：


```
var angular = require('angular');
require('angular-ui-router');

// 实际使用中，请将setTimeout替换成 server call
setTimeout(function() {
    // 此时，user state 已经拿到了
    var app = angular.module('monitor', ['ui.router']);
    app.config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {

        if (userstate) {
            $urlRouterProvider.otherwise("/dashboard");
        } else {
            $urlRouterProvider.otherwise("/login");
        }
        $stateProvider
            .state('login', {
                url: "/login",
                templateUrl: 'app/component/login/login.html',
                controller: 'LoginCtrl'
            })
            .state('dashboard', {
                url: "/dashboard",
                templateUrl: 'app/component/dashboard/dashboard.html',
                controller: 'DashboardCtrl'
            });
    }]);

    // 注意！注意！注意！我们使用的是Manual Initialization
    angular.bootstrap(document, ['monitor'], {strictDi: true});

}, 1000);
```

运行一下，一切OK了。这样看起来是不是就顺眼多了，不过这样的实现方式是不是最优雅的方式我不能知道，也许 Angular 有更优雅的方式，暂时没有发现，还没有系统的读过 Angular 的文档跟代码，话说我对读代码不在行，以后如果发现有更优雅的方式，会再贴出来。

