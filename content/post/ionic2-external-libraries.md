---
date: "2016-02-17T11:38:32-05:00"
title: "Ionic 2 and External Libraries"
---


Now that Ionic 2 is out in beta, people are setting aside some time to give it a shot and investigate everything it has to offer. Ionic 2 and Angular 2 bring a lot of improvements, but it's a fairly different style of developing that what people were used to before. Now, since everything needs to be imported and libraries aren’t global, it can be tricky to figure out how to integrate with other libraries.


### NPM all the things

So, how do you add third-party-libraries, such as Lodash? In Ionic 22, we've moved over to NPM for all our package management. So for our case, if we had a project setup, we could just install Lodash though npm's CLI.

```bash
$ ionic start myApp --v2 --ts
$ cd myApp
$ npm install lodash --save
```

Now, this is going to give us a starter tabs project, so let’s open our `page1.ts` file.

```javascript
import {Page} from 'ionic-framework/ionic';
@Page({
  templateUrl: 'build/pages/page1/page1.html',
})
export class Page1 {
  constructor() {
  }
}
```

Now, we can import individual methods from Lodash, using the typical import method that we're seeing. Note, this would be the same import state if we were to use Javascript or Typescript.

```javascript
import {Page} from 'ionic-framework/ionic';
import {sum} from 'lodash'
@Page({
  templateUrl: 'build/pages/page1/page1.html',
})
export class Page1 {
  constructor() {
    console.log(sum)
  }
}
```

If we were to use regular ES6, that would be the end of it. But since we're using Typescript, we'll get an error.


 [![lodash-typescript-error](/img/lodash-typescript-error.png)](/img/lodash-typescript-error.png)

This might lead you to believe that everything is broken, and Lodash is working. Your editor will also probably yell at you, too. But oddly enough, everything will still work. What gives?

So, Typescript needs to analyze the code, in order to do its type checking. Normal Javascript libraries typically don't include any definition files, which means Typescript won't be able to understand them.

To circumvent this, we can use [Typings](https://www.npmjs.com/package/typings) which allow us to install definition files for various libraries.

### Typings

We'll install Typings globally on our system.

```bash
$ npm install -g typings
```

Now we have the `typings` command to install the necessary files. So we can install the Lodash definitions with

```
typings install lodash --ambient --save
```

We should now have a `typings` directory looking like this

```bash
├── typings
│   ├── browser
│   │   └── ambient
│   │       └── lodash
│   │           └── lodash.d.ts
│   ├── browser.d.ts
│   ├── main
│   │   └── ambient
│   │       └── lodash
│   │           └── lodash.d.ts
│   └── main.d.ts
├── typings.json
```

Last thing we need to do, is add a reference to this main `main.d.ts` file.

Now we could do this two ways, either adding a `reference` tag, which most people are probably familiar with, or we can add it to our `files` array in our tsconfig. We'll use this option because as our application grows, we don't have to worry about adding more reference tags all over the place.

```json

  "files": [
    "app/app.ts",
    "typings/main.d.ts"
  ],
```

Now when we start up our server, no errors!
