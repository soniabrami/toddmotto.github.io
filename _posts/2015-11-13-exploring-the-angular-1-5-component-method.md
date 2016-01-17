---
layout: post
permalink: /exploring-the-angular-1-5-component-method
title: Exploring the Angular 1.5 .component() method
path: 2015-11-13-exploring-the-angular-1-5-component-method.md
tags:
- New features
- Performance
- Components
- Directives
- Angular
---

Angular 1.5 is set to introduce the `.component()` helper method, which is much simpler than the `.directive()` definition and advocates best practices and common default behaviours. Using `.component()` will allow developers to write in an Angular 2 style as well, which will in turn make upgrading to Angular 2 an easier feat.

Let's compare the differences in syntax and the super neat abstraction that `.component()` gives us over using the `.directive()` method.

_Please note: Angular 1.5 is still in `beta` phase, keep an eye out for it's release!_

### Update: use component() now in Angular 1.3+

I've back-ported the Angular 1.5 `component()` functionality to Angular 1.3 and above! [Read the article](/angular-component-method-back-ported-to-1.3) and grab the [code on GitHub](https://github.com/toddmotto/angular-component).

### Screencast

My screencast overview from [Voux.io](http://voux.io):

<div class="embed">
  <iframe src="https://www.youtube.com/embed/Ca5rzXGmld4?rel=0&amp;showinfo=0" frameborder="0" allowfullscreen></iframe>
</div>

### .directive() to .component()

The syntax change is very simple:

{% highlight javascript %}
// before
module.directive(name, fn);

// after
module.component(name, options);
{% endhighlight %}

The `name` argument is what we want to define our Component as, the `options` argument is a definition Object passed into the component, rather than a function that we know so well in versions 1.4 and below.

I've prebuilt a simple `counter` component for the purposes of this exercise in Angular `1.4.x` which we'll refactor into a version `v1.5.0` build to use `.component()`.

{% highlight javascript %}
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter',
    template: [
      '<div class="todo">',
        '<input type="text" ng-model="counter.count">',
        '<button type="button" ng-click="counter.decrement();">-</button>',
        '<button type="button" ng-click="counter.increment();">+</button>',
      '</div>'
    ].join('')
  };
});
{% endhighlight %}

A live embed of the `1.4.x` Directive:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/avdezer7/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

We'll continue building this alongside how we'd build the Angular 1.4 version to compare differences.

### Function to Object, method name change

Let's start from the top and refactor the `function` argument to become an `Object`, and change the name from `.directive()` to `.component()`:

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    
  };
});

// after
.component('counter', {
    
});
{% endhighlight %}

Nice and simple. Essentially the `return {};` statement inside the `.directive()` becomes the Object definition inside `.component()` - easy!

### "scope" and "bindToController", to "bindings"

In a `.directive()`, the `scope` property allows us to define whether we want to isolate the `$scope` or inherit it, this has now become a sensible default to (usually) always make our Directives have isolate scope. So repeating ourselves each time just creates excess boilerplate. With the introduction of `bindToController`, we can explicitly define which properties we want to pass into our isolate scope and bind directly to the Controller.

With the `bindings` property on `.component()` we can remove this boilerplate and simply define what we want to pass down to the component, under the assumption that the component will have isolate scope.

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    }
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  }
});
{% endhighlight %}

### Controller and controllerAs changes

Nothing has changed in the way we declare `controller`, however it's now a little smarter.

If we're using a controller local to the component, we'll do this:

{% highlight javascript %}
// 1.4
{
  ...
  controller: function () {}
  ...
}
{% endhighlight %}

If we're using another Controller defined elsewhere, we'll do this:

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl'
  ...
}
{% endhighlight %}

If we want to define `controllerAs` at this stage, we'll need to create a new property and define the instance alias:

{% highlight javascript %}
// 1.4
{
  ...
  controller: 'SomeCtrl',
  controllerAs: 'something'
  ...
}
{% endhighlight %}

This then allows us to use `something.prop` inside our `template` to talk to the instance of the Controller.

Now, there are some changes in `.component()` that make sensible assumptions and automatically create a `controllerAs` property under the hood for us, and automatically assign a name based on three possibilities:

{% highlight javascript %}
// inside angular.js
controllerAs: identifierForController(options.controller) || options.controllerAs || '$ctrl',
{% endhighlight %}

Possibility one uses this aptly named `identifierForController` function that looks like so:

{% highlight javascript %}
// inside angular.js
var CNTRL_REG = /^(\S+)(\s+as\s+(\w+))?$/;
function identifierForController(controller, ident) {
  if (ident && isString(ident)) return ident;
  if (isString(controller)) {
    var match = CNTRL_REG.exec(controller);
    if (match) return match[3];
  }
}
{% endhighlight %}

This allows us to do the following inside `.component()`:

{% highlight javascript %}
// 1.5
{
  ...
  controller: 'SomeCtrl as something'
  ...
}
{% endhighlight %}

This saves adding the `controllerAs` property... _however_...

