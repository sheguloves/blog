#### 当处理数据非常慢时，如何添加loading
我们都知道，要更新dom，可以利用Angular的数据绑定，通过更改style或者class来动态的更新dom，但是有一种情况是如果用户做了一个操作，更新了$scope中的数据，同时我们给更新的数据添加了事件监听函数，并在此函数中执行一些数据处理的操作，正常情况下，数据操作应该在后端完成，前端只负责展示，但是如果数据量特别大，处理函数要2秒以上才能处理完，而此时页面就好像死掉了一样，这样的用户体验是非常差的。如果我们在处理数据的时候，在页面上显示一个loading的状态，这样会很好的改善用户体验。
<!--more-->
但是如何添加loading并且在loading结束后取消loading呢？第一个想到的方法就是添加一个loading的div，数据开始处理之前，设置当前的状态为loading，同时更新loading div的style或者class,看下面的代码：

{% highlight html %}
<div id="loading" ng-style="{display: loading ? 'block' : 'none'}"></div>
// or
<div id="loading" ng-class="{show: loading}"></div>
{% endhighlight %}
{% highlight javascript %}
// controller
// before process data
$scope.loading = true;
// process data
processData();
$scope.loading = false;

{% endhighlight %}
好了，我们运行一下。
结果呢？我擦，不work。为什么呢？我们知道，javascript是单线程，当我们执行`$scope.loading = true;`后，紧接着`processData();`，此时Angular并不会更新dom，而是会接着执行`$scope.loading = false;`，最后一切都执行结束后，更新dom，也就是说我们先设置loading为true，然后又设置为false，dom上根本没更新。
一般情况下，数据都是通过Ajax请求从后端获取，获取过程中设置loading为true，数据取回来后，在回调函数中将loading重新设置为false，这是起作用的。But why？因为Ajax是异步请求，我们在发送请求之前，将loading设置为true，这时所有的操作就已经结束了，Angular会去更新dom，等到数据取回来后，是重新启动了一系列的操作，这时我们将loading设置为false，然后更新数据，最后更新dom。重点就是**异步操作**。那么可以将我们上面的代码改改：
{% highlight html %}
<div id="loading" ng-style="{display: loading ? 'block' : 'none'}"></div>
// or
<div id="loading" ng-class="{show: loading}"></div>
{% endhighlight %}
{% highlight javascript %}
// controller
// before process data
$scope.loading = true;
timeout(function() {
    // process data
    processData();
    $scope.loading = false;
});

{% endhighlight %}
运行一下，结果呢？OK了。
我只是在做Angular项目的时候遇到了这个问题，如果是用jQuery直接更新dom，不用其他的框架，会不会还有这个bug，我不是太清楚，正常情况下应该是同样会有这个问题的，不过并未做实际的测试。

#### 何时需要手动调用$apply()
AngularJS的文档明确指出，如果当前的执行环境不在AngularJS的context之内，就需要手动的调用`$apply()`方法。

{% highlight javascript %}
angular.module('myApp',[]).controller('MessageController', function($scope) {
  $scope.getMessage = function() {
    setTimeout(function() {
      $scope.message = 'Fetched after 3 seconds';
      console.log('message:'+$scope.message);
    }, 2000);
  }

  $scope.getMessage();

});
{% endhighlight %}

如果页面上有绑定`$scope.message`,那么message将不会更新，更新代码如下：

{% highlight javascript %}
angular.module('myApp',[]).controller('MessageController', function($scope) {
  $scope.getMessage = function() {
    setTimeout(function() {
      $scope.$apply(function() {
        //wrapped this within $apply
        $scope.message = 'Fetched after 3 seconds';
        console.log('message:' + $scope.message);
      });
    }, 2000);
  }

  $scope.getMessage();

});
{% endhighlight %}
message将自动更新。当然我们也可以替换为下面的代码：

{% highlight javascript %}
$scope.getMessage = function() {
  setTimeout(function() {
    $scope.message = 'Fetched after two seconds';
    console.log('message:' + $scope.message);
    $scope.$apply(); //this triggers a $digest
  }, 2000);
};
{% endhighlight %}
上面的代码用的是不带参数的`$scope.$apply()`,请记住你应该一直使用带参数的`$scope.$apply()`因为当你传一个function给`$scope.$apply()`的时候，这个function将会被封装在`try...catch`块之内，如果有任何的错误，都会传给`$exceptionHandler` service.

