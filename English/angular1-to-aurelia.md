# Migrating to Aurelia from Angular 1

## Introduction

Angular 1 has been discarded in favor of Google's new initiative, Angular 2 - leaving many existing applications in need of a rewrite. Rewriting an existing Angular 1 application doesn't have to be difficult, though. With a thorough understanding of modern web standards, emerging language features, and the Javascript ecosystem, you can migrate an existing Angular 1 application to Aurelia.

## Requirements

To ensure that you are ready to begin migrating your application, you will need to have NodeJS and JSPM installed, along with access to your AngularJS application.

Migration will be simpler if your application is:

 1. using Angular's `controller as` syntax in views
 2. assigning properties to `this` instead of `$scope` in controllers
 3. separated across files, using one file per Angular module

## The migration

There are five main things we have to do to migrate an Angular application to Aurelia. **First**, we need to change the way we load our scripts, adhering to the SystemJS loader model. **Second**, we'll configure and bootstrap our application in Aurelia. **Third**, we'll migrate our modules from Angular's dependency injection to Aurelia's injection; **fourth**, we'll migrate our views and viewmodels to work with Aurelia. **Finally** (and this is the big one), we'll learn how to migrate Angular directives to Aurelia custom elements.

It's recommended to start with the Aurelia Skeleton Navigation project, as loading and bootstrapping have already been provided.

### Loading script dependencies

First, we'll need to change the way we load our scripts. Some existing Angular applications use a loader such as RequireJS, but some applications load their scripts through script tags. Aurelia has been designed to adhere to the ES6 module loading specification; by default, it uses JSPM for package management, Babel for code compilation, and SystemJS for loading.

### Bootstrapping and configuring the application

Second, we'll bootstrap and configure our application in Aurelia. Angular applications typically bootstrap automatically, using the `ng-app` attribute on an HTML element to call a specific Angular module, like so:

``` language-markup
<!DOCTYPE html>
<html lang="en" data-ng-app="myApp">
<head>

    <title>Skeleton Navigation</title>
...
```

Aurelia uses a similar attribute that should feel familiar, like so:

``` language-markup
<!DOCTYPE html>
<html lang="en">
<head>

    <title>Skeleton Navigation</title>

</head>
<body aurelia-app="main">
 ...
</body>
```

Instead of a module name, the `aurelia-app` attribute takes a filename as its value (without the extension).

> NOTE: The `aurelia-app` attribute will never be on the root `html` tag.

Configuring an Aurelia application at startup is slightly different than its Angular equivalent. In Angular, the application is configured with a static set of options - Angular configuration, application controllers, services, etc. As such, service configuration in Angular only occurs before Angular bootstraps the applicaiton. Services in Aurelia can be configured at any time - not just at startup.

Angular might look something like this. In this configuration block, we might configure certain services - such as interceptors for HTTP requests, or client-side routing templates - but after Angular has bootstrapped, configuration is locked.

```language-javascript
angular.config(['$locationProvider', '$routeProvider', function ($locationProvider, $routeProvider) {
	... // We configure certain services here, but after Angular has bootstrapped, configuration is locked.
}])
```

Aurelia uses a similar configuration model, but services don't have to be configured before they are used; configuration is generally only used for global resource locations and framework plugins.

```language-javascript
export function configure(aurelia) {
  aurelia.use
    .standardConfiguration() //
    .plugin('aurelia-animator-css')
    .feature('someFeature')
    .globalResources('resources/my-element', 'resources/my-other-element');

  aurelia.start().then(a => a.setRoot());
}
```

This means that certain important service configurations (especially routing and HTTP) can be configured during application runtime. We'll touch upon this later.

### Using Modules and Dependency Injection

Third step - we'll migrate our modules from Angular to Aurelia. Like Aurelia, Angular supported modules and dependency injection; however, Angular used a non-standard, Angular-specific module format. Whether loaded through a loader or through generic script tags, Angular declared a global `angular` object. Using any controller, service, or other functionality with Angular meant that it had to be registered with an Angular module at bootstrap time in order to be used. An example Angular configuration might look like the code below.

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

Aurelia handles things a little differently. Instead of framework-specific modules, Aurelia leverages native Javascript modules using either the ES6 import statement or the `require`-style AMD syntax. (These end up transpiled to AMD modules for browser use.) When registering objects with Angular modules Angular made a distinction between `service`, `factory`,  `constant`, and `value` - all syntactic sugar for `provider`. Aurelia, in contrast, makes no special framework distinction between "services", "factories", and other module types - any viewmodels or services are just simple Javascript modules, simplifying the way you can write your code.

So, the angular code above might end up looking more like this:

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

