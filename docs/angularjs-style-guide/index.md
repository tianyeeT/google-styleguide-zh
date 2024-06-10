# Google AngularJS 风格指南

这份文档是为谷歌工程师编写的外部版本，主要描述了在谷歌内部使用的采用 Closure 的 AngularJS 应用程序的推荐风格。更广泛的 AngularJS 社区的成员可以根据自己的用例自由地应用（或不应用）这些建议。

该文档描述了在 google3 中的 AngularJS 应用程序的风格。此指南是对 Google JavaScript 风格指南的补充和扩展。

**风格注意事项：** AngularJS 外部网页上的示例以及许多外部应用程序的写法自由使用闭包，偏爱函数继承，并且不经常使用 JavaScript 类型。谷歌遵循更严格的 JavaScript 风格，以支持 JSCompiler 优化和大型代码库——请参阅 javascript-style 邮件列表。这不是一个特定于 Angular 的问题，在本风格指南中不再进一步讨论。（但如果你想进一步阅读：[Martin Fowler 关于闭包的文章](http://martinfowler.com/bliki/Lambda.html)、[更详细的描述](http://jibbering.com/faq/notes/closures/)、[闭包书](https://books.google.com/books/about/Closure_The_Definitive_Guide.html?id=p7uyWPcVGZsC)的附录A有关于继承模式的描述以及为什么它更喜欢伪经典风格的解释，以及[《JavaScript权威指南》](https://books.google.com/books/about/JavaScript.html?id=PXa2bby0oQ0C)作为对照。）

## Angular 语言规则

### 使用 Closure 的 goog.require 和 goog.provide 来管理依赖关系

为你的项目选择一个命名空间，并使用 goog.provide 和 goog.require。

```javascript
goog.provide('hello.about.AboutCtrl');
goog.provide('hello.versions.Versions');
```

**为什么？** Google BUILD 规则与 closure provide/require 集成得很好。

### 模块

你的主应用程序模块应该位于根客户端目录中。模块除了定义它的文件外，不应进行任何修改。

模块可以在与其组件相同的文件中定义（对于包含完全一个服务的模块来说，这样做效果很好），也可以在单独的文件中用于连接各个部分。

**为什么？**一个模块应该对任何希望将其包含为可重用组件的人都是一致的。如果一个模块根据包含的文件不同而意味着不同的东西，那就不是一致的。

### 模块应使用 Angular 模块的“name”属性引用其他模块

例如：

```javascript
// 文件 submodulea.js:
  goog.provide('my.submoduleA');

  my.submoduleA = angular.module('my.submoduleA', []);
  // ...

// 文件 app.js
  goog.require('my.submoduleA');

  正确: my.application.module = angular.module('hello', [my.submoduleA.name]);
  
      错误: my.application.module = angular.module('hello', ['my.submoduleA']);
```

**为什么？** 使用 `my.submoduleA` 的属性可以避免 Closure 的 presubmit 失败，该失败会提示错误显示该文件是需要的但从未被使用。使用 `.name` 属性可以避免重复字符串。

### 使用一个常用的外部文件

这样可以最大程度地让 JS 编译器在存在来自 Angular 的外部提供的类型时强制执行类型安全，并且意味着你不必担心 Angular 变量以令人困惑的方式被混淆。

对于 Google 外部的读者：当前的外部文件位于一个 Google 内部的目录中，但是你可以在 github 上找到一个[示例](https://github.com/angular/angular.js/pull/4722)。

### JSCompiler 标志

**提醒：** 根据 JS 风格指南，面向客户的代码必须经过编译。

**推荐：** 使用 JSCompiler（默认情况下与 `js_binary` 一起工作的 closure 编译器）和 //javascript/angular/build_defs/build_defs 中的 `ANGULAR_COMPILER_FLAGS_FULL` 作为基本标志。

**注意：** 如果你正在使用 `@export` 导出方法，则需要添加编译器标志。

```sh
"--generate_exports",
```

如果你正在使用 `@export` 导出属性，则需要添加以下标志：

```sh
"--generate_exports",
"--remove_unused_prototype_props_in_externs=false",
"--export_local_property_definitions",
```

### 控制器和作用域

控制器是类。方法应该在 `MyCtrl.prototype` 上定义。

Google Angular 应用程序应该使用 `'controller as'` 风格将控制器导出到作用域上。这在 Angular 1.2 中已完全实现，并且可以在 Angular 1.2 之前的构建中模仿。

在 Angular 1.2 之前，看起来像这样：

```javascript
/**
 * Home 控制器。
 *
 * @param {!angular.Scope} $scope
 * @constructor
 * @ngInject
 * @export
 */
hello.mainpage.HomeCtrl = function($scope) {
  /** @export */
  $scope.homeCtrl = this; // 这是一个桥梁，直到 Angular 1.2 实现 controller-as

  /**
   * @type {string}
   * @export
   */
  this.myColor = 'blue';
};


/**
 * @param {number} a
 * @param {number} b
 * @export
 */
hello.mainpage.HomeCtrl.prototype.add = function(a, b) {
  return a + b;
};
```

模板：

```html
<div ng-controller="hello.mainpage.HomeCtrl"/>
  <span ng-class="homeCtrl.myColor">I'm in a color!</span>
  <span>{{homeCtrl.add(5, 6)}}</span>
</div>
```

在 Angular 1.2 之后，看起来像这样：

```javascript
/**
 * Home 控制器。
 *
 * @constructor
 * @ngInject
 * @export
 */
hello.mainpage.HomeCtrl = function() {
  /**
   * @type {string}
   * @export
   */
  this.myColor = 'blue';
};


/**
 * @param {number} a
 * @param {number} b
 * @export
 */
hello.mainpage.HomeCtrl.prototype.add = function(a, b) {
  return a + b;
};
```

如果你正在使用属性重命名进行编译，请使用 `@export` 注释来公开属性和方法。记得也要 `@export` 构造函数。

模板：

```html
<div ng-controller="hello.mainpage.HomeCtrl as homeCtrl"/>
  <span ng-class="homeCtrl.myColor">I'm in a color!</span>
  <span>{{homeCtrl.add(5, 6)}}</span>
</div>
```

**为什么？** 将方法和属性直接放在控制器上，而不是构建一个作用域对象，更符合 Google Closure 类风格。此外，使用 `'controller as'` 使得在一个元素上应用多个控制器时可以明确地看到正在访问的控制器。由于绑定中始终存在一个 `'.'`，因此不必担心原型继承掩盖基本类型。

### 指令

所有 DOM 操作应该在指令内完成。指令应该保持小巧，并使用组合。定义指令的文件应该提供一个静态函数，该函数返回指令定义对象。

```javascript
goog.provide('hello.pane.paneDirective');

/**
 * 描述和使用
 * @return {angular.Directive} 指令定义对象。
 */
hello.pane.paneDirective = function() {
  // ...
};
```

**例外情况：** 在服务中可能会出现 DOM 操作，用于与视图的其余部分断开连接的 DOM 元素，例如对话框或键盘快捷方式。

### 服务

在模块上使用 `module.service` 注册的服务是类。除非需要进行初始化以创建类的新实例之外，否则应使用 `module.service` 而不是 `module.provider` 或 `module.factory`。

```javascript
/**
 * @param {!angular.$http} $http Angular http 服务。
 * @constructor
 */
hello.request.Request = function($http) {
  /** @type {!angular.$http} */
  this.http_ = $http;
};

hello.request.Request.prototype.get = function() {/*...*/};
```

在模块中：

```javascript
module.service('request', hello.request.Request);
```

## Angular 风格规则

### 保留 `$` 用于 Angular 属性和服务

不要使用 `$` 来添加自己的对象属性和服务标识符。考虑到 AngularJS 和 jQuery 保留了此类命名风格。

正确：

```javascript
$scope.myModel = { value: 'foo' }
myModule.service('myService', function() { /*...*/ });
var MyCtrl = function($http) {this.http_ = $http;};
```

错误：

```javascript
$scope.$myModel = { value: 'foo' } // 错误
$scope.myModel = { $value: 'foo' } // 错误
myModule.service('$myService', function() { ... }); // 错误
var MyCtrl = function($http) {this.$http_ = $http;}; // 错误
```

**为什么？** 区分 Angular / jQuery 的内置内容和你自己添加的内容是有用的。此外，在 JS 风格指南中，`$` 不是变量名称的可接受字符。

### 自定义元素

对于自定义元素（例如 `<ng-include src="template"></ng-include>`），IE8 需要特殊支持（类似 html5shiv 的 hack）才能启用 CSS 样式。在针对旧版本 IE 的应用程序中要注意这个限制。

## Angular 提示、技巧和最佳实践

这些不是严格的风格指南规则，但放在这里作为 Angular 刚开始的人的参考。

### 测试

Angular 设计用于测试驱动开发。

推荐的单元测试设置是 Jasmine + Karma（虽然你可以使用 closure tests 或 js_test）。

Angular 提供了轻松的适配器来加载模块并在 Jasmine 测试中使用注入器。

- [module](https://docs.angularjs.org/api/angular.mock.module)
- [inject](https://docs.angularjs.org/api/angular.mock.inject)

### 考虑使用应用程序结构的最佳实践

这个[目录结构文档](https://docs.google.com/document/d/1XXMvReO8-Awi1EZXAXS4PzDzdNvV6pGcuaF4Q9821Es/pub)描述了如何使用嵌套子目录来组织你的应用程序，并将所有组件（例如服务和指令）放在一个“components”目录中。

### 注意作用域继承的工作原理

查看[作用域原型继承的细微差别](https://github.com/angular/angular.js/wiki/Understanding-Scopes#wiki-JSproto)

### 使用 `@ngInject` 进行简单的依赖注入编译

这样可以避免添加 `myCtrl['$inject'] = ...` 来防止缩小操作破坏 Angular 的依赖注入。

用法：

```javascript
/**
 * 我的控制器。
 * @param {!angular.$http} $http
 * @param {!my.app.myService} myService
 * @constructor
 * @export
 * @ngInject
 */
my.app.MyCtrl = function($http, myService) {
  //...
};
```

## 最佳实践链接和文档

- Angular 在 GitHub 上的[最佳实践](https://github.com/angular/angular.js/wiki/Best-Practices)
- [Meetup 视频](https://www.youtube.com/watch?v=ZhfUv0spHCY)（非 Google 特定）