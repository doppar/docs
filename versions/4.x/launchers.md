---
title: Launchers
description: Doppar Launchers
meta:
  - name: keywords
    content: launchers, ghostable launchers, container, vendor publish, doppar
---

## Launchers

### Introduction

In Doppar, a launcher is the place where a feature joins the framework.

That feature might be small, like one container binding, or large, like a package that brings routes, migrations, translations, console commands, and publishable assets. Instead of scattering setup across random files, Doppar keeps that integration work inside launcher classes.

This makes launchers the framework's assembly layer. They tell Doppar:

- what should be bound into the container
- what should run only after the framework is ready
- what package resources should be loaded
- what assets should be publishable
- what services can be ghost-loaded only when needed

## The Doppar launcher Model

### Register First, Launch After

Every launcher has two core methods: `register()` and `launch()`.

Use `register()` for container bindings and raw service wiring. Use `launch()` for work that depends on other services already being available.

This split matters in Doppar because launchers can now be loaded eagerly or ghost-loaded later. Keeping `register()` focused on service registration and `launch()` focused on follow-up setup keeps that flow predictable.

```php
<?php

namespace App\Launchers;

use Phaseolies\Launchers\ServiceLauncher;

class BillingServiceLauncher extends ServiceLauncher
{
    public function register(): void
    {
        //
    }

    public function launch(): void
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

### What Belongs in `launch()`

`launch()` runs after launcher registration, so this is the right place for integration work that uses other registered services.

Typical work here:

- loading routes
- loading migrations
- loading views or translations
- registering console commands
- defining publishable files

Example:

```php
public function launch(): void
{
    $this->loadRoutes(__DIR__ . '/routes/web.php');
    $this->loadViews(__DIR__ . '/templates/views', 'reports');
    $this->commands([
        SyncReportsCommand::class,
    ]);
}
```

## Creating a launcher

### Generate a New launcher

Use Doppar's generator when you want a clean launcher scaffold inside `src/launchers`.

```bash
php pool make:launcher BillingLauncher
```

This creates a class that extends `Phaseolies\Launchers\ServiceLauncher` and includes empty `register()` and `launch()` methods.

## Registering launchers

### Add Your launcher to `runtime/config/app.php`

Application and package launchers are registered through the `launchers` array in `runtime/config/app.php`.

```php
'launchers' => [
    \App\Launchers\BillingLauncher::class,
],
```

Once listed there, Doppar will include the launcher during application startup.

## Working With the Container

Each launcher receives the application instance through `$this->app`. That gives you direct access to the container and framework services.

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

## Ghostable launchers

### Lazy-Loading launcher Services

Doppar follows a launcher pattern that is more explicit than simply "ghost" a package: ghostable launchers.

When a launcher implements `GhostableLauncher`, Doppar can queue that launcher during web startup and wait to fully register it until one of its declared services is actually requested.

This is useful for features that are not needed on every request, such as queue workers, optional package services, or heavier integrations.

```php
<?php

namespace App\Launchers;

use App\Services\ReportCache;
use Phaseolies\launchers\GhostableLauncher;
use Phaseolies\launchers\ServiceLauncher;

class ReportServicelauncher extends Servicelauncher implements GhostableLauncher
{
    public function register(): void
    {
        $this->app->singleton(ReportCache::class, ReportCache::class);
    }

    public function launch(): void
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

When `ReportCache::class` is resolved, Doppar loads this launcher on demand.

### How Ghostable Loading Behaves

The lifecycle is:

1. Doppar sees a launcher that implements `Ghostablelauncher`
2. on web requests, it queues the launcher instead of registering it immediately
3. when one of the service identifiers from `ghosts()` is resolved, the launcher is registered
4. if launcher launching has already started, `launch()` runs at that point too
5. in console mode, the launcher stays on the normal eager-loading path

That last rule is important. Console flows such as migrations, publishing, and command registration should still behave like normal launchers.

### Best Practices for Ghostable launchers

Ghostable launchers work best when you keep them narrow and predictable.

- return only real trigger services from `ghosts()`
- keep the launcher constructor side-effect free
- put bindings in `register()`
- keep follow-up integration in `launch()`
- avoid doing expensive work until the bound service itself is used

### Keep launchers Focused

In Doppar, launchers are not just a framework formality. They are the main contract for plugging features into the application lifecycle.

Use them to:

- register services cleanly
- load package resources in one place
- expose publishable assets
- register console commands
- ghost-load optional services when performance matters

If you keep each launcher focused on one feature boundary, the rest of the application stays easier to reason about.
