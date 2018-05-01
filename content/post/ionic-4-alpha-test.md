---
title: "Ionic 4 Alpha Test"
date: 2018-05-01T08:17:01-05:00
thumbnail: "/img/v4-upgrade/ionic-icon.png"
draft: true
---

**IONIC 4**! It's a thing that we (the Ionic team) have been hard at work on for the last few months. While we're currently finishing up our testing, I wanted to touch a bit on how to actually setup up and start initially testing 4.0.

**Warning, Ionic 4 is ALPHA, meaning things are not finalized and ready for production. Use with caution**

### CLI Update

It just so happens that the Ionic CLI is also nearing it's 4.0 release, which we'll need in order to create a new project type.

```bash

npm install -g ionic@rc

```

> Depending on your setup, you might need to add `sudo` in order to work with permissions.

With this installed, we now have access to a new project type, `angular`.

You can access this project type by passing the flag `--type=angular` when you start a new project.

```bash
ionic start myApp blank --type=angular
```

After finishing install and project creation, cd in to your directory and open your editor.

### Project structure

One of the major changes between a V3 app and a V4 is the over all project layout and structure. In V3, we created our own convention for how an app should be setup and what that folder structure should look like. In V4, we've moved away from this to something that follows a typical Angular project. This change, while not too difficult to update, keep us in the typical Angular setup.


![A V4 project on the left and a V3 project on the right](/img/v4-upgrade/v4-vs-v3-project-setup.png)
_V4 on the left, V3 on the right_


On the left, we have the newer V4 layout. If you've work in a vanilla Angular project, this should feel really familiar. We have a `src` directory that acts as the home for our webapp. This includes the `index.html`, any assets, environment variables, and any app specific config files. Compared to a V3 project, which is similar, just slightly different.

In migrating your app to take advantage of this new layout, we don't suggest trying to make all these files yourself. Because the CLI is able to create a "base" project, it's easier to start a new project, and migrate the features of your app piece by piece.

Another different piece is the moving of pages/components/etc to inside the app folder. Since `app` is now essentially the home for all of our Angular specific code, this move makes sense.

### Changes in Package Name

Another change in V4 is the actual package name of Ionic. For V4, we moved to a scoped package, `@ionic/angular`. While you migrate your app over, update the imports from `ionic-angular` to `@ionic/angular`.

### Navigation

In V4, one of the major changes we made was in navigation and routing. We've deprecated the `ion-nav` component and `NavController` provider. While they're deprecated, you can still use them, if your app is not using lazy loading though. But we strongly suggest moving away from them.

In place of `ion-nav` and `NavController`, we've moved to the `@angular/router` package for our navigation/routing needs. Again, this brings us closer to a typical setup for an Angular project. If you're not familiar with the Angular router, there's an [excellent guide](http://angular.io/guide/router) on the Angular docs site that covers everything in great detail.

One difference that you'll need to know is that instead of using the `router-outlet` component, you'll use the `ion-router-outlet` for an Ionic app. This component has the same functionality as the standard Angular `router-outlet`, but just includes the logic to animate pages.

### Lazy Loading

Now if `ion-nav` and `NavController` are deprecated, how can we setup our app for lazy loading. We can use the Angular Router for this! Let's take a typical a sample app in V3.

```js
// home.page.ts
@IonicPage({
  segment: 'home'
})
@Component({...})
export class HomePage{}

//home.module.ts
@NgModule({
  declarations: [HomePage],
  imports: [IonicPageModule.forChild(HomePage)]
})
export class HomePageModule {}
```

If we wanted to upgrade this logic to V4, we can do the follow.

```js
// home.module.ts
@NgModule({
  imports: [
    IonicModule,
    RouterModule.forChild([{ path: '', component: HomePage }])
  ],
  declarations: [HomePage]
})
export class HomePageModule {}

// app.module.ts
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    IonicModule.forRoot(),
    RouterModule.forRoot([
      { path: 'home',loadChildren: './pages/home/home.module#HomePageModule'},
      { path: '', redirectTo: 'home', pathMatch: 'full'}
    ])
  ],
  bootstrap: [AppComponent]
})
export class AppModule {}
```

In V3, most of the logic for lazy loading was done in the `@IonicPage` decorator. But as this have been removed in V4, we can utilize the Angular Router and it's `loadChildren` property to correctly lazy load the component.

### Modifications to Overlays

Now while routing and navigation have gotten the biggest changes, it's time to look at a part of Ionic that's still pretty much the same, overlays! Overlays are pretty much the same as they were in V3, though some slight changes in the option names and how the alerts are created. In V3, when you created an overlay, the overlay instance was return after the `create` call. In V4 though, `create` returns a promise.

```js
// V3
showAlert(){
  let alert = this.alertCtrl.create({
    message: "Hello There",
    subHeader: "I'm a subheader"
  })
  alert.present()
}
```

In V4, we can do:

```js
showAlert(){
  this.alertCtrl.create({
    message: "Hello There",
    subHeader: "I'm a subheader"
  }).then(alert => alert.present())

// Or if you like async/await

async showAlert(){
  let alert = await this.alertCtrl.create({
    message: "Hello There",
    subHeader: "I'm a subheader"
  })
alert.present()
```

### Markup Changes

Since Ionic moved to Custom Elements, there's been a significant change to the markup for each component. These changes have all been made to follow the Custom Elements spec. We've documented these changes in a [dedicated file on Github](https://github.com/ionic-team/ionic/blob/master/angular/BREAKING.md#breaking-changes)


### Getting Help and Providing feedback

We (the Ionic team) are looking to gather some feedback and see how we can improve the upgrade/migration process for developers. If you do run into issues, please make a post on the Ionic forum. Since the forum content search about a last a while, it is preferred over a slack post.