As long as you define your exports semantically, Aurelia doesn't need to register or define them in a specific way ("factory" vs "service" vs "value"). Everything is an exported module; everything is generic, and modules can be imported and then injected using Aurelia's dependency injection.

For more information on dependency injection, see the documentation here.

### Migrating your ViewModels

Fourth, we'll migrate our viewmodels.

In Angular, a controller must be registered with a specific name, using the `angular` global object.

```language-javascript
function Users ($http, $scope) {
	...
}

angular.controller('users', ['$scope', '$http', Users]);
```

Using Aurelia, any object can act as a viewmodel without being explicitly registered, as long as that object is exported as a module. Essentially, this simply means removing the Angular boilerplate registration code. So, essentially, you would have this:

```language-javascript
function Users (httpClient) {
	...
}
```

> NOTE: Angular was able to inject Angular-specific dependencies based on the argument name (such as `$scope` and `$http`) through its service locator. Because Aurelia does not couple your application to Aurelia-specific implementations, all dependencies must be explicitly injected.

#### Routing

Aurelia supports client-side routing in the browser. The main difference between Angular's router and Aurelia's router is in the configuration; the Aurelia router can be configured outside of startup, and it can be configured asynchronously. Finally, the Aurelia router supports nested routers (also known as child routers) as well as multiple viewports.

An Angular router configuration, in startup, may look like this:

Alternatively, an Aurelia router is configured inside the viewmodel that uses it. Thus, Aurelia configuration may look like this:

```language-javascript
export class App {
  configureRouter(config, router) {
    config.title = 'Aurelia';
    config.map([
      { route: ['', 'welcome'], name: 'welcome',      moduleId: 'welcome',      nav: true, title: 'Welcome' },
      { route: 'users',         name: 'users',        moduleId: 'users',        nav: true, title: 'Users' }
    ]);

    this.router = router;
  }
}
```

> NOTE: Migration to Aurelia will be easier if your application uses Angular's new router; this router shares many similarities with the router that Aurelia uses, and the configuration will be easier to migrate.

#### Removing `$scope`

Angular made heavy use of an Angular-specific service named `$scope`; responsible for computed properties, change detection (through `$scope.watch()`), signalling (via `$scope.broadcast` and `$scope.emit`), and view-to-viewmodel binding, this service was often a staple of an Angular controller.

A typical controller might yield something like this:

```language-javascript
angular.controller('Users', ['$scope', '$http', function ($scope, $http) {
	$scope.heading = 'Welcome to the Angular Navigation App!';	$scope.firstName = 'John';
	$scope.lastName = 'Doe';
	$scope.previousValue = $scope.fullName();
	
	$scope.fullName = function () {
		return $scope.firstName + ' ' + $scope.lastName;
	};
	
	$scope.submit = function () {
		$scope.previousValue = $scope.fullName();
		alert('Welcome, ' + $scope.fullName());
	};
}])
```

In Aurelia, viewmodel properties are declared as properties on the viewmodel object.

The Aurelia version of our typical Angular controller might look like this:

```language-javascript
export class Welcome {
  heading = 'Welcome to the Aurelia Navigation App!';
  firstName = 'John';
  lastName = 'Doe';
  previousValue = this.fullName;

  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }

  submit() {
    this.previousValue = this.fullName;
    alert(`Welcome, ${this.fullName}!`);
  }
}
```

> NOTE: Later versions of Angular supported the "controller as" syntax, allowing developers to declare viewmodel properties on the `this` property. However, `$scope` was still necessary for watches and signalling.) Your application will be easier to migrate to Aurelia if you are only using `$scope` for signalling.

### Migrating your Views

Fourth step, part two - we'll migrate the views. Aurelia views and Angular views are very similar, but Aurelia views must be wrapped in a `<template>` tag, and references to custom views or custom elements must be included in the view using the `<require>` tag.

#### Naming Conventions

In Angular, a view was often strongly coupled to a viewmodel (or controller) instance, using something similar to the following code:

```language-markup
<section data-ng-controller="myController as MyCtrl">
...
</section>
```

Aurelia, by default, prefers convention over configuration. Aurelia's default convention is that a viewmodel will have a view with the same name. To take advantage of this, keep a view in the same directory alongside its viewmodel, and make sure that the HTML file and Javascript file share the same filename.

This behavior is also configurable using the ViewLocator service; by configuring Aurelia's ViewLocator, views and viewmodels can be linked by location, by a file naming convention, or by other means.

#### Changing Angular interpolation to Template String Interpolation

Angular, by default, uses a double-bracket syntax for interpolation, like so:

```language-markup
<div id="hello">Hello, my name is {{name}}!</div>
```

Aurelia makes use of ES6 template strings in its templates, so the Aurelia version becomes this:

```language-markup
<div id="hello">Hello, my name is ${name}!</div>
```

