+++
date = "2016-01-14T20:54:55-08:00"
title = "Object Fit"
+++

Today I was watching a random video on ES6 tooling when I saw the author use a CSS property I'd never heard of. `object-fit`, and it was pretty awesome.

>Method of specifying how an object (image or video) should fit inside its box. object-fit options include "contain" (fit according to aspect ratio), "fill" (stretches object to fill) and "cover" (overflows box but maintains ratio), where object-position allows the object to be repositioned like background-image does.

_Or_ It's similar to CSS background-size, where you can adjust how an image scales as you resize the page.

This is something I'd never even heard of before, so I decided to give it a test.
Now according to [Can I Use](http://caniuse.com/#feat=object-fit), object-fit is supported on modern Firefox, Chrome, and has partial support on Safari. The partial support on Safari is due to it not supported `object-position`.


### Demo

Now the demo is pretty simple. I create a list of images and size them up to 50% of the viewport's width and height.

```html
<ion-pane ng-controller="MainCtrl as main">
  <ion-header-bar class="bar-positive">
    <h1 class="title">Blank Starter</h1>
  </ion-header-bar>
  <ion-content>
    <div class="row row-wrap">
      <div class="col col-50" ng-repeat="image in main.images">
        <img src="https://unsplash.it/200/300/?random" alt="" />
      </div>
    </div>
  </ion-content>
</ion-pane>
```

There's some Angular behind there to just ease the process of creating all the images. The real magic comes from the CSS

```css
img{
  display: block;
  width: 50vw;
  height: 50vh;
  object-fit: cover

}
```
`object-fit: cover` will size the image so that no matter what the dimensions are, the images aspect ratio will always be preserved.


This makes creating something like an image gallery so much easier since you don't have to worry about the images dimensions. You can just use `object-fit` and know everything will be work.


<p data-height="268" data-theme-id="0" data-slug-hash="xZXGGX" data-default-tab="result" data-user="mhartington" class='codepen'>See the Pen <a href='http://codepen.io/mhartington/pen/xZXGGX/'>Object-fit</a> by Mike Hartington (<a href='http://codepen.io/mhartington'>@mhartington</a>) on <a href='http://codepen.io'>CodePen</a>.</p>
<script async src="//assets.codepen.io/assets/embed/ei.js"></script>


