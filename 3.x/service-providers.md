---
title: Service Providers
description: Doppar Service Providers page
meta:
  - name: keywords
    content: Service Providers
---

## Service Providers
### Introduction
In Doppar, service providers are the backbone of application bootstrapping—every core system and feature starts here. Whether it’s your application logic or Doppar’s native services, providers are the unified place for setup, registration, and configuration.

But what exactly does "bootstrapping" mean in Doppar?

Simply put, it’s the process of preparing your app for execution: binding services into the container, loading config values, registering session, routing logic, facades, and more—all from a central, predictable place.

Unlike traditional frameworks where bootstrapping logic is scattered, Doppar’s service provider system is designed for clarity, performance, and control.

💡 Why It’s More Powerful in Doppar
- Minimal overhead: No bloated lifecycle or unnecessary resolution.
- Pure core binding: Everything is tightly integrated with Doppar's high-performance core.
- Smarter autoloading: Providers are lazy-loaded only when needed.
- Unified structure: Middleware, routes, events, config, and facades can all be registered in one place.

You can add your own custom service provider in `config/app.php` under the providers array. Doppar scans and loads them efficiently—giving you complete control over your application lifecycle

## Creating Service Providers
Doppar provides `make:provider` pool command to create a new service provider.
```bash
php pool make:provider MyOwnServiceProvider
```

### The Register Method
As mentioned previously, within the register method, you should only bind things into the service container. You should never attempt to register other services. Otherwise, you may accidentally use a service that is provided by a service provider which has not loaded yet.

Let's take a look at a basic service provider. Within any of your service provider methods, you always have access to the `$app` property which provides access to the service container:
```php
<?php

namespace App\Providers;

use Phaseolies\Application;
use Phaseolies\Providers\ServiceProvider;

class MyOwnServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     */
    public function register(): void
    {
        //
    }
}
```

### The Boot Method
This boot method is called after all other service providers have been registered, meaning you have access to all other services that have been registered by the framework:
```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;

class MyOwnServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     */
    public function boot(): void
    {
        //
    }
}
```