---
title: Service Providers
description: Doppar service provider guide
meta:
  - name: keywords
    content: service providers, ghostable providers, container, vendor publish, doppar
---

## Service Providers

### Introduction

In Doppar, a service provider is the place where a feature joins the
framework.

That feature might be small, like one container binding, or large, like a package that brings routes, migrations, translations, console commands, and publishable assets. Instead of scattering setup across random files, Doppar keeps that integration work inside provider classes.

This makes providers the framework's assembly layer. They tell Doppar:

- what should be bound into the container
- what should run only after the framework is ready
- what package resources should be loaded
- what assets should be publishable
- what services can be ghost-loaded only when needed

## The Doppar Provider Model

### Register First, Boot After

Every provider has two core methods: `register()` and `boot()`.

Use `register()` for container bindings and raw service wiring. Use `boot()` for work that depends on other services already being available.

This split matters in Doppar because providers can now be loaded eagerly or ghost-loaded later. Keeping `register()` focused on service registration and `boot()` focused on follow-up setup keeps that flow predictable.

```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;

class BillingServiceProvider extends ServiceProvider
{
    public function register(): void
    {
        //
    }

    public function boot(): void
    {
        //
    }
}
```

### What Belongs in `register()`

`register()` is for declaring what your feature provides to the application.

Typical work here:

- binding classes into the container
- registering singletons
- merging package configuration
- preparing lightweight service definitions

Example:

```php
public function register(): void
{
    $this->app->singleton('reports.generator', ReportGenerator::class);

    $this->mergeConfig(__DIR__ . '/config/reports.php', 'reports');
}
```

### What Belongs in `boot()`

`boot()` runs after provider registration, so this is the right place for integration work that uses other registered services.

Typical work here:

- loading routes
- loading migrations
- loading views or translations
- registering console commands
- defining publishable files

Example:

```php
public function boot(): void
{
    $this->loadRoutes(__DIR__ . '/routes/web.php');
    $this->loadViews(__DIR__ . '/resources/views', 'reports');
    $this->commands([
        SyncReportsCommand::class,
    ]);
}
```

## Creating a Provider

### Generate a New Provider

Use Doppar's generator when you want a clean provider scaffold inside `app/Providers`.

```bash
php pool make:provider BillingServiceProvider
```

This creates a class that extends `Phaseolies\Providers\ServiceProvider` and includes empty `register()` and `boot()` methods.

## Registering Providers

### Add Your Provider to `config/app.php`

Application and package providers are registered through the `providers` array in `config/app.php`.

```php
'providers' => [
    App\Providers\BillingServiceProvider::class,
],
```

Once listed there, Doppar will include the provider during application startup.

## Working With the Container

Each provider receives the application instance through `$this->app`. That gives you direct access to the container and framework services.

```php
public function register(): void
{
    $this->app->singleton('reports.generator', function ($app) {
        return new ReportGenerator(
            cache: $app->make('cache'),
            logger: $app->make('log')
        );
    });
}
```

You can bind by string key, class name, or interface name depending on how you want the service to be resolved later.

## Ghostable Providers

### Lazy-Loading Provider Services

Doppar follows a provider pattern that is more explicit than simply "ghost" a package: ghostable providers.

When a provider implements `GhostableProvider`, Doppar can queue that provider during web startup and wait to fully register it until one of its declared services is actually requested.

This is useful for features that are not needed on every request, such as queue workers, optional package services, or heavier integrations.

```php
<?php

namespace App\Providers;

use App\Services\ReportCache;
use Phaseolies\Providers\GhostableProvider;
use Phaseolies\Providers\ServiceProvider;

class ReportServiceProvider extends ServiceProvider implements GhostableProvider
{
    public function register(): void
    {
        $this->app->singleton(ReportCache::class, ReportCache::class);
    }

    public function boot(): void
    {
        //
    }

    public function ghosts(): array
    {
        return [
            ReportCache::class,
        ];
    }
}
```

When `ReportCache::class` is resolved, Doppar loads this provider on demand.

### How Ghostable Loading Behaves

The lifecycle is:

1. Doppar sees a provider that implements `GhostableProvider`
2. on web requests, it queues the provider instead of registering it immediately
3. when one of the service identifiers from `ghosts()` is resolved, the provider is registered
4. if provider booting has already started, `boot()` runs at that point too
5. in console mode, the provider stays on the normal eager-loading path

That last rule is important. Console flows such as migrations, publishing, and command registration should still behave like normal providers.

### Best Practices for Ghostable Providers

Ghostable providers work best when you keep them narrow and predictable.

- return only real trigger services from `ghosts()`
- keep the provider constructor side-effect free
- put bindings in `register()`
- keep follow-up integration in `boot()`
- avoid doing expensive work until the bound service itself is used

### Keep Providers Focused

In Doppar, providers are not just a framework formality. They are the main contract for plugging features into the application lifecycle.

Use them to:

- register services cleanly
- load package resources in one place
- expose publishable assets
- register console commands
- ghost-load optional services when performance matters

If you keep each provider focused on one feature boundary, the rest of the application stays easier to reason about.
