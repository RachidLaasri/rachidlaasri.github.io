---
layout: post
title:  "Laravel class-based Macros"
date:   2018-04-28 14:01:00
categories: ["php", "laravel", "macro"]
comments_enabled: true
---
Laravel Macros are a clean way to add pieces of functionality to classes you don’t own (core Laravel components) and re-use them in your projects. It was first introduced in the 4.2 version but it was only recently that I discovered  the ability to define class-based macros. So, this is what this article is going to be about.


Macros can be defined on any class that makes use of the [Macroable trait](https://github.com/laravel/framework/blob/5.6/src/Illuminate/Support/Traits/Macroable.php) by simply calling the `macro` method providing the required arguments:

```PHP
Response::macro('macroName', function ($value) {
    // code
});
```

## Quick examples
In these two examples, we will be be looking at how we can add two macros `at` and `toPairs` to Laravel Collection class.

- “at” macro will help us retrieve an item at any given index:

```PHP
Collection::macro('at', function ($index) {
    return $this->slice($index, 1)->first();
});

# Usage
$collection = collect([1, 2, 3]);
$collection->at(0); // 1
```

- “toPairs” macro is used to transform a collection into an array with pairs:

```PHP
Collection::macro('toPairs', function () {
    return $this->keys()->map(function ($key) {
        return [$key, $this->items[$key]];
    });
});

# Usage
$collection = collect(['a' => 'b', 'c' => 'd', 'e' => 'f'])->toPairs();
$collection->toArray(); // returns ['a', 'b'], ['c', 'd'], ['e', 'f']
```


Note: if you are interested in implementing those macros in your app, consider checking [this package](https://github.com/spatie/laravel-collection-macros).

## Creating class-based macros
Generally, the default `AppServiceProvider` is a perfectly fine place for defining your macros, but it can quickly become bloated. A better way is to put them within their own classes especially if they are related.
Let’s take a look at how we can do that.

1. Inside your `AppServiceProvider` boot method:

```PHP
Collection::mixin(new CollectionMacros);
```

2. Create the `CollectionMacros` class:

```PHP
class CollectionMacros
{
    /**
     * Retrieve an item at any given index.
     *
     * @return \Closure
     */
    public function at()
    {
        return function ($index) {
            return $this->slice($index, 1)->first();
        };
    }

    /**
     * Transform a collection into an array with pairs.
     *
     * @return \Closure
     */
    public function toPairs()
    {
        return function () {
            return $this->keys()->map(function ($key) {
                return [$key, $this->items[$key]];
            });
        };
    }
}
```


Laravel will automatically scan the `CollectionMacros` class and makes all the *public* and *protected* methods available as macros. Using this technique, our `AppServiceProvider` is much cleaner now and our macros are grouped into one single class when they are related.

Happy refactoring!