We can add the `controllerAs` property to maintain backwards compatibility or keep it if that's within your style for writing Directives/Components.

The third option, and better yet, completely removes all need to think about `controllerAs`, and Angular automatically uses the name `$ctrl`. For instance:

{% highlight javascript %}
.component('test', {
  controller: function () {
    this.testing = 123;
  }
});
{% endhighlight %}

The would-be `controllerAs` definition automatically defaults to `$ctrl`, so we can use `$ctrl.testing` in our `template` which would give us the value of `123`.

Based on this information, we add our `controller`, and refactor our Directive into a Component by dropping the `controllerAs` property:

{% highlight javascript %}
// before
.directive('counter', function counter() {
  return {
    scope: {},
    bindToController: {
      count: '='
    },
    controller: function () {
      function increment() {
        this.count++;
      }
      function decrement() {
        this.count--;
      }
      this.increment = increment;
      this.decrement = decrement;
    },
    controllerAs: 'counter'
  };
});

// after
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  }
});
{% endhighlight %}

Things are becoming much simpler to use and define with this change.

### Template

There's a subtle difference in the `template` property worth noting. Let's add the `template` property to finish off our rework and then take a look.

{% highlight javascript %}
.component('counter', {
  bindings: {
    count: '='
  },
  controller: function () {
    function increment() {
      this.count++;
    }
    function decrement() {
      this.count--;
    }
    this.increment = increment;
    this.decrement = decrement;
  },
  template: [
    '<div class="todo">',
      '<input type="text" ng-model="$ctrl.count">',
      '<button type="button" ng-click="$ctrl.decrement();">-</button>',
      '<button type="button" ng-click="$ctrl.increment();">+</button>',
    '</div>'
  ].join('')
});
{% endhighlight %}

The `template` property can be defined as a function that is now injected with `$element` and `$attrs` locals. If the `template` property _is_ a function then it needs to return an String representing the HTML to compile:

{% highlight javascript %}
{
  ...
  template: function ($element, $attrs) {
    // access to $element and $attrs
    return [
      '<div class="todo">',
        '<input type="text" ng-model="$ctrl.count">',
        '<button type="button" ng-click="$ctrl.decrement();">-</button>',
        '<button type="button" ng-click="$ctrl.increment();">+</button>',
      '</div>'
    ].join('')
  }
  ...
}
{% endhighlight %}

Let's checkout the live working example with Angular version `v1.5.0-build.4376+sha.aff74ec`:

<iframe width="100%" height="300" src="//jsfiddle.net/toddmotto/xqauz9aa/embedded/result,js,html" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

That's it for our Directive to Component refactor, however there are a few other changes worth exploring before we finish.

### Disabling isolate scope

Components are always created with isolate scope. Here's the relevant part from the source code:

{% highlight javascript %}
{
  ...
  scope: {},
  ...
}
{% endhighlight %}

### Sourcecode for comparison

Throughout the article I've referred to some Angular source code snippets to cross reference against. You can [find the source code here](https://github.com/angular/angular.js/blob/54e816552f20e198e14f849cdb2379fed8570c1a/src/loader.js#L362-L396) or check it out below, it's a really nice abstraction:

{% highlight javascript %}
component: function(name, options) {
  function factory($injector) {
    function makeInjectable(fn) {
      if (angular.isFunction(fn)) {
        return function(tElement, tAttrs) {
          return $injector.invoke(fn, this, {$element: tElement, $attrs: tAttrs});
        };
      } else {
        return fn;
      }
    }

    var template = (!options.template && !options.templateUrl ? '' : options.template);
    return {
      controller: controller,
      controllerAs: identifierForController(options.controller) || options.controllerAs || '$ctrl',
      template: makeInjectable(template),
      templateUrl: makeInjectable(options.templateUrl),
      transclude: options.transclude,
      scope: {},
      bindToController: options.bindings || {},
      restrict: 'E'
    };
  }

  if (options.$canActivate) {
    factory.$canActivate = options.$canActivate;
  }
  if (options.$routeConfig) {
    factory.$routeConfig = options.$routeConfig;
  }
  factory.$inject = ['$injector'];

  return moduleInstance.directive(name, factory);
}
{% endhighlight %}

Again, please note that Angular 1.5 isn't released just yet, so this article uses an API that _may_ be subject to slight change.

### Upgrading to Angular 2

Writing components in this style will allow you to upgrade your Components using `.component()` into Angular 2 very easily, it'd look something like this in ECMAScript 5 and new template syntax:

{% highlight javascript %}
var Counter = ng
.Component({
  selector: 'counter',
  template: [
    '<div class="todo">',
      '<input type="text" [(ng-model)]="count">',
      '<button type="button" (click)="decrement();">-</button>',
      '<button type="button" (click)="increment();">+</button>',
    '</div>'
  ].join('')
})
.Class({
  constructor: function () {
    this.count = 0;
  },
  increment: function () {
    this.count++;
  },
  decrement: function () {
    this.count--;
  }
});
{% endhighlight %}
