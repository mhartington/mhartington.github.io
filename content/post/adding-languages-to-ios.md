---
title: "Adding Languages to iOS"
date: 2018-01-19T07:24:04-05:00
thumbnail: "/img/thumbnails/adding-languages-to-ios/i8n-v2.png"
draft: false
---

In hybrid apps, when we want to add support for different languages, we tend to rely on JavaScript libraries to make this possible. This is perfect when we need to deploy a PWA, but when it comes to the App Store, we then have a different challenge. How do we tell the App Store that we support different languages as well?


### How iOS Checks this

For native iOS project, the process is actually quite similar to how we manage localization in web-based apps. We setup a string file that is for the languages we want to support and create a mapping of keywords and their translated version. Then the translated value can be accessed in the app via `NSLocalizedString`. But for hybrid apps that do not use native code, what do we do?


Well turns out, we can trick Xcode/iOS App store by declaring supported languages in a plist file.


![Xcode screen shot showing supported languages](/img/xcode-multiple-languages.png)

In this plist file, we have an entry called `Localizations`, which is an array. With this entry, we can add list out the languages our app supports.

![screen shot with English and Italian](/img/english-and-italian.png)

Perfect! Now we can just add the languages we want to support in there and be off to the races! But we can take this even further and make the whole process simpler.


### Plugins FTW

For some people, opening up xcode is not ideal, and I am one of those people. Plus, once we add the language support in XCode, we need to commit the entire project in git, which isn't ideal. Instead, we can rely on a plugin to do the work for us. Enter the [localized-strings](https://github.com/enricodeleo/cordova-plugin-ios-localized-strings) plugin.

This plugin, has zero native code, no JS interface, and is not loaded at runtime (how sweet is that), but instead is a simple hook that runs during `cordova build`. To install this plugin you can run:


```bash
cordova plugin add cordova-plugin-ios-localized-strings --variable MAIN_LANGUAGE=English --variable ADDITIONAL_LANGUAGES=en-US,es,it
```

Now our two variables are:

`MAIN_LANGUAGE`: The projects main language
`ADDITIONAL_LANGUAGES`: The locales we want to support


Once this plugin is installed and we build iOS, it will take care of all that manual work we would have had to do before.

![screen shot of 2 iphones localized](/img/ipones-localized.png)

