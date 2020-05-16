---
title: 'TypeScript ESLint Setup'
date: 2019-02-21T21:10:20-05:00
---

I was digging into ESLint tonight to see if there was a Language Service plugin available for TypeScript. In doing so, I realized I had no clue how to setup ESLint, especially considering the changes with regard to `@typescript-eslint`.

Last time I used ESLint, I was still writing AngularJS and ES6 was still far away. So, somethings have changed.

## Installing

First, we'll want to install the necessary packages, ESLint, the parser, and the plugin itself.

```shell
npm i eslint -D
npm i @typescript-eslint/parser -D
npm i @typescript-eslint/eslint-plugin -D
```

As noted on the TypeScript/ESLint plugin repo, if you have ESLint installed globally, you'll need to install the plugin globally as well.

## Set Up

With things installed, we need to setup an RC file for ESLint to read. This can be generated with

```shell
npx eslint --init
```

You will be prompted with different question about what you want to do with ESLint, most of which you can ignore for now. The last question will ask you how do you want to store your ESLint settings, a JavaScript file, JSON, or YAML. This is up to you, but for the rest of this post, I'll assume JavaScript.

With our config file created, we'll want to edit it and remove the unnecessary settings.

We should have something like this

```js
module.exports = {};
```

At a bare minimum, we need to include the correct parser and plugin for ESLint to work.

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  plugins: ['@typescript-eslint']
};
```

This will let us lint any TypeScript file, but it won't be really helpful. By default, ESLint doesn't read any project configuration from a `tsconfig.json`, you need to specify the path to your `tsconfig.json` in a `parserOptions` object.

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint']
};
```

Now if we run ESLint, it will understand how we have configured our project.

## Rules and Extends

Like TSLint, ESLint offers different rules and packages we can use to quickly enforce code style. The `@typescript-eslint` plugin ships with recommended rules that can be used.

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint'],
  extends: ['plugin:@typescript-eslint/recommended']
};
```

This is nice to get up and running, but what if you need to customize the rules?

To do this, we just add a `rules` entry, and configure the rules available

```js
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json'
  },
  plugins: ['@typescript-eslint'],
  extends: ['plugin:@typescript-eslint/recommended'],
  rules: {
    '@typescript-eslint/restrict-plus-operands': 'error'
  }
};
```

To see what rules are available, you can read [the docs](https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin#supported-rules).

### Closing

I'm excited to see where ESLint goes and start integrating it in my projects.
