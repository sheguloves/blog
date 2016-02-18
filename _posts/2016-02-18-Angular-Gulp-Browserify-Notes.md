---
layout: post
title: Angular with gulp and browserify
description: "Build Angularjs app with gulp and browserify"
modified: 2016-02-18
tags: [Angular, Angularjs, gulp, browserify]
---

## Background
一直在关注`Angular`, `gulp`, `browserify`,正好公司项目更新,就用这些新的东西来做更新我们的项目.项目地址：[https://github.com/sheguloves/monitor](https://github.com/sheguloves/monitor)
下面是在构建项目的过程中遇到的一些问题,记录下来,以免忘掉.

<!--more-->

## ng-route | angular-route
the folder structure like:

{% highlight yaml %}
example
    public
    src
        app
            controller
                login.js
                dashboard.js
                index.js
            partials
                login.html
                dashboard.html
            service
                index.js
                service.js
            main.js
        assets
            fonts
                Arial.ttf
            images
                1.png
                2.png
            styles
                styles.css
        data
            model.js
        env
            prod.js
            dev.js
    index.html
{% endhighlight %}

ng-route config in `main.js`:

{% highlight javascript %}
app.config(function($routeProvider) {
    $routeProvider.when('/login', {
            templateUrl: 'app/partials/login.html',
            controller: 'LoginCtrl',
        })
        .when('/dashboard', {
            templateUrl: 'app/partials/dashboard.html',
            controller: 'DashboardCtrl',
        })
        .otherwise({
            redirectTo: '/login',
        });
});
{% endhighlight %}

You can see, the `templateUrl` should be `'app/partials/login.html'`, cannot use `./partials/login.html`

## Angular with browserify

### Minify the code generate by browserify

First you need to install `gulp-uglify` and `gulp-streamify`

{% highlight javascript %}
npm install gulp-uglify gulp-streamify --save-dev
{% endhighlight %}

Now before piping the concatenated bundle into public/js/bundle.js, you can minify/uglify the response by writing

{% highlight javascript %}
.pipe(streamify(uglify()))
{% endhighlight %}

So, the task becomes :

{% highlight javascript %}
return browserify('app.js')
        .transform('reactify', {stripTypes: true, es6: true})
        .bundle()
        .pipe(source('bundle.js'))
        .pipe(streamify(uglify())) //note this
        .pipe(gulp.dest('public/js/bundle.min.js'));
{% endhighlight %}

Now your bundle.js is minified thanks to uglify and streamify. Note that there are multiple ways to do it and this is one of the ways.

### Minify angular code

**Example**

*Don't* rely on inference:

{% highlight javascript %}
app.controller('LoginCtrl', function ($scope, $timeout, myFooService) {
});
{% endhighlight %}

If you use above code, you need to use [ng-annotate](https://github.com/olov/ng-annotate), see the following code if you use gulp to build your code.

{% highlight javascript %}
var gulp = require('gulp'),
    streamify = require('gulp-streamify'),
    source = require('vinyl-source-stream'),
    browserify = require('browserify'),
    uglify = require('gulp-uglify'),
    ngAnnotate = require('gulp-ng-annotate');

gulp.task('annotation', ['clean'], function() {
    return gulp.src(['src/app/**/*.js'])
          .pipe(ngAnnotate())
          .pipe(gulp.dest('src/app/ngAnnotate'));
});

// minify code
gulp.task('browserify-min', ['clean', 'annotation'], function() {
    var bundleStream = browserify('src/app/ngAnnotate/main.js', {
            insertGlobals: true,
            debug: false
        }).bundle()
        .pipe(source('main.js'))
        .pipe(rename({ suffix: '.min' }))
        .pipe(streamify(uglify()))
        .pipe(gulp.dest('public/app'))
        .pipe(notify({ message: 'browserify main.js and minification complete' }));
});
{% endhighlight %}

**But** if you use browserify build your code, you may write code like this:

index.js
{% highlight javascript %}
var app = require('angular').module('monitor');

//Don't use this, ng-annotate doesn't work for this
//app.controller('LoginCtrl', require('./login'));
//app.controller('DashboardCtrl', require('./dashboard'));

//use this
app.controller('LoginCtrl', ['$scope', '$http', '$location', require('./login')]);
app.controller('DashboardCtrl', ['$scope', require('./dashboard')]);

{% endhighlight %}
login.js
{% highlight javascript %}
module.exports = function($scope, $http, $location) {
    $scope.siteName = "test";
    $scope.logoSrc = "";
    $scope.validate = true;
    ...
    ...
};
{% endhighlight %}

Or you can merge two files code into one

{% highlight javascript %}
app.controller('LoginCtrl', function ($scope, $timeout, myFooService) {
});
{% endhighlight %}

then use ng-annotate and ng-annotate will generate the following code:

{% highlight javascript %}
app.controller('LoginCtrl', ['$scope', '$timeout', 'myFooService', function ($scope, $timeout, myFooService) {
});
{% endhighlight %}

**Other way**

If you just want write code like:

index.js
{% highlight javascript %}
var app = require('angular').module('monitor');
app.controller('LoginCtrl', require('./login'));
app.controller('DashboardCtrl', require('./dashboard'));

{% endhighlight %}
login.js
{% highlight javascript %}
module.exports = function($scope, $http, $location) {
    $scope.siteName = "test";
    $scope.logoSrc = "";
    $scope.validate = true;
    ...
    ...
};
{% endhighlight %}

You can modify the build script like:

{% highlight javascript %}
// minify code
gulp.task('browserify-min', ['clean'], function() {
    var bundleStream = browserify('src/app/main.js', {
            insertGlobals: true,
            debug: false
        }).bundle()
        .pipe(source('main.js'))
        .pipe(rename({ suffix: '.min' }))
        .pipe(streamify(uglify({ mangle: false }))) // Important, if there is no "mangle: false", build cannot run
        .pipe(gulp.dest('public/app'))
        .pipe(notify({ message: 'browserify main.js and minification complete' }));
});
{% endhighlight %}
You can see we add option: `{mangle: false}` for `uglify`, then you don't need ng-annotate any more.

**Otherwise**

Use either inline annotation:

{% highlight javascript %}
[ '$scope', '$timeout', 'myFooService', function ($scope, $rootScope, myFooService) {
}]
{% endhighlight %}

Or the `$inject` property:

{% highlight javascript %}
function MyFactory($scope, $timeout, myFooService) {
}
MyFactory.$inject = [ '$scope', '$timeout', 'myFooService' ];
{% endhighlight %}

You just use the following code to build your code

{% highlight javascript %}
// minify code
gulp.task('browserify-min', ['clean'], function() {
    var bundleStream = browserify('src/app/main.js', {
            insertGlobals: true,
            debug: false
        }).bundle()
        .pipe(source('main.js'))
        .pipe(rename({ suffix: '.min' }))
        .pipe(streamify(uglify()))
        .pipe(gulp.dest('public/app'))
        .pipe(notify({ message: 'browserify main.js and minification complete' }));
});
{% endhighlight %}

Pick up the favorite to build your code

### Gulp
When I use `gulp` to build my project, I use the following script

{% highlight javascript %}
var gulp = require('gulp'),
    concat = require('gulp-concat'),
    del = require('del');

gulp.task('clean', function() {
    del(['public/**']);
});

gulp.task('concat', function() {
    gulp.src('app/**/*.js').pipe(concat('main.js')).pipe(gulp.dest('dist'));
});

gulp.task('build', ['clean', 'concat']);
{% endhighlight %}

But sometimes, it does work. Why? I found `del` has a function `sync`, So I use the following code:

{% highlight javascript %}
gulp.task('clean', function() {
    del.sync(['public/**']);
});
{% endhighlight %}
But I still get error sometime. After a little research I noticed gulp runs all its tasks in parallel. So if you need to run tasks in orders, you just need doing this:

{% highlight javascript %}
gulp.task('clean', function() {
    return del.sync(['public/**']);
});

gulp.task('concat', function() {
    return gulp.src('app/**/*.js').pipe(concat('main.js')).pipe(gulp.dest('dist'));
});

gulp.task('build', ['clean', 'concat']);
{% endhighlight %}

All tasks will execute one by one.