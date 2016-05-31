---
layout:     post
title:      "Angular-ui-router Abstract State Active"
subtitle:   "Make angular-ui-router abstract state active"
date:       2016-04-08
author:     "Felix Xi"
header-img: "img/banner-7.jpg"
catalog: true
tags:
    - Angular-ui-router
    - Angular
    - Directive
---

### Background

`Agnualr` 原生提供了一个 `angular-router`，不过我想更多人用 `angular-ui-router` 吧。本文谈的就是关于 `angular-ui-router` 的一个问题及相应的解决办法，我想原生的 `angular-router` 如果有同样的问题，可以参考写出对应的解决办法。

`angular-ui-router` abstract state 的使用方式：

```
/***** javascript *****/
angular
    .module('app', [])
    .config(["$stateProvider", "$urlRouterProvider", function($stateProvider, $urlRouterProvider) {
        $urlRouterProvider.otherwise("/aa");
        $stateProvider
            .state('aa', {
                url: "/aa",
                templateUrl: "aa.html"
            })
            .state('bb', {
                url: "/bb",
                abstract: true,
                templateUrl: "bb.html"
            })
            .state('bb.cc', {
                url: "",
                templateUrl: "cc.html"
            })
            .state('bb.dd', {
                url: "/dd",
                templateUrl: "dd.html"
            });
    }]);

/****** html ********/
// aa.html

<div>This is aa view</div>

// bb.html
<div ui-view></div>

// cc.html
<div>This is cc view</div>

// dd.html
<div>This is dd view</div>

```

我们看到，`/bb` 是一个 abstract 的 state ，`bb.html` 中没有自己的 `DOM node`, 如果我们要在页面上加 `link` 怎么办

```
/***** nav bar ******/
<ul>
    <li ui-sref-active="active" ui-sref="aa">aa</li>
    <li ui-sref-active="active" ui-sref="bb.cc">bb</li>
</ul>
```

上面代码中我们看到，`ui-sref="bb.cc"`，因为 `bb` 是一个 abstract state，所以不能 link，必须 link 到 `bb.cc`，但是

```
.state('bb.cc', {
    url: "",
    templateUrl: "cc.html"
})
```

所以，实际上的 url 为 `path/bb` 而不是 `path/bb/cc`，而 `<li ui-sref-active="active" ui-sref="bb.cc">bb</li>` 对应的 state是 `bb.cc` ， 而不是 `bb` ，因此 `ui-sref-active="active"` 只有在 url 为 `path/bb` 时会使 link 处于 active 状态，如果 url 为 `path/bb/dd` ，那么这个 link 就会是 inactive 状态。

那么如何解决呢，接着往下看。

### Key point

在搜索的时候发现一个比较天才的解决方法：

```
    app.directive('uiSrefActiveIf', ['$state', function($state) {
        return {
            restrict: "A",
            controller: ['$scope', '$element', '$attrs', function ($scope, $element, $attrs) {
                var state = $attrs.uiSrefActiveIf;
                function update() {
                    if ( $state.includes(state) || $state.is(state) ) {
                        $element.addClass("active");
                    } else {
                        $element.removeClass("active");
                    }
                }
                $scope.$on('$stateChangeSuccess', update);
                update();
            }]
        };
    }])

    // Then you use matching HTML ui-sref-active-if="parent" will add the class "active"
```

[参考链接](https://github.com/angular-ui/ui-router/issues/1431)。

话说这才是本文的精华所在，虽然很短。这种解决方法给我们一个很好的启发，脑洞大开的我们可以很好的借鉴一下。

上面的方法只能使用 active class ，根据上面给出的解决办法，我做了一下加工

```
app.directive('uiSrefActiveIf', ['$state', function($state) {
    return {
        restrict: "A",
        link: function(scope, element, attrs) {
            var attr = JSON.parse(attrs.uiSrefActiveIf);
            var state = attr && attr.state;
            var clazz = attr && attr.class;

            function update() {
                if ( $state.includes(state) || $state.is(state) ) {
                    element.addClass(clazz);
                } else {
                    element.removeClass(clazz);
                }
            }

            scope.$on('$stateChangeSuccess', update);
            update();
        }
    };
}])

/****** 自己的directive ********/

function controller($scope) {
    $scope.activeObject = function(parent) {
        return {
            class: 'selected',
            state: parent
        };
    };
}

/****** html *****/

<ul>
    <li ui-sref-active-if="{{activeObject(parent)}}" ui-sref="aa">aa</li>
    <li ui-sref-active-if="{{activeObject(parent)}}" ui-sref="bb.cc">bb</li>
</ul>

```

为什么要用 `link`， 如果是 `controller`，那么 `$attrs` 中的值都是没有经过 `angular` 处理的，例如如果我们的代码是： `ui-sref-active-if="{{activeObject('bb')}}"`, 那么在 `controller` 的 `$attrs` 中我们拿到的值就是 `"{{activeObject('bb')}}"`, 而在 `link` 中，所有的值已经被 `angular` 处理过，我们拿到的就是 `"{class: 'selected', state: 'bb'}"`。这就是我为什么要加工。

脑洞大开呀，还没有时间看 `angular` 的源代码，得赶紧抽时间系统的看一下，必定会有很多意想不到的收获。