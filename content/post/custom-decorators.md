+++
date = "2017-05-31T09:55:05-04:00"
tags = []
title = "Custom Decorators"
draft = false
+++

I've been getting more into decorators lately, especially since I used them in Angular/Ionic, but also in Python. But I realized that I really didn't know much about them or how they actually worked. So I spent some time last night and built a small decorator, `@platformReady()`.

### WTH is a decorators

Decorators can get confusing real fast, so we're going to try to stay pretty high level here. In its simplest form, a decorator allows developers to perform higher-level functions on an annotated class or method. Python has these already so Let's look at that for an example.

```python
class Client(object):

    @property
    def serverPath(self):
        return self._serverPath

    @serverPath.setter
    def serverPath(self, value):
        self._serverPath = value
```

Here we have a simple property getter/setter setup. The first method, `serverPath` is the getter, and it just returns a value. What's nice about this is when we want to get that value, we can just do `Client.serverPath` and it will know how to handle it. The second function is the setter for our property. So when we want to change the value of our property, we can just do `Client.serverPath = newValue`.


Without getting into too much detail, we can see "what" decorators are, and how they can be used from a conceptual level. In JavaScript, they follow the same idea, annotate classes or methods to provide additional functionality. [Addy wrote a blog post](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841) going into some more detail, so be sure to give that a read as well.

### Our decorator

The decorator we'll build addresses an issue I have when trying to call Cordova plugins in my Ionic apps, I always forget to wrap them in a device ready callback. So we'll create a `platformReady()` decorator that automatically handles this. Before going into detail, here is the decorator in it's [final form](https://gist.github.com/mhartington/5fdb572a561cd2e39d3362f36760aa55)


To start off, we're going to first want to create a function. We'll make a new file: `src/decorators/platform-ready.ts`

```js
export function platformReady() {

}
```

In this function, we're going to return a new function that has the information about our class/function. In this case, we'll just call it decorate, but you can name it what ever you want.


```js
  return function decorator(target, method, descriptor) {}
```

Now from these arguments, we have full access to our class and the method we're attached to. Inside of this function, we're going to do a bit of setup work for our decorator.

```js
return function decorator(target, method, descriptor) {
  const {
    ngOnInit = () => {},
    ngOnDestroy = () =>{}
  } = target;

  const symbolHandler = Symbol(method);
}
```

So the first `const` is creating `ngOnInit` and `ngOnDestroy` method on the class. We set these to `noop`s  so we they don't really step on users code. Then, we create a `symbolHandler` by passing the `method` to the `Symbol` function. Now we can manually call our method when we need to.

After our `symbolHandler` we'll add two more functions, just to set up our event listeners.

```js
function addListener() {
  const handler = this[symbolHandler] = (...args) => {
    descriptor.value.apply(this, args);
  };
  document.addEventListener(DEVICEREADYEVENT, handler);
}
function removeListener() {
  document.addEventListener(DEVICEREADYEVENT, this[symbolHandler]);
}
```

So this is a bit more involved, and could probably refactored. But the `addListener` function is setting up our event listener, and the `handler` is taking that, and passing the event to our method on the class. This is optional really, and doesn't need to be done, but was good to know it's possible.

The `removeListener` is a bit more self-explanatory, we're simple removing the event listener.

Our last bit of code is to wire up these event listeners so that they play nicely with Angular's event life cycle.

```js
target.ngOnInit = function() {
  ngOnInit.call(this);
  addListener.call(this);
};

target.ngOnDestroy = function() {
  ngOnDestroy.call(this)
  removeListener.call(this);
}
```

Since our we mapped our `ngOnInit` and `ngOnDestroy` to `noop`s, we can just pass through what ever the developer has setup in their own code. Then to finish things up, we simple call our `addListener` and our `removeListen`.


To better see what this looks like with some finishing touches, we have:

```js
const noop = () => { };
const DEVICEREADYEVENT = window['cordova'] ? "deviceready" : "DOMContentLoaded"
export function platformReady() {
  return function decorator(target, method, descriptor) {

    const { ngOnInit = noop, ngOnDestroy = noop } = target;
    const symbolHandler = Symbol(method);

    function addListener() {
      const handler = this[symbolHandler] = (...args) => {
        descriptor.value.apply(this, args);
      };
      document.addEventListener(DEVICEREADYEVENT, handler);
    }

    function removeListener() {
      document.addEventListener(DEVICEREADYEVENT, this[symbolHandler]);
    }

    target.ngOnInit = function onInitWrapper() {
      ngOnInit.call(this);
      addListener.call(this);
    };

    target.ngOnDestroy = function onDestroyWrapper() {
      ngOnDestroy.call(this)
      removeListener.call(this);
    }
  }
}
```

Then, when we want to use the decorator in our code, we can simply import it, and attach it to our method.

```js
import { Component } from '@angular/core';
import { NavController } from 'ionic-angular';
import { platformReady } from '../../decorators/platform-ready'
import { StatusBar } from '@ionic-native/status-bar';
@Component({
  ...
})
export class HomePage {
  constructor(
    public navCtrl: NavController,
    public statusbar: StatusBar
  ) { }
  ngOnInit(){
    console.log('on init')
  }
  @platformReady()
  onDeviceReady(event) {
    console.log('device ready called', event)
    this.statusbar.backgroundColorByName('red')
  }
}
```

Something to notice is that we're passing event through from our decorator to the method and that our `ngOnInit` will still log out "on init".

And to see it in action:

<center>

 ![device-ready](/img/device-ready-small.gif)

</center>


### Parting words

This is just a quick exploration of decorators and what they can do. As I stated, I expect this decorator could be improved upon and refactored to be much simpler.

If you're interested in some other decorators, or want to read the spec/proposal check the links down below.


- [TC39 Docs](http://tc39.github.io/proposal-decorators/)
- [Lodash Decorators](https://github.com/steelsojka/lodash-decorators)
- [Core Decorators](https://github.com/jayphelps/core-decorators.js)
- [Addy's Decorator post](https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841)
