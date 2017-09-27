---
title: "Ionic Colors Map: How to use them correctly"
date: 2017-09-27T16:15:09-04:00
thumbnail: "path/thumbnail.jpg"
draft: true
---


If you jumped from Ionic 1 to Ionic 2 when it was first released, a new thing that was added was this concept of `$colors`. Something that looks like this :

```scss
$colors: (
  primary:    #488aff,
  secondary:  #32db64,
  danger:     #f53d3d,
  light:      #f4f4f4,
  dark:       #222
);
```

This was a big change from how we did styles in Ionic 1, where all of our theme colors were defined upfront:

```scss
$light:                           #fff !default;
$stable:                          #f8f8f8 !default;
$positive:                        #387ef5 !default;
$calm:                            #11c1f3 !default;
$balanced:                        #33cd5f !default;
$energized:                       #ffc900 !default;
$assertive:                       #ef473a !default;
$royal:                           #886aea !default;
$dark:                            #444 !default;
```

The major difference here is that at compile time, we can generate the css rule using the map, instead of having to manually write the rules for each variable.

From both a framework developer and consumer of a framework, this is great! Now we'll only get the correct css rules, for any entry in `$colors`.

### Where this can go wrong

So how can this be misused? Well, consider having a `$colors` map that looks like this:

```scss
$colors: (
  primary: #387ef5,
  secondary: #32db64,
  danger: #f53d3d,
  light: #f4f4f4,
  dark: #222,
  dark1: #222,
  dark2: #222,
  dark3: #222,
  dark4: #222,
  dark5: #222,
  dark6: #222,
  dark7: #222,
  dark8: #222,
  dark9: #222,
  darka: #222,
  darkz: #222,
  darke: #222,
  darkr: #222,
  darkt: #222,
  darky: #222,
  darku: #222,
  darki: #222,
  darko: #222,
  darkp: #222,
  darkq: #222,
  darks: #222,
  darkd: #222,
  darkf: #222,
  darkg: #222,
);
```

> Don't worry about the dark entries being the same, it's just an example

Where we have 10+ entries in our map. Remember, the compiled css will depend on how many entries you have in `$colors`, so now we'll get all the rules for every single color entry. So our css could go from being very optimized, to **reallly** heavy.

Ok, so we get a heavy css file as an output, this can't be that bad, can it? Well it turns out that it can be very bad!

At some point, browsers will just stop trying to render a page if there's just too much css to parse. This can lead to loading a "blank" page. It's not actually blank, just that there's so much css, the page will not display.

As far as to a why this happens, "too much css" is all I can really provide. I'm not sure what happens internally in the browser when this happens.

### So, what can we do?

A common reason why we see this is that there is a misunderstanding with `$colors`. It's not meant to be a "catch all" place to define sass variables, it's meant to generate app-level themes that are tied to your branding. If you need to have a few sass variables that are not going to be used with UI components, then they don't belong in the colors map.

Even better, instead of adding additional colors to the map, override the values of the map. It's main purpose wasn't to be a "here are the Ionic colors, do not touch", but an approachable way to creating some sort of application theme.

So rule of thumb, if you need a color that is going to be used in UI components, replace an existing value, else use standard sass variables instead.