Aurelia's template strings use pipe syntax for ValueConverters (filters in Angular).

#### Template behaviors

`ng-show` becomes `show.bind`. `ng-if` becomes `if.bind`. `ng-click` becomes `click.delegate`.

Angular's repeat syntax looks like this:

```language-markup
<ul data-ng-repeat="item in items">
	<li id="{{item.id}}">{{item.name}}</li>
</ul>
```

In Aurelia, the repeat is not on the parent element. It looks like this:

```language-markup
<ul>
	<li repeat.for="item of items" id="${item.id}">${item.name}</li>
</ul>
```

Angular:
```language-markup
<button data-ng-click="myFunction(argument)">myFunction button</button>
```
Aurelia:
```language-markup
<button click.delegate="myFunction(argument)">myFunction button</button>
```

#### Converting `ng-include` to `<compose>`

Angular:
```language-markup
<div data-ng-include="myView"></div>
```
Aurelia:
```language-markup
<compose view-model="myViewModel" view="myView"></compose>
```

### Converting Angular directives to Custom Elements and Attributes

Finally, the meat and potatoes - it's time to migrate our Angular directives.

Angular and Aurelia both support the concept of extending HTML by creating custom elements and attributes. However, Aurelia handles the creation of these in a different way.

Custom elements in Angular required an Angular directive; we return a configuration object that tells Angular how to create our object through object properties including `scope`, `restrict`, `template`, `templateUrl`, and `link`.

Instead of a configuration object, Aurelia takes a class instance.

An Angular directive may have looked like this:

```language-javascript
angular.directive('display', ['$window', '$document', 'animate', function ($window, $document, animate) {
	return {
		scope: true,
		restrict: 'A',
		templateUrl: 'myTemplate.html',
		link: function (scope, element, attrs) {
			var windowheight = angular.element($window).height();

			scope.holeHeight = function (depth) {
				return (depth * 0.2) < windowheight ? (depth * 0.2) + 'px' : undefined;
			};

			scope.bgPos = function (depth) {
				return (depth * 0.2) < windowheight ? 0 : (depth - windowheight) * 0.2;
			};

			scope.holeWidth = function () {
				return Math.min(700, Math.max(350, _(scope.shop)
					.reduce(function (p, c) {
						return p + c.owned * c.digValue;
					}, 0)
				));
			};

			scope.displayArray = [];

			scope.$watch('shop', function () {
				scope.displayArray = _.values(scope.shop).reverse();
			});

			(function tick(timestamp) {
				angular.element($window).scrollTop($document.height());
				animate(tick);
			})();
		}
	};
}])
```

Converting this directive to an Aurelia custom element requires us to remove the scope references, migrate from `link` and `controller` properties to lifecycle methods, and simplify our object. Finally, we'll have to use Aurelia's decorator functions to register our object and its configuration as a custom element instance.

```
export class Display {
	constructor (window, document, animate) {
		this.window = window;
		this.document = document;
		this.animate = animate;
	}
	
	attached () {
	
	}
}
```

#### Custom Attributes

Custom attributes are handled similarly to custom elements in Aurelia.

#### Transclusion

Aurelia provides the `content` tag to provide transclusion features. This is part of the view itself, instead of being part of the directive.



####

## Quick Reference

#### $scope

No equivalent exists in Aurelia.

Instead of `$scope.property`, use `this.property` inside of a viewmodel object.

Instead of `$scope.$watch()`, use a computed property - or, to trigger a function, use the BindingEngine.

Instead of `$scope.$emit()` and `$scope.$broadcast()`, use `EventAggregator.publish()`. Likewise, instead of $scope.$on, use `EventAggregator.subscribe()`.

#### $rootScope

For signalling (using `$emit` and `$broadcast`, use Aurelia's event aggregator. For state that persists across viewmodels, use a separate service.

#### $timeout

No equivalent exists in Aurelia; it's not needed as Aurelia's change detection works differently to change detection in Angular. Aurelia will detect changes made from `setTimeout`, so use that.

#### $cacheFactory

Not sure yet.

#### $animate/$animateCss

Use one of the 'aurelia-animate' plugins.

#### $controller


#### $document

Use `window.document`.

#### $filter

Use a `ValueConverter`, providing your own `toView` and `fromView` methods.

#### $http/$httpBackend

Use `fetch-client`. If you need cancellable requests, or IE9 support, use `http-client`.

#### $locale


#### $location

Use router.navigateToRoute or router.navigate.

#### $log

Use the logger.

#### $q

No equivalent exists in Aurelia; Aurelia will detect changes made asynchronously. Use ES6 native promises or a promise library.

#### $templateCache


#### $window

Use the `DOM` object from Aurelia's Platform Abstraction Layer (PAL).


