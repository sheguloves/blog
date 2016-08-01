---
layout:     post
title:      "Angular Set Focus When Action Taken"
subtitle:   "How to set focus on input field"
date:       2016-08-01
author:     "Felix Xi"
header-img: "img/code-3.jpg"
tags:
    - Angular
---

Reference: [http://stackoverflow.com/questions/14833326/how-to-set-focus-on-input-field](http://stackoverflow.com/questions/14833326/how-to-set-focus-on-input-field)

##### When a Modal is opened, set focus on a predefined `<input>` inside this Modal.

Define a directive and have it `$watch` a property/trigger so it knows when to focus the element:

```
Name: <input type="text" focus-me="shouldBeOpen">
```

```
app.directive('focusMe', function($timeout, $parse) {
  return {
    //scope: true,   // optionally create a child scope
    link: function(scope, element, attrs) {
      var model = $parse(attrs.focusMe);
      scope.$watch(model, function(value) {
        console.log('value=',value);
        if(value === true) {
          $timeout(function() {
            element[0].focus();
          });
        }
      });
      // to address @blesh's comment, set attribute value to 'false'
      // on blur event:
      element.bind('blur', function() {
         console.log('blur');
         scope.$apply(model.assign(scope, false));
      });
    }
  };
});
```

The `$timeout` seems to be needed to give the modal time to render.

##### Everytime `<input>` becomes visible (e.g. by clicking some button), set focus on it.

Create a directive essentially like the one above. Watch some scope property, and when it becomes true (set it in your ng-click handler), execute `element[0].focus()`. Depending on your use case, you may or may not need a `$timeout` for this one:

```
<button class="btn" ng-click="showForm=true; focusInput=true">show form and
 focus input</button>
<div ng-show="showForm">
  <input type="text" ng-model="myInput" focus-me="focusInput"> {{ myInput }}
  <button class="btn" ng-click="showForm=false">hide form</button>
</div>
```

```
app.directive('focusMe', function($timeout) {
  return {
    link: function(scope, element, attrs) {
      scope.$watch(attrs.focusMe, function(value) {
        if(value === true) {
          console.log('value=',value);
          //$timeout(function() {
            element[0].focus();
            scope[attrs.focusMe] = false;
          //});
        }
      });
    }
  };
});
```

**Update 7/2013**: I've seen a few people use my original isolate scope directives and then have problems with embedded input fields (i.e., an input field in the modal). A directive with no new scope (or possibly a new child scope) should alleviate some of the pain. So above I updated the answer to not use isolate scopes. Below is the original answer:

Original answer for 1., using an isolate scope:

```
Name: <input type="text" focus-me="{{shouldBeOpen}}">
```

```
app.directive('focusMe', function($timeout) {
  return {
    scope: { trigger: '@focusMe' },
    link: function(scope, element) {
      scope.$watch('trigger', function(value) {
        if(value === "true") {
          $timeout(function() {
            element[0].focus();
          });
        }
      });
    }
  };
});
```


Original answer for 2., using an isolate scope:

```
<button class="btn" ng-click="showForm=true; focusInput=true">show form and
 focus input</button>
<div ng-show="showForm">
  <input type="text" focus-me="focusInput">
  <button class="btn" ng-click="showForm=false">hide form</button>
</div>
```

```
app.directive('focusMe', function($timeout) {
  return {
    scope: { trigger: '=focusMe' },
    link: function(scope, element) {
      scope.$watch('trigger', function(value) {
        if(value === true) {
          //console.log('trigger',value);
          //$timeout(function() {
            element[0].focus();
            scope.trigger = false;
          //});
        }
      });
    }
  };
});
```

Since we need to reset the trigger/focusInput property in the directive, `'='` is used for two-way databinding. In the first directive, `'@'` was sufficient. Also note that when using `'@'` we compare the trigger value to `"true"` since `'@'` always results in a string.