### User Authentication with the MEAN Stack

下面的Notes是读文章[User Authentication with the MEAN Stack](http://www.sitepoint.com/user-authenication-mean-stack/)的时候学习到的一些知识,虽然是MEAN项目，同样适用于其它技术的full-stack开发，例如，后端如果是Java，只需要把后端的实现用Java替代就可。

#### Protect a Route for Logged in Users Only

{% highlight javascript %}
(function () {

  angular.module('meanApp', ['ngRoute']);

  function config ($routeProvider, $locationProvider) {
    $routeProvider
      .when('/', {
        templateUrl: 'home/home.view.html',
        controller: 'homeCtrl',
        controllerAs: 'vm'
      })
      .when('/register', {
        templateUrl: '/auth/register/register.view.html',
        controller: 'registerCtrl',
        controllerAs: 'vm'
      })
      .when('/login', {
        templateUrl: '/auth/login/login.view.html',
        controller: 'loginCtrl',
        controllerAs: 'vm'
      })
      .when('/profile', {
        templateUrl: '/profile/profile.view.html',
        controller: 'profileCtrl',
        controllerAs: 'vm'
      })
      .otherwise({redirectTo: '/'});

    // use the HTML5 History API
    $locationProvider.html5Mode(true);
  }

  function run($rootScope, $location, authentication) {
    $rootScope.$on('$routeChangeStart', function(event, nextRoute, currentRoute) {
      if ($location.path() === '/profile' && !authentication.isLoggedIn()) {
        $location.path('/');
      }
    });
  }

  angular
    .module('meanApp')
    .config(['$routeProvider', '$locationProvider', config])
    .run(['$rootScope', '$location', 'authentication', run]);

})();
{% endhighlight %}

#### common service call

{% highlight javascript %}
function meanData ($http, authentication) {

  var getProfile = function () {
    return $http.get('/api/profile', {
      headers: {
        Authorization: 'Bearer '+ authentication.getToken()
      }
    });
  };

  return {
    getProfile : getProfile
  };
}
{% endhighlight %}


#### factory vs service
* factory return a object, you can use it in controller initialization.
* service behavior like a "New" function, you cannot get its properties and methods in controller initialization

#### component
* **remember**: if define a property in controller of a component like "aaName", pass value to this component should use property name "aa-name"

  > Normalization
  > Angular normalizes an element's tag and attribute name to determine which elements match which directives. We typically refer to directives by their case-sensitive camelCase normalized name (e.g. ngModel). However, since HTML is case-insensitive, we refer to directives in the DOM by lower-case forms, typically using dash-delimited attributes on DOM elements (e.g. ng-model).
  > The normalization process is as follows:
  > * Strip x- and data- from the front of the element/attributes.
  > * Convert the :, -, or _-delimited name to camelCase.

#### $http call
* Please check whether the parameters need to serilize if the call cannot work, and also check if need to add the headers for $http

#### $timeout and $interval
* please use $timeout and $interval in angularjs, because the two service will update properties of $scope but default timeout and interval do not.

#### customize fonts in css
I'm using the following code in my css:
{% highlight css %}
@font-face {
    font-family: "rsfont";
    src: url("rsfont.ttf");
}

body {
    font-family: 'rsfont';
}
{% endhighlight %}
I have a `rsfont.ttf` file in the same folder as my css file, but it just doesn't work. Why?

.ttf font works in Safari, Android, iOS. To make the font work in all browsers you need to make more font formats using a fontface generator. You can use the one on fontsquirrel

Your final @fontface declarations should be something like this to work in all browsers supporting the @fontface
{% highlight css %}
@font-face {
  font-family: 'rsfont';
  src: url('rsfont.eot'); /* IE9 Compat Modes */
  src: url('rsfont.eot?#iefix') format('embedded-opentype'), /* IE6-IE8 */
       url('rsfont.woff') format('woff'), /* Modern Browsers */
       url('rsfont.ttf')  format('truetype'), /* Safari, Android, iOS */
       url('rsfont.svg#svgFontName') format('svg'); /* Legacy iOS */
}
{% endhighlight %}