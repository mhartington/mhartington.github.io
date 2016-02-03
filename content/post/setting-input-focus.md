+++
date = "2016-02-03T09:37:41-05:00"
title = "Setting Focus to an Input in Ionic 2"

+++


A Question came up in the Ionic Worldwide Slack today about how to set focus to an input. Now normally you'd think you would be able to call `.focus()` and call it day. But Ionic wraps native text inputs with custom Angular 2 components to better control the user experience.


### The markup

Let's take this simple Ionic 2 page


```javascript

import {Page} from 'ionic-framework/ionic';
@Page({
  template: `
  <ion-navbar *navbar>
    <ion-title>Tab 1</ion-title>
  </ion-navbar>
  <ion-content>
  <ion-list>
    <ion-item>
      <ion-label>Home</ion-label>
      <ion-input #input type="text"></ion-input>
    </ion-item>
    <button>Focus</button>
  </ion-list>
</ion-content>`
})

export class Page1 {
  constructor() { }
}
```

Now we really just want to focus on this little part

```
<ion-item>
  <ion-label>Home</ion-label>
  <ion-input #input type="text"></ion-input>
</ion-item>
<button>Focus</button>
```

Now that `#input` is setting a local variable of input to this page, and we can interact with it when we click our button


```
<ion-item>
  <ion-label>Home</ion-label>
  <ion-input #input type="text"></ion-input>
</ion-item>
<button (click)="focusInput(input)">Focus</button>
```

### The method call

Since our button is calling `focusInput(input)`, passing along the input we have, we can use that in our method.

```javascript
export class Page1 {
  constructor() { }
  focusInput(input) {}
}
```

You could `console.log` and see all the properties that are on ion-input, but we only care about a method on it called `setFocus`. From it's name, you can kind of guess what it does.

```javascript
  focusInput(input) {
    input.setFocus();
  }
```

Now when we run this, if we click our button, it will set focus to the input.


### Bonus: Making the keyboard appear on a device

Now setting focus to an input and getting the keyboard to appear on a device can be quite a pain. But it's rather simple to get it to work. Just add the following to your cordova's config.xml


```xml

<preference name="KeyboardDisplayRequiresUserAction" value="false" />

```

Now when you call `focusInput`, the keyboard will display as well.

