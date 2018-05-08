---
title: "Angular Schematics and Dynamic Content"
date: 2018-05-08T18:07:00-04:00
draft: false
---

Recently, I've been diving into the new tooling setup being worked on for upcoming `ionic/angular@4.0`. Since we've moved all of our tooling over to the Angular CLI, we get to take advantage of new features, like Angular's Schematics. Schematics is a pipeline for building out new files/features in an app. What's even more impressive, is that schematics itself can be used in a non-angular project. In theory, you could have a Vue or P/React project, and it could also use schematics, though without some utility libs that exist for Angular.


### Adding routePath

One of the newest features we added to our `page` schematic was the ability to specify the path for the router config. For example, if you ran:

```bash

ionic g page user

```

We'd generate a page for `user`, but also a route config that looks like:

```js
{ path: 'user', loadChildren: './pages/user/user.module#UserPageModule'  },
```


With the new `routePath` option, we could run:

```bash
ionic g page user --routePath="user/:userId"
```

And create a router config that looks like:


```js
{ path: 'user/:userID', loadChildren: './pages/user/user.module#UserPageModule'  },
```

This is really nice for users who want to just start scaling their app right away.


### Conditional templates


While this was already a nice feature, I wanted to take it further by conditionally adding the correct imports and injecting `ActivatedRoute` if the user passed `routePath`.

To do this, I needed to look at the templating system used for schematics. Diving into the [template core](https://github.com/angular/devkit/blob/master/packages/angular_devkit/core/src/utils/template.ts#L341-L350), it's loosely based on EJS, meaning we can do some standard if statements in our templates. Basically, we have a EJS template tag, `<% %>`, meaning anything inside of this tag can be evaluated as JavaScript. The most common place you see this is inside of class declarations when we want to transform the class name.

```js

export class <%= upperFirst(camelCase(name)) %>Page implements OnInit {

}

```

So for our conditional blocks, we can simply add

```js
<% if(condition) { %> <div>HTML will render when condition evals to true</div> <% } %>
```

So for out import statement, we can add:

```js

<% if(routePath) { %>import { ActivatedRoute, Params } from '@angular/router';<% } %>

```

And have this added if the user passes a `routePath` during the generate process.

Finishing off the full component class, we ended up with:

```js

import { Component, OnInit } from '@angular/core';
<% if(routePath) { %>import { ActivatedRoute, Params } from '@angular/router';<% } %>
@Component({
  selector: '<%= selector %>',
  templateUrl: './<%= kebabCase(name) %>.page.html',
  styleUrls: ['./<%= kebabCase(name) %>.page.<%= styleext %>'],
})
export class <%= upperFirst(camelCase(name)) %>Page implements OnInit {

  <% if(routePath) { %>public params: Params;<% } %>

  constructor(<% if(routePath) { %>private route: ActivatedRoute <% } %>) { }

  ngOnInit() {
    <% if(routePath) { %>this.params = this.route.snapshot.params;<% } %>
  }

}
```

