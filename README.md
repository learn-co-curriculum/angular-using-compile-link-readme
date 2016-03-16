# Using the compile/link functions

## Overview

We used the `link` function in our previous lab, however we don't actually know much about it. Let's look into `link` and the power it provides us.

## Objectives

- Describe the link function
- Describe the compile function
- Describe the Directive's lifecycle

## Directive Lifecycle

Each directive has a lifecycle. Up until now we've only been using one part of the lifecycle - the controller. The controller is initiated and ran just before the directive is about to be mounted into the DOM. This is so we can manipulate any values that we need in time for the rendering.

However, it might come to the point where we need to actually do raw DOM manipulation on the directive itself. For instance, we might have a jQuery plugin that straps into a DOM element - we'd need the actual DOM element in order to initiate it. One problem - the controller is initiated *before* the directive is in the DOM. What do we do?


### `link`

This is where the `link` function comes in. If we specify this, we get a function executed *after* the directive has been mounted in the DOM. We also then get the DOM element passed through to us as the second parameter.

```js
function someDirective() {
  return {
    template: [
	  '<div>',
	    'Hello!',
	  '</div>'
	].join(''),
    link: function ($scope, $element, $attrs) {

    }
  };
}

angular
  .module('app', [])
  .directive('someDirective', someDirective);
```

You can see above that we get three parameters (they must be in that order!). `$scope` - much like what we can inject into our controllers. `$element`, our DOM mounted element, and `$attrs`, an object of all the attributes used when the directive is initiated.

### `compile`

Compile allows us to manipulate the element before it is inserted into the DOM. This runs in between the `controller` and the `link` function. This gives us access to the unmounted DOM node, and the attributes.

```js
function someDirective() {
  return {
    template: [
      '<div>',
        'Hello!',
      '</div>'
    ].join(''),
    compile: function ($element, $attrs) {

    }
  };
}

angular
  .module('app', [])
  .directive('someDirective', someDirective);
```

In our compile function, we'd then return an object with `pre` and `post` values. Both of these are functions.

These are `pre-link` and `post-link` functions, giving us access to every single point in the directive lifecycle.

Take a look at this diagram of the lifecycle:

![http://i.stack.imgur.com/XRDs6.png](Image)

This means that if we put some `console.log` calls in our compile function, as so:

```js
function someDirective() {
  return {
    template: [
      '<div>',
        'Hello!',
      '</div>'
    ].join(''),
    controller: function () {
        console.log('controller');
    },
    compile: function ($element, $attrs) {
        console.log('compile');

        return {
            pre: function (scope, element, attrs) {
                console.log('pre-link');
            },
            post: function (scope, element, attrs) {
                console.log('post-link');
            }
        }
    }
  };
}

angular
  .module('app', [])
  .directive('someDirective', someDirective);
```

We would get this printed out in the console:

```js
compile
controller
pre-link
post-link
```

What are all of these stages?

- `compile` - ready to compile the directive
- `controller` - manipulate all data before the directive is actually compiled
- `pre-link` - the directive is compiled to DOM nodes, but isn't inserted into the DOM
- `post-link` - the directive has been inserted into the DOM (same as the link function)

Generally, we should be using the pre-link (compile) function to do any DOM manipulation - moving nodes, changing HTML/styles etc.

We'd want to use the post-link (link) function to add event listeners.

This gives us full control over what we want to do with our directives.