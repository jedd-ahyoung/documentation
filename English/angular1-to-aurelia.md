# Upgrading to Aurelia from Angular 1

## Introduction

Angular 1 has been abandoned in favor of Google's new initiative, Angular 2 - leaving many existing applications in need of a rewrite. Rewriting an existing Angular 1 application doesn't have to be difficult, though. With a thorough understanding of modern web standards, emerging language features, and the Javascript ecosystem, you can migrate an existing Angular 1 application to Aurelia.

## Requirements

To ensure that you are ready to begin migrating your application, you will need to have NodeJS and JSPM installed.

You'll need access to your AngularJS application.

Migration will be simpler if your application is:
 1. using Angular's `controller as` syntax in views
 2. assigning properties to `this` instead of `$scope` in controllers
 3. your angular modules are separated per file

## Getting started

### Loading script dependencies

Some existing Angular applications use a loader such as RequireJS, but some applications load their scripts through script tags. Aurelia has been designed to adhere to the SystemJS loading specification; by default, it uses JSPM for package management, Babel for code compilation, and SystemJS for loading.

### Bootstrap and configuration

Angular applications typically bootstrap automatically, using the `ng-app` attribute on an HTML element like so:

``` language-markup
<!DOCTYPE html>
<html lang="en" data-ng-app="myApp">
<head>

    <title>My Angular Application/title>
...
```

Aurelia applications use a similar attribute, but are bootstrapped manually, like so:


Angular configuration generally uses providers....

```language-javascript
angular.config(['$locationProvider', '$routeProvider', function ($locationProvider, $routeProvider) {
	...
}])
```

### Modules and dependency injection

Like Aurelia, Angular supported modules and dependency injection. A user-defined service, however, had to be registered with the Angular global object at bootstrap time in order to be used. Finally, Angular made a distinction between `service`, `factory`,  `constant`, and `value` - all syntactic sugar for `provider`. All of this meant that code might look like the code below.

```language-javascript
angular.module('myModule')
	.service('myService', ['$q', '$http',function ($q, $http) {
		...
	}]);

angular.module('myModule')
	.service('anotherService', ['$q', '$router',function ($q, $router) {
		...
	}]);

angular.module('myModule')
	.factory('myFactory, ['$storage', function ($storage) {

	}]);

angular.module('myModule')
	.value('myValue', "value")
	... // and so on
```

Aurelia handles things a little differently. Instead of framework-specific modules, Aurelia leverages ES6 modules using the import statement. (These end up transpiled to AMD modules for browser use.) Additionally, Aurelia makes no special framework distinction between "services", "factories", and other module types, simplifying the way you can write your code.

So, the angular code above ends up looking more like this:

```language-javascript

@inject(Q, HttpClient)
export class myService {
	constructor (Q, http) {
		...
	}
}

@inject(Q, Router)
export class anotherService {
	constructor (Q, router) {

	}
}

@inject(Storage)
export class myFactory {
	constructor (storage) {

	}
}

export var myValue = value;
```

As long as you define your exports semantically, Aurelia doesn't need to register or define them in a specific way ("factory" vs "service" vs "value"). Everything is an exported module; modules can be imported and then injected using Aurelia's dependency injection.

For more information on dependency injection, see the documentation here.

### Views and Viewmodels

Angular views and controllers were declared at bootstrap time; controllers were registered using the `angular` object.

```language-javascript
angular.controller('myCtrl', ['$scope', '$http', '$otherDependency', function ($scope, $http, $otherDependency) {
	...
}])
```

#### Naming Conventions

In Angular, a view was often strongly coupled to a viewmodel (or controller) instance, using something similar to the following code:

```language-markup
<section data-ng-controller="myController as MyCtrl">
...
</section>
```

Aurelia, by default, prefers convention over configuration. Thus, simply name your viewmodel and your view with the same name.

This behavior is configurable using the ViewLocator service.

#### $scope

Angular made heavy use of an Angular-specific service named `$scope`; responsible for computed properties, change detection (through `$scope.watch()`), signalling (via `$scope.broadcast` and `$scope.emit`), and view-to-viewmodel binding, this service was often a staple of an Angular controller.

A typical scenario might yield something like this:

```language-javascript
code here
```

(Later versions of Angular supported the "controller as" syntax, allowing developers to declare viewmodel properties on the `this` property. However, $scope was still necessary for watches and signalling.)

In Aurelia, viewmodel properties are declared as properties on the viewmodel object.

The Aurelia version of our typical Angular controller might look like this:

```language-javascript
code here
```

#### Template interpolation

Angular, by default, uses a double-bracket syntax for interpolation, like so:

```language-markup
<div id="hello">Hello, my name is {{name}}!</div>
```

Aurelia adheres to web standards for string templating, so the Aurelia version becomes this:

```language-markup
<div id="hello">Hello, my name is ${name}!</div>
```

#### Repeating data

Angular:

```language-markup
<ul data-ng-repeat="item in items">
	<li id="{{item.id}}">{{item.name}}</li>
</ul>
```

Aurelia:

```language-markup
<ul>
	<li repeat.for="item of items" id="${item.id}">${item.name}</li>
</ul>
```

#### Event binding
Angular:
```language-markup
<button data-ng-click="myFunction(argument)">myFunction button</button>
```
Aurelia:
```language-markup
<button click.delegate="myFunction(argument)">myFunction button</button>
```

### Templating

#### Compose binding handler

#### Angular directives

## Quick Reference

#### $scope

No equivalent exists in Aurelia.

Instead of `$scope.property`, use `this.property` inside of a viewmodel object.

Instead of `$scope.$watch()` use a computed property #### or, to trigger a function, use the BindingEngine.

Instead of `$scope.$emit()` and `$scope.$broadcast()`, use `EventAggregator.publish()`. Likewise, instead of $scope.$on, use `EventAggregator.subscribe()`.

#### $rootScope
#### $timeout

No equivalent exists in Aurelia #### it's not needed. Aurelia will detect changes made from `setTimeout`. Use `setTimeout`.

#### $cacheFactory

Not sure yet.

#### $animate/$animateCss

Use one of the 'aurelia-animate' plugins.

#### $controller
#### $document

Use `window.document`.

#### $filter
#### $http/$httpBackend
#### $locale
#### $location

Use router.navigateToRoute or router.navigate.

#### $log

Use the logger.

#### $q

No equivalent exists in Aurelia; Aurelia will detect changes made asynchronously. Use ES6 native promises or a promise library.

#### $templateCache
#### $window

Use window.


