# laravel-mix-inertia-package-loader
Bring laravel package view loading to inertia


## Idea
Laravel comes with a view loading mechanism for packages, which looks into multiple places.
This becomes handy when you develop packages where you can define which views are publishable.
That gives the consumer an easy way to customize views for a package.
Because InertiaJS uses JS as a rendering engine the laravel build in functionality didnt work.

## Goal
We want to build the same mechanism laravel provide for views:

when we load `public.welcome`, we look for `resources/views/public/welcome.inertia.vue`
when we load `courier::public.welcome`, we look in two different places:

 - `resources/views/vendor/courier/public/welcome.inertia.vue`
 - `vendor/organization/package/resource/views/public/welcome.inertia.vue`
 
When we want to ship our own components, we register a package `@courier` for example:

Given we have a Layout Component:
- `vendor/organization/package/resource/js/Layout.vue`

we will reference this in our `vue files` as follows:
```js
import { Layout } from '@courier`;
```

When we make this components publishable, we copy this files into:
  - `resources/js/vendor/courier/Layout.vue`

and use `webpack` to make an alias to overwrite the package.

## Dependencies
We have to introduce a place to gather information to map the `composer package name` to its`alias`, for example `courier`.
The alias is used to publish files to the application, example from the laravel docs:
```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
    ]);
}
```
Laravel uses the `extra` field in the `composer.json` file for auto discovery.
We make use of the same pattern an introduce the `extra field` to the `package.json` file, for example:
```js
"extra": {
    "inertia": {
      "namespace": "courier",
      "vendor": "organization/package"
    }
  },
```

To archieve the component loading, we rely on `yarn workspaces` (https://yarnpkg.com/features/workspaces) and interpret our app as an monorepository.
```js
//package.json
{
    "private": true,
    "workspaces": [
        "vendor/organization/package"
    ],
}
```

## Installation

To install the `laravel-mix-extension`, run:
```
yarn add https://github.com/thoresuenert/laravel-mix-inertia-package-loader.git --save
```

Add the following to the `webpack.mix.js`:

```js
require('laravel-mix-inertia-package-loader');

mix.registerInertiaPackages('resources/views');
```


## Package Development
