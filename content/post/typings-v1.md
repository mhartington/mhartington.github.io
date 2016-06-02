+++
date = "2016-06-02T13:03:06-04:00"
title = "Revisiting Typings: Post 1.0"

+++

### Typings hits 1.0!

Typings finally hit its 1.0 recently (well more like a month or so ago), and with this it brought some drastic change to the API. So my previous post is now out of date.
This is a perfect chance to go over these changes and how it may affect your project.


### What new?

So a simple overview what's changed from 0.X to 1.X has been provided in the [README.md](https://github.com/typings/typings#updating-from-0x-to-10)

## Updating From `0.x` to `1.0`?

* `rm -rf typings/`
  * The directory contains "main", by default
  * Update `tsconfig.json` to match (the bundle file is `typings/index.d.ts`)
* Want `main` and/or `browser` output again? See [where the typings install](https://github.com/typings/typings/blob/master/docs/faq.md#where-do-the-type-definitions-install) in the FAQ
* Usages of `ambient` are now `global`
  * That means in `typings.json` any `ambientDependencies` should be renamed `globalDependencies` and any `ambientDevDependencies` should be renamed `globalDevDependencies`.
  * It also means `--ambient` is now `--global`
* Removed `defaultAmbientSource`
  * If you want to install from DefinitelyTyped, be explicit (use `dt~<pkg> --global`). For example: `typings install dt~angular-component-router --global --save`
* See [the release notes](https://github.com/typings/typings/releases/tag/v1.0.0) for more information!


While this is great, it also helps to go over some of the changes one by one.


### No more `main` and `browser`

One the of the more confusing parts about typings was the use of `main` and `browser` typings. I'll be honest, I never _really_ understood the difference, and it often confused many new users in Ionic 2. Thankfully, this is gone, and we now have a much more familiar `index`. This is just much easier to understand and follows convention already out there in the JS/TS ecosystem.


### Ambient becomes Global

Ambient typing always we a pain point when first being introduced to typescript.

>An ambient declaration introduces a variable into a TypeScript scope, but has zero impact on the emitted JavaScript program. Programmers can use ambient declarations to tell the TypeScript compiler that some other component will supply a variable. For example, by default the TypeScript compiler will print an error for uses of undefined variables. To add some of the common variables defined by browsers, a TypeScript programmer can use ambient declarations

Basically these types are there to tell the compiler that something will at run that has this API.

imagine if you are writing code that uses jQuery. If you just try to write $() TypeScript will think you are using an undeclared variable $ and will throw an error. Ambient declarations like declare var $ tell the TS compiler that, even though $ isn't visible to the compiler, it will exist when the JS is executed.


So why change the name?

I can't speak for the maintainers of typings, but Global makes more sense to me than Ambient as a choice of word. I hear global types, I think globals in terms of JavaScript.

So a little terminology change, and we get a better understanding of the types mean and do.


### You must define a source

While most types live in the DefinitelyTyped repo, many authors are publishing types to npm, bower, or directly with the the module.
Since we have more sources, we need to specify which source we want pull from.

Take Lodash for example. A quick search in typings will reveal that we have multiple sources for lodash.

```bash
typings search lodash
Viewing 4 of 4

NAME              SOURCE HOMEPAGE                                        DESCRIPTION VERSIONS UPDATED
lodash            dt     http://lodash.com/                                          2        2016-05-24T13:56:08.000Z
lodash            npm    https://www.npmjs.com/package/lodash                        1        2016-04-16T21:15:19.000Z
lodash            global                                                             1        2016-04-12T19:14:14.000Z
lodash-decorators dt     https://github.com/steelsojka/lodash-decorators             1        2016-03-16T15:55:26.000Z
```

How is typings supposed to know which one you want? Now we can specify which source we want to pull from.

```bash
typings i npm~lodash --save
```

I suggest always searching typings before trying to install anything.


## Overview

So while these changes can seem confusing or drastic, they're really not that bad. Typings now seems much easier to approach and you understand where you're getting your types from.
