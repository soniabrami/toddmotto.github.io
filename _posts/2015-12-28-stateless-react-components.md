---
layout: post
permalink: /stateless-react-components
title: Stateless React components
path: 2015-12-28-stateless-react-components.md
---

React v0.14.0 introduces "stateless" components, which takes a more functional approach to creating chunks of JSX and templates. In my previous article on [creating a tab component](/creating-a-tabs-component-with-react) I created the components using the traditional way, using `React.createClass()` or ES6 Classes that extend `React.Component`.

### Traditional components

Prior to v0.14.0, we would create React components using `React.createClass()` (or extending `React.Component` with ES6 Classes) and pass in an Object definition. Taking the `Pane` component from the previous article, let's look how that looks.

Here we use `React.createClass()`, where we have a `propTypes` property and `render` property.

{% highlight javascript %}
const Pane = React.createClass({
  propTypes: {
    label: React.PropTypes.string.isRequired,
    children: React.PropTypes.element.isRequired
  },
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
});
{% endhighlight %}

Here we use `class Pane extends React.Component`, where we no longer have a `propTypes` property and instead move it to a property of the `class` itself, using `Pane.propTypes`. We keep the `render` property.

{% highlight javascript %}
class Pane extends React.Component {
  render() {
    return (
      <div>
        {this.props.children}
      </div>
    );
  }
}
Pane.propTypes = {
  label: React.PropTypes.string.isRequired,
  children: React.PropTypes.element.isRequired
};
{% endhighlight %}

This component has a `displayName` which isn't a necessity, however `propTypes` is a nice-to-have and `render` is pretty essential. It's a lot of boilerplate code for a component that simply passes `this.props.children` through into the new component.

### Enter stateless components

With stateless components, we can kill the entire boilerplate code and keep everything in (some cases) a single line of code.

Let's look at how we can refactor the above example to use a stateless component approach.

{% highlight javascript %}
const Pane = (props) => <div>{props.children}</div>;
{% endhighlight %}

Done.

Adding in `propTypes` we mirror the ES6 syntax by adding a property to the `Pane` constant:

{% highlight javascript %}
const Pane = (props) => <div>{props.children}</div>;
Pane.propTypes = {
  label: React.PropTypes.string.isRequired,
  children: React.PropTypes.element.isRequired
};
{% endhighlight %}

Note how we no longer refer to the `this` keyword, and the arguments of the stateless component function become the "props".

When supplying default props (or other Object definition props) to a component, this would also be done using `Pane.defaultProps`.
