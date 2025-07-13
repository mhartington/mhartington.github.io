---
title: 'Recreating TanStack Query in Angular'
date: 2025-07-12T10:39:50-04:00
draft: false
---

Recently I've been building a React e-commerce app in a few frameworks for some Nx work, and in doing so I got to play around more with [TanStack Query](https://tanstack.com/query/latest). I'm very late to the party when it comes to using TanStack Query but it's damn good. While building an Angular version of my demos, I started missing some of TanStack Query's features and wanted to take a shot at recreating something for Angular that would have something similar in a simple service.

> There is an Angular version of TanStack Query and you should definitely use it! At the time, I was aware of the Angular version, but wanted to take a shot at building my own version and see how simple it could be.

## Starting simple

Let's start with a basic service that we can inject into our app and in it we have a `query` method that accepts a `key`, a `fetcher` function, and some `options`

```ts
@Injectable({ providedIn: 'root' })
export class QueryState {
  query(key, fetcher, options) {}
}
```

From here, we'll need a way to keep track of queries we'll be creating, so can just use a `Map` for this:

```ts
@Injectable({ providedIn: 'root' })
export class QueryState {
  private queries = new Map<string, IQuery<unknown>>();

  query(key, fetcher, options) {}
}
```

Now let's check first to see if we have the query saved already. This way we can just return it and any data that might have already been fetched or created:

```ts
query(key, fetcher, options){
  if (this.queries.has(key)) {
    return this.queries.get(key);
  }
}
```

We'll next create some variables that will get returned from this `query` method. 

```ts
query(key, fetcher, options){
  if (this.queries.has(key)) {
    return this.queries.get(key);
  }
    const data = signal(null);
    const isLoading = signal(false);
    const error = signal(null);
}
```

Now for the real work, we need a `load` variable that we can use to call our `fetcher` function:

```ts
const load = async () => {
  isLoading.set(true);
  error.set(null);
  try {
    const result = await fetcher();
    data.set(result);
  } catch (err) {
    error.set(err);
  } finally {
    isLoading.set(false);
  }
};
```

Cool, now we can run our fetcher and set the `data` to what ever return value we get from it. Now store all of these variables to a `state` variable and save it in our `queries` map from earlier

```ts
const state = {
  data,
  isLoading,
  error,
  refetch: load,
  invalidate: async () => {
    data.set(null);
    load();
  },
};

this.queries.set(key, state);
return state;
```

There's two addiontal features in here that we did not cover, `refecth` to rerun the `fetcher` and `invalidate` which we can use to clear the data in the query and refetch.

In addition to `invalidate`, let's add some methods to call invalidate on one or all of the queries we have.

```ts
  invalidateQuery(key: string): void {
    if (this.queries.has(key)) {
      const query = this.queries.get(key)!;
      query.invalidate();
    }
  }
  invalidateQueries(): void {
    this.queries.forEach((query) => {
      query.invalidate();
    });
  }
```

And that's it! Using this service comes down to something like this

```ts
export class ProductList {

  readonly queryState = inject(QueryState);
  data = this.queryState.query(
    `products`, 
    async () => getProducts(this.category()),
  );

}
```

All together the final version with types looks like so: 

```ts
import { Injectable, signal, Signal, effect, EffectRef } from '@angular/core';

interface IQuery<T> {
  data: Signal<T | null>;
  isLoading: Signal<boolean>;
  error: Signal<unknown>;
  refetch: () => void;
  invalidate: () => void;
}

interface QueryOptions {
  enabled?: Signal<unknown>;
}

@Injectable({ providedIn: 'root' })
export class QueryState {
  private queries = new Map<string, IQuery<unknown>>();
  private effects = new Map<string, EffectRef>();

  query<T>(
    key: string,
    fetcher: () => Promise<T>,
    options: QueryOptions = {}
  ): IQuery<T> {
    if (this.queries.has(key)) {
      return this.queries.get(key)! as IQuery<T>;
    }

    const data = signal<T | null>(null);
    const isLoading = signal<boolean>(false);
    const error = signal<unknown>(null);

    const load = async () => {
      isLoading.set(true);
      error.set(null);
      try {
        const result = await fetcher();
        data.set(result);
      } catch (err) {
        error.set(err);
      } finally {
        isLoading.set(false);
      }
    };

    // Only run the query if enabled is true (or not provided)
    if (options.enabled) {
      const effectRef = effect(() => {
        if (options.enabled!()) {
          load();
        }
      });
      this.effects.set(key, effectRef);
    } else {
      load();
    }

    const state: IQuery<T> = {
      data,
      isLoading,
      error,
      refetch: load,
      invalidate: async () => {
        data.set(null);
        load();
      },
    };

    this.queries.set(key, state);
    return state;
  }

  invalidateQuery(key: string): void {
    if (this.queries.has(key)) {
      const query = this.queries.get(key)!;
      query.invalidate();
    }
  }
  invalidateQueries(): void {
    this.queries.forEach((query) => {
      query.invalidate();
    });
  }
}
```


This was a fun experiment to work on. It probably has some room for improvment, but as an attempt to make a MVP, it works well. Let me know if you have any suggestions and improvments. 
