---
title: "A Moduleless Future for Angular"
date: 2024-01-10T13:51:49.275Z
thumbnail: ""
draft: false
---

Recently a [Pull Request](https://github.com/angular/angular/pull/53861) from Matthieu Riegler came to the Angular repo that wanted to mark the `HTTPClientModule` and similar modules as deprecated. Overall it seems the idea has receieved positive traction, though in Matthieu's tweet about the PR, there was a comment in there that did stand out to me.

> I think I need an example before I decide whether I need pitchforks or not.

So let's make a quick example to show this person what this PR does and how it can impact non-standalone apps.



In an Angular app that uses `NgModules`, if you want to add `HttpClientModule` to the app, you need to go to the module that requires HTTP, and then add that to the `imports` array of the `NgModule`:



```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { HttpClientModule } from '@angular/common/http';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, BrowserAnimationsModule],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

With Standalone apps, this has moved to the simpler method of just providing the `provideHttpClient` in the main bootstrapping function

```ts
import { bootstrapApplication } from '@angular/platform-browser';
import { provideHttpClient } from '@angular/common/http';

import { AppComponent } from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    provideHttpClient(),
  ],
});
```

Why does this change matter? Well under the hood, `HttpClientModule` was doing the same exact thing as `provideHttpClient`. If we reference a very old version of `HTTPClientModule`, you can see how this works.


```ts
@NgModule({
  imports: [
    HttpClientXsrfModule.withOptions({
      cookieName: 'XSRF-TOKEN',
      headerName: 'X-XSRF-TOKEN',
    }),
  ],
  providers: [
    HttpClient,
    {provide: HttpHandler, useClass: HttpInterceptingHandler},
    HttpXhrBackend,
    {provide: HttpBackend, useExisting: HttpXhrBackend},
    BrowserXhr,
    {provide: XhrFactory, useExisting: BrowserXhr},
  ],
})
export class HttpClientModule {}
```

So when you add the `HttpClientModule` in your app, you're just having the module setup and provide the rest of your providers. Whta does `provideHttpClient` do? 

```ts
export function provideHttpClient(...features: HttpFeature<HttpFeatureKind>[]): EnvironmentProviders {
  const providers: Provider[] = [
    HttpClient,
    HttpXhrBackend,
    HttpInterceptorHandler,
    {provide: HttpHandler, useExisting: HttpInterceptorHandler},
    {provide: HttpBackend, useExisting: HttpXhrBackend},
    {
      provide: HTTP_INTERCEPTOR_FNS,
      useValue: xsrfInterceptorFn,
      multi: true,
    },
    {provide: XSRF_ENABLED, useValue: true},
    {provide: HttpXsrfTokenExtractor, useClass: HttpXsrfCookieExtractor},
  ];

  for (const feature of features) {
    providers.push(...feature.ɵproviders);
  }

  return makeEnvironmentProviders(providers);
}
```

The exact same thing! It's very nice to see how the two map back to one another. 



So this begs the question, what do you do if you still are using `NgModules` and need to use `HttpClient`? Well first off, don't panic. The PR isn't merged yet and it's only a deprecation notice. There's still time.


But since `provideHttpClient` and `HttpClientModule` fundamentally did the same thing, migration could be as simple as replacing the module with the `provideHttpClient` code.


```ts
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { provideHttpClient } from '@angular/common/http'

import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, ],
  providers: [ provideHttpClient() ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Then in your components, you'd be good to go!

```ts
export class HomePage {
  constructor(private http: HttpClient) {}
  public user: any = null;

  ngAfterViewInit() {
    this.http
      .get('https://randomuser.me/api/')
      .pipe(map((res: any) => res.results))
      .subscribe((data) => (this.user = data))
  }
}
```

To me, this feels like the best of both worlds, and helps people as they continue their migration to a standalone app setup.
