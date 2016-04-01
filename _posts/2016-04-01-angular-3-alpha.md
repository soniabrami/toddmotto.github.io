---
layout: post
permalink: angular-3-alpha
title: A sneak peak at Angular 3 alpha
path: 2016-04-01-angular-3-alpha.md
---

<img src="/img/posts/angular-3.jpg">

Austin ([@amcdnl](https://twitter.com/amcdnl)) and I are huge Angular 1.x and 2.x fans, and have taken some time out to get some behind the scenes scoop on the Angular 3 developments. The Google team have been working long and hard at delivering Angular 2, and with it's imminent release, we're excited about the Angular 3 developments. Here's the lowdown on things to come in 2017:

### Components

Components are inspired by the upcoming Angular 2 release, but use much more expressive and powerful syntax. A Component boilerplate looks something like this, however these APIs may be subject to change in forthcoming months:

{% highlight javascript %}
export class Toggle@Component(::@scope(() => { 'button' )) {
  async constructor(@Variable()::MyService: service?) => {
    await super(...@Compile(arguments));
  }
  @ngOnInit(scope)
  init(scope: NgScope) {...}
  translcude(@Element() elm) => {
    transclude();
  }
}
{% endhighlight %}

### Template syntax with ASX

After AtScript was dropped in Angular 2's development, the Angular team have worked very closely with React to create a fork of JSX's internal virtual DOM implementation, dubbed "ASX".

<blockquote class="twitter-tweet" data-conversation="none" data-lang="en-gb"><p lang="en" dir="ltr"><a href="https://twitter.com/spcfran">@spcfran</a> <a href="https://twitter.com/toddmotto">@toddmotto</a> <a href="https://twitter.com/michlbrmly">@michlbrmly</a> If folks are wondering, ASX is pronounced &#39;ask&#39;.</p>&mdash; Brad Green (@bradlygreen) <a href="https://twitter.com/bradlygreen/status/716000239291932673">1 April 2016</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

An implementation for our above `Toggle` Component might look something like this:

{% highlight javascript %}
import {ButtonComponent} from './button';
import {ToggleSwitcher} from './toggleSwitcher';

export class Toggle@Component(::@scope(() => { 'button' )) {

  ...

  template(@Render('ASX')::ElementNode: element?) {
    return (
      <div class="toggle">
        <ButtonComponent />
        <ToggleSwitcher />
      </div>
    );
  }

}
{% endhighlight %}

ASX is an extremely powerful and expressive language, and ships with a vast amount of internal shortcuts, we can refactor the above to this using the `ASX-micro` parsing module:

{% highlight javascript %}
...

  template(@Render('ASX')::ElementNode: element?) {
    => div[class=toggle]
      match [element] with
        | 0 -> ButtonComponent
        | 1 -> ToggleSwitcher
      /div
  }

...
{% endhighlight %}

### Expressions

There's no longer the requirement for `{% raw %}{{ foo }}{% endraw %}` style handlebar templating expressions, we can use the `ASX-exp` module to handle inline JavaScript with the `@EXP()` decorator:

{% highlight javascript %}
...

  template(@Render('ASX')::ElementNode: element?) {
    => div[class=toggle]
      match [element] with
        | 0 -> ButtonComponent
          @EXP() => {
            ::this.innerHTML = 'Click me!';
          }
        | 1 -> ToggleSwitcher
          @EXP() => {
            ::this.onClick -> (element);
          }
      /div
  }

...
{% endhighlight %}

### Built-in Directives

We're familiar with Angular 1.x concepts such as `ng-repeat`, now `ngFor` in Angular 2. In Angular 3, we can import the ASX "EXP" renderer that handle this stuff for us when passing in specific arguments, such as `@EXP('repeat')`. An example to iterate over a collection:

{% highlight javascript %}
...

  template(@Render('ASX')::ElementNode: element?) {
    => ul[class=my-list]
        li:@EXP('repeat', collection) ? (value, index) => {
          ::this.innerHTML = value.name;
          ::this.setAttribute('data-index', index);
        } : null
      /ul
  }

...
{% endhighlight %}

The `?` operator here is used to conditionally assign custom renderers to the internal Angular 3 compiler, which gets handed down to Zones for change detection.

### Async forms

Forms are one of the most powerful Angular 3 features, let's explore the new features and API syntax. Declaring a form inside a Component:

{% highlight javascript %}
export class Form@Component(::@scope(() => { 'form' )) {
  async constructor(@Variable()::FormService: service?) => {
    await super(...@Compile(arguments));
  }
  @ngOnInit(scope)
  init(scope: NgScope) {
    ::this.setPristine();
  }
  template(@Render('ASX')::ElementNode: element?) {
    => form
        input:@EXP('bind', value) ? value => {
          ::this.setModelValue(value.label);
        } : null
      /form
  }
}
{% endhighlight %}

Complex validation comes built-in and is extremely automated and simple to use:

{% highlight javascript %}
export class Form@Component(::@scope(() => { 'form' )) {
  async constructor(@Variable()::FormService: service?) => {
    await super(...@Compile(arguments));
  }
  @ngOnInit(scope)
  init(scope: NgScope) {
    ::this.setPristine();
  }
  template(@Render('ASX')::ElementNode: element?) {
    => form
        input:@EXP('bind', value) ? value => {
          ::this.setModelValue(value.label);
          ::this.validate(@Validators():['email', 'min-length=5']);
        } : null
      /form
  }
}
{% endhighlight %}

There's also an `ASX-forms` module available that allows for async expression evaluations to be carried out on form fields, which means users can use multiple cursors to fill in multiple fields at once, and Angular spins up another emulated UI thread to handle the complex Model diffing.

### Event handling

Events are abstracted and wrapped in the `ASX-events` module, instead of listening for events, you simply ask for the events you want. This works via Angular 3's dependency injection process and simplifies the DOM abstractions, leaving you to just focus on building.

Let's assume we want to listen to the `submit` event, we do this by importing `Events` from the Angular 3 core and binding where necessary:

{% highlight javascript %}
import {Events} from 'angular3/core/events';

export class Form@Component(::@scope(() => { 'form' )) {
  ...
  template(@Render('ASX')::ElementNode: element?) {
    => form@Event('submit') => {
      ::this.onSubmit(form);
    }
        ...
      /form
  }
}
{% endhighlight %}

### Hot reloading > Zones

Future plans for Angular 3 include removing the entire Zone detection, carefully ported from the Dart programming language, to make way for hot-reloading. Instead of Zones, we'll have native hot reloading, a feature in `stage-0` of the ECMAScript proposal and is being heavily considered by TC39.

`Babel` has already made ~54% implementation of native hot reloading features, with `ASX` having 92% support Instead of a heavy change detection strategy, hot reloading simply _knows_ natively when Components and data-models have changed, and will hot reload necessary Components and DOM nodes to reflect data changes. The hot reload API ships with a model-syncing counterpart to first re-render the View, and then update the Model. User feedback was critical before updating JavaScript models.

### Upgrade path

With the rise of Angular 2, the Angular team have pushed the upgrade technology forward even further, creating a simple AST transformer to automatically upgrade _any_ Angular 2 app to an Angular 3 application through generated code transitioning. You can grab the module on GitHub or use npm:

{% highlight sh %}
cd <my-project>
npm install ng2-transformer-asx
ng2 upgrade --april-fools
{% endhighlight %}

### Getting started

There's a simple tutorial on the [Angular3.io](https://www.youtube.com/watch?v=dQw4w9WgXcQ) webpage that will get you kickstarted. Enjoy!
