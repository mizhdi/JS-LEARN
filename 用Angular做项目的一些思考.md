# 简介

> 时间能力有限，没有读过源码，只是简单的摘录项目中用到的方法，踩过的坑，还需很长时间的深入学习。

AngularJS是一款了不起的前端框架，随着项目的开发进展，越发的感觉自己了解的不够深入，我们用到的只是框架中很少的一部分功能，还有很多的功能模块和设计思想等着我们去研究。


# 内容目录

* 构建
* 结构
* 路由
* 数据
* 指令
* Providers
* 缓存
* 优化
* Tips

## 构建

个人用 [generator-angular](https://github.com/yeoman/generator-angular), 因为做过的项目属于内容比较多的Single Page Application，简单的渲染页面推荐使用React。

用yeoman(Grunt)构建ng项目，有一套比较成熟的体系，比如网站配置文件的生成，可以使用 `grunt-ng-constant`, 打不同环境配置的包。

```javascript
ngconstant: {
  // Options for all targets
  options: {
    space: '  ',
    wrap: '"use strict";\n\n {%= __ngModule %}',
    name: 'config',
  },
  // Environment targets
  development: {
    options: {
      dest: '<%= yeoman.app %>/scripts/config.js'
    },
    constants: {
      ENV: {
        name: 'development',
        url: 'http://127.0.0.1',
        version: '0.1.0'
      }
    }
  },
  production: {
    options: {
      dest: '<%= yeoman.dist %>/scripts/config.js'
    },
    constants: {
      ENV: {
        name: 'production',
        url: 'http://baidu.com',
        version: '0.1.0'
      }
    }
  }
},
```

## 结构

由于一个大型的AngularJS应用有较多组成部分，所以最好通过分层的目录结构来组织。 有两个主流的组织方式：

－ 按照类型优先，业务功能其次的组织方式

```
 ├── app
 │   ├── app.js
 │   ├── controllers
 │   │   ├── page1
 │   │   │   ├── FirstCtrl.js
 │   │   │   └── SecondCtrl.js
 │   │   └── page2
 │   │       └── ThirdCtrl.js
 │   ├── directives
 │   │   ├── page1
 │   │   │   └── directive1.js
 │   │   └── page2
 │   │       ├── directive2.js
 │   │       └── directive3.js
 │   ├── filters
 │   │   ├── page1
 │   │   └── page2
 │   └── services
 │       ├── CommonService.js
 │       ├── cache
 │       │   ├── Cache1.js
 │       │   └── Cache2.js
 │       └── models
 │           ├── Model1.js
 │           └── Model2.js
 ├── lib
 └── test
```

－ 按照业务功能优先，类型其次的组织方式

```
├── app
│   ├── app.js
│   ├── common
│   │   ├── controllers
│   │   ├── directives
│   │   ├── filters
│   │   └── services
│   ├── page1
│   │   ├── controllers
│   │   │   ├── FirstCtrl.js
│   │   │   └── SecondCtrl.js
│   │   ├── directives
│   │   │   └── directive1.js
│   │   ├── filters
│   │   │   ├── filter1.js
│   │   │   └── filter2.js
│   │   └── services
│   │       ├── service1.js
│   │       └── service2.js
│   └── page2
│       ├── controllers
│       │   └── ThirdCtrl.js
│       ├── directives
│       │   ├── directive2.js
│       │   └── directive3.js
│       ├── filters
│       │   └── filter3.js
│       └── services
│           └── service3.js
├── lib
└── test
```

## 路由

用[ui-router](https://github.com/angular-ui/ui-router), 参考文档 [https://github.com/angular-ui/ui-router/wiki](https://github.com/angular-ui/ui-router/wiki)

ui-router不只是路由那么简单，简单介绍下应用

### 模版

可以通过ui-view使一个复杂页面分模块开发，目录结构清晰，可以使controller不冗余

```html
<div ui-view="info"></div>
<div ui-view="require"></div>
<div ui-view="stat"></div>
<div ui-view="order"></div>
<div ui-view="record"></div>
```

```javascript
.state('home.customer.profile', {
  views: {
    '': {
      controller: 'CustomerProfileCtrl',
      templateUrl: 'views/customer-profile.html'
    },
    'info@home.customer.profile': {
      controller: 'CustomerProfileInfoCtrl',
      templateUrl: 'views/customer-profile-info.html'
    },
    'require@home.customer.profile': {
      controller: 'CustomerProfileRequireCtrl',
      templateUrl: 'views/customer-profile-require.html'
    },
    'stat@home.customer.profile': {
      controller: 'CustomerProfileStatCtrl',
      templateUrl: 'views/customer-profile-stat.html'
    },
    'order@home.customer.profile': {
      controller: 'CustomerProfileOrderCtrl',
      templateUrl: 'views/customer-profile-order.html'
    },
    'record@home.customer.profile': {
      controller: 'CustomerProfileRecordCtrl',
      templateUrl: 'views/customer-profile-record.html'
    }
  }
})
```

### 权限控制

简单的权限控制 `resolve: { authenticated: authenticated }`

```javascript
var authenticated = ['$q', '$rootScope', 'AuthService', function($q, $rootScope, AuthService) {
  var deferred = $q.defer();

  if (AuthService.isAuthenticated()) {
    deferred.resolve();
  } else {
    deferred.reject('Not logged in');
  }
  return deferred.promise;
}];
```

### State Change Events && View Load Events

`$stateChangeStart`, `$stateNotFound`, `$stateChangeSuccess`, `$stateChangeError`, `$viewContentLoading`, `$viewContentLoaded`, 这些都可以应用到具体的业务逻辑中去。

```javascript
$rootScope.$on('$stateChangeError', function(event, toState, toParams, fromState, fromParams, error) {
  $state.go('login');
  $rootScope.$broadcast(AUTH_EVENTS.notAuthenticated);
});
```

### 数据加载

resolve挺强大的，可以加载完数据之后渲染页面，注入到controller中后就可以使用该数据

```javascript
resolve: {
  customerId: function($stateParams) {
    return $stateParams.id;
  },
  initialData: function(initialService) {
    return initialService();
  }
}
```

## 数据

### Constant

一些常量，可以直接在controller中使用, 依赖注入ENV

```javascript
.constant('ENV',{
  name: 'development',
  url: 'http://127.0.0.1',
  version:'0.1.0'
})
```

### 全局的controller

```html
<body ng-app="app" ng-controller="ApplicationCtrl">
```
继承 `$scope`，作用有点类似于 `$rootScope`

### ui-router resolve

上文有介绍，不过多阐述

### service

就是调用接口

#### $http, $q



#### $resource

一般使用的就是$http, 但是ng提供了更为强大的$resource

## 指令

非常重要的部分，可以处理很多业务逻辑，包括对jquery插件的使用。

教程：

[AngularJS 指令实践指南（一）](http://blog.jobbole.com/62249/)

[AngularJS 指令实践指南（二）](http://blog.jobbole.com/62999/)


## Providers

[providers doc](https://docs.angularjs.org/guide/providers)

简单的区分：[Service vs provider vs factory](http://stackoverflow.com/questions/15666048/service-vs-provider-vs-factory)

**一个栗子：**，基本可以理解这三个概念

```javascript
var myApp = angular.module('myApp', []);

//Service style, probably the simplest one
myApp.service('helloWorldFromService', function() {
    this.sayHello = function() {
        return "Hello, World!"
    };
});

//Factory style, more involved but more sophisticated
myApp.factory('helloWorldFromFactory', function() {
    return {
        sayHello: function() {
            return "Hello, World!"
        }
    };
});

//Provider style, full blown, configurable version
myApp.provider('helloWorld', function() {
    // In the provider function, you cannot inject any
    // service or factory. This can only be done at the
    // "$get" method.

    this.name = 'Default';

    this.$get = function() {
        var name = this.name;
        return {
            sayHello: function() {
                return "Hello, " + name + "!"
            }
        }
    };

    this.setName = function(name) {
        this.name = name;
    };
});

//Hey, we can configure a provider!
myApp.config(function(helloWorldProvider){
    helloWorldProvider.setName('World');
});


function MyCtrl($scope, helloWorld, helloWorldFromFactory, helloWorldFromService) {

    $scope.hellos = [
        helloWorld.sayHello(),
        helloWorldFromFactory.sayHello(),
        helloWorldFromService.sayHello()];
}
```

## 缓存

不会频繁的发生变化的数据，客户端不必需要重复进行同样的请求，利用缓存可以减少节省时间和带宽。

```javascript
$http.get(url, { cache: true}).success(...);
```

```javascript
var cache = $cacheFactory('myCache');

var data = cache.get(someKey);

if (!data) {
   $http.get(url).success(function(result) {
      data = result;
      cache.put(someKey, data);
   });
}
```

全局设置

```javascript
var app = angular.module('myApp',[])
  .config(['$httpProvider', function ($httpProvider) {
        // enable http caching
       $httpProvider.defaults.cache = true;
  }])
```

使用ng自带的$cacheFactory有个缺点就是不能设置过期时间，所以使用 [angular-cache](https://github.com/jmdobry/angular-cache)

```javascript
if (!CacheFactory.get('sourceCache')) {
  CacheFactory.createCache('sourceCache', {
    deleteOnExpire: 'aggressive',
    recycleFreq: 60000,
    maxAge: 60 * 60 * 1000
  });
}

var sourceCache = CacheFactory.get('sourceCache');
```

## 优化

写的时候注意一下ng的思想和语法就行，和React一样不要随意操作DOM(除了directive和componentDidMount中)

搬运工：[AngularJS性能优化心得](https://github.com/atian25/blog/issues/5)

## Tips

### 手动启动ng

```javascript
angualr.element(document).ready(function() {
  angular.module('mymodule',[]);
  angular.bootstrap(document,['mymodule']);
})
```

### update bindings

用jquery插件的时候用的到。

```javascript
$scope.$apply(function () {
    // update
});
```
一些时候会报错：`Error: $apply already in progress`

一种方法是：If scope must be applied in some cases, then you can set a timeout so that the $apply is deferred until the next tick

```javascript
$timeout(function(){ scope.$apply(); });
```
正确的方法(没试过，可以研究一下)：You are getting this error because you are calling $apply inside an existing digestion cycle.

The big question is: why are you calling $apply? You shouldn't ever need to call $apply unless you are interfacing from a non-Angular event. The existence of $apply usually means I am doing something wrong (unless, again, the $apply happens from a non-Angular event).

If $apply really is appropriate here, consider using a "safe apply" approach:

```javascript
$scope.safeApply = function(fn) {
  var phase = this.$root.$$phase;
  if(phase == '$apply' || phase == '$digest') {
    if(fn && (typeof(fn) === 'function')) {
      fn();
    }
  } else {
    this.$apply(fn);
  }
};
```

```javascript
$scope.safeApply(function() {
  alert('xxx');
});
```

### 关于双向绑定失效的问题

ng建议使用 `$scope.xxx.xxx` 的格式，有时候  `$scope.xxx` 双向绑定失效

### factory中使用service, 通过$injector注入

```javascript
var notify = $injector.get('notify');
notify({
  message: '网络请求错误，请查看网络设置',
  duration: '2000',
  position: 'left'
});
```

### 本地化

引入 `angular-locale_zh-cn.js`

### 使用$scope or this，控制器实例的别名机制

```javascript
function MainCtrl () {

  var vm = this;

  function doSomething() {

  }

  // exports
  vm.doSomething = doSomething;

}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
```

更好的写法，使用angular.extend

```javascript
function MainCtrl () {

  // private
  function someMethod() {

  }

  // public
  var someVar = { name: 'Todd' };
  var anotherVar = [];
  function doSomething() {
    someMethod();
  }

  // exports
  angular.extend(this, {
    someVar: someVar,
    anotherVar: anotherVar,
    doSomething: doSomething
  });
}

angular
  .module('app')
  .controller('MainCtrl', MainCtrl);
```

### 依赖注入更好的写法

```javascript
app.factory('AuthService', ['$http', '$rootScope', 'Session', 'ENV', function($http, $rootScope, Session, ENV) {
  // do
}]);
```

### Is everything attached to $scope being watched?  no!!

Angular team gave us two ways of declaring some $scope variable as being watched

一个是上文讲到的 `$apply`，还有一个就是 `$watch`，监听$scope的变化

```javascript
$scope.$watch('watchExpression', function(newValue, oldValue, scope) {

}, objectEquality);
```
### Module

在AngularJS中，有module的概念，但是它这个module，跟我们通常在AMD里面看到的module是完全不同的两种东西，大致可以相当于是一个namespace，或者package，表示的是一堆功能单元的集合。然并卵。这块可以优化一下用Requirejs。

### var newScope = $scope.$new()

Creates a new child scope.

### 事件

订阅发布模式

```javascript
app.factory("EventBus", function() {
    var eventMap = {};

    var EventBus = {
        on : function(eventType, handler) {
            //multiple event listener
            if (!eventMap[eventType]) {
                eventMap[eventType] = [];
            }
            eventMap[eventType].push(handler);
        },

        off : function(eventType, handler) {
            for (var i = 0; i < eventMap[eventType].length; i++) {
                if (eventMap[eventType][i] === handler) {
                    eventMap[eventType].splice(i, 1);
                    break;
                }
            }
        },

        fire : function(event) {
            var eventType = event.type;
            if (eventMap && eventMap[eventType]) {
                for (var i = 0; i < eventMap[eventType].length; i++) {
                    eventMap[eventType][i](event);
                }
            }
        }
    };
    return EventBus;
});
```
### $compile典型应用场景

```javascript
app.directive("otcDynamic", function($compile) {

    var template = "<button ng-click='doSomething()'>{{label}}</button>";

    return{
        link: function(scope, element){
            element.on("click", function() {
                scope.$apply(function() {
                    var content = $compile(template)(scope);
                    element.append(content);
               })
            });
        }
    }
});
```

### $parse

 manually parse an expression, you can inject the $parse service into a controller and call the service to do the parsing for you

```html
 <div ng-controller="MyCtrl">
   <input ng-model="expr" type="text" placeholder="Enter an expression" />
     <h2>{{ parsedValue }}</h2>
 </div>
```

```javascript
 angular.module("myApp", [])
  .controller('MyCtrl',['$scope', '$parse', function($scope, $parse) {
     $scope.$watch('expr', function(newVal, oldVal, scope) {
       if (newVal !== oldVal) {
         // Let's set up our parseFun with the expression
         var parseFun = $parse(newVal);
         // Get the value of the parsed expression
          $scope.parsedValue = parseFun(scope);
       }
     });
  }]);
```

类似于js中的apply，call

$parse takes an expression, and returns you a function. When you call the returned function with context (more on that later) as first argument. It will execute the expression with the given context.

```javascript
angular.module('my-module', [])
  .directive('myClick', function ($parse) {
     return {
         link: function (scope, elm, attrs) {
             var onClick = $parse(attrs.myClick);
             elm.on('click', function (e){
                 // The event originated outside of angular,
                 // We need to call $apply
                 scope.$apply(function () {
                     onClick(scope, {$event: e});
                 });
             });
         }
     }
});
```

```html
<a href="" ng-click="doSomething($event.target)">link</a>
```

### angular.element &&  jqLite

### 通信的两种方法，服务与事件

There are multiple ways how to communicate between controllers.

The best one is probably sharing a service:

```javascript
function FirstController(someDataService)
{
  // use the data service, bind to template...
  // or call methods on someDataService to send a request to server
}

function SecondController(someDataService)
{
  // has a reference to the same instance of the service
  // so if the service updates state for example, this controller knows about it
}
```

Another way is emitting an event on scope:

```javascript
function FirstController($scope)
{
  $scope.$on('someEvent', function(event, args) {});
  // another controller or even directive
}

function SecondController($scope)
{
  $scope.$emit('someEvent', args);
}
```

In both cases, you can communicate with any directive as well.

### ng-model doesn't work inside ng-if

[http://stackoverflow.com/questions/18342917/angularjs-ng-model-doesnt-work-inside-ng-if](http://stackoverflow.com/questions/18342917/angularjs-ng-model-doesnt-work-inside-ng-if)

### When the browser autofills the fields, the scope doesn’t get updated.

[http://timothy.userapp.io/post/63412334209/form-autocomplete-and-remember-password-with](http://timothy.userapp.io/post/63412334209/form-autocomplete-and-remember-password-with)

# 参考

比较好的写法参考

- [angularjs-style-guide](https://github.com/mgechev/angularjs-style-guide/blob/master/README-zh-cn.md)

- [https://github.com/johnpapa/angular-styleguide](https://github.com/johnpapa/angular-styleguide)

- [Google's AngularJS Style Guide](http://google.github.io/styleguide/angularjs-google-style.html)

好多内容来源于网络。。。未完，待补充。。。