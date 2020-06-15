---
title: "HMR in Ionic Angular"
date: 2020-06-15T10:28:58-04:00
thumbnail: ""
draft: false
---


Something I've heard from a few people in the Ionic community is that the miss the updates speed from Ionic App scripts when updating CSS. They often ask "how can I use app scripts with V5?" or "Why is Angular CLI so much slower?". Well it's a bit more involved that just those blanketed statements, but it is possible wit the Angular CLI flag `--hmr`.


## App Scripts and Styles

For those who don't know, App Scripts was a project we made at Ionic that was essentially the Angular CLI, but before the Angular CLI was built. It was a full featured build pipeline that processed your Angular Components (HTML, CSS, and TS) and created the necessary JavaScript and CSS needed for the browser. A key feature about it was that it treated CSS just as regular CSS. Meaning everything was hoisted globally, and was just loaded via a link tag. The cons for this were that we didn't follow Angular best practices or enabled style scoping, but the pros where that we could support live updates to an app's CSS. This was possible because we didn't treat CSS as a JS module. We just loaded CSS as you would any CSS library and would inject any updates on the fly.


## Angular CLI and Webpack

Since V4 and the move to use a frameworks specific tooling, we've adopted the Angular CLI for all of our tooling needs. Under the hood, the Angular CLI is using Webpack to process your CSS, HTML, and TS. This is good in that we have consistent tooling, but not great when you consider that Webpack treats everything as a JS module. So a CSS file is treat the same as JavaScript and any changes will require a full refresh for them to take effect. Not great.

## Hot Module Replacement

Now Hot Module Replacement (HMR) is a method in which incoming changes to JS or CSS will be automatically injected into an app without requiring a full refresh. This sounds great in theory, though it can lead to some problems like stale app state between hard reloads. But HMR **can** let CSS updates be replaced on the fly like the old method we had in App Scripts.

Let's look at an example. Here we'll have a standard tabs based app that is served via `ionic serve`:

{{< vid src="./images/non-hmr.mp4" >}}


Here, when ever we make a change to CSS (or in this case, SCSS) file, we do a full refresh of the browser.


Now let's make a change to to our serve command. We'll use the `--hmr` flag to tell the Angular CLI to enable Hot Module Replacement.

```bash
ionic serve -- --hmr
```

> Notice the additional `--` in there. This is just a way to tell the current process to pass the following flag to the child process.

With this in place, changes to style will be injected without a full reload.

{{< vid src="./images/with-hmr.mp4" >}}


Pretty awesome, huh?

## Parting comments

The style updates with HMR in Angular does provide a lot of improvements to the feedback cycle. There are still some edge cases with regards to components being updated, but for those wanting faster style changes... There you go!
