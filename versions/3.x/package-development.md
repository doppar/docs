---
title: Package Development
description: Doppar Package Development page
meta:
  - name: keywords
    content: Package Development
---

## Doppar Framework – Package Development Guide
### Introduction
Welcome to the official Doppar Framework Package Development Guide! This document walks you through the full lifecycle of creating, structuring, and maintaining packages within the Doppar PHP ecosystem.

In Doppar, packages are designed to promote modularity, reusability, and maintainability within applications. By encapsulating functionality into discrete, self-contained components, developers can isolate concerns and build features that are easy to test, share, and maintain.

A Doppar package can include everything necessary for a feature to operate independently—such as configuration settings, route definitions, views, migrations, and service classes. This architecture mirrors the core framework's own modular structure, making packages first-class citizens within your application.

Packages are typically integrated into a Doppar application through Service Providers, which act as the entry point for registration and bootstrapping. A service provider handles tasks like binding services to the container, publishing resources, and registering event listeners or middleware.

This separation of concerns enables you to:
- Develop and maintain reusable functionality across multiple projects.
- Share packages with the community or team via private or public repositories.
- Test features in isolation without coupling them to the core application logic.
- Cleanly structure large-scale applications using domain-driven or modular approaches.

Whether you're creating a logging utility, an admin panel, or a payment gateway integration, Doppar's package system empowers you to build with flexibility and scalability in mind.

In Doppar, a package is a self-contained module that can include:
- Configuration files
- Routes
- Views
- Migrations
- Service classes

## Creating a New Package
Start by creating a directory structure similar to this:
```markdown
packages/
└── VendorName/
    └── PackageName/
        ├── config/
        │   └── flarion.php
        ├── routes/
        │   └── web.php
        ├── views/
        │   └── (Odo files)
        ├── database/
        │   └── migrations/
        ├── src/
        │   └── PackageServiceProvider.php
        └── composer.json
```

Keep all business logic under a `src/` folder for cleaner autoloading and separation of concerns.

## Building the Service Provider
The Service Provider is the core class responsible for registering and bootstrapping your package’s functionality into a Doppar application. It tells Doppar how to load your package, and it acts as the central point to hook into the application's lifecycle.

Here's a breakdown of what goes into a typical service provider and its responsibilities:
```php
<?php

namespace Vendor\PackageName;

use Phaseolies\Providers\ServiceProvider;

class PackageServiceProvider extends ServiceProvider
{
    /**
     * Register services and bindings into the container.
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     */
    public function boot()
    {
        //
    }
}
```
## Register Service Provider
To integrate your package into a Doppar application, you need to register your package's service provider. The service provider is the entry point that tells Doppar how to load your package's components such as routes, views, config, and migrations.

Open your application's configuration file `config/app.php` and add your service provider with the `providers` array
```php
'providers' => [
    // Other service providers...
    Vendor\PackageName\PackageServiceProvider::class,
],
```
Replace `Vendor\PackageName\PackageServiceProvider` with the actual namespace and class name of your service provider.

## Configuration Files
Packages in Doppar can include their own configuration files to allow developers to customize behavior without modifying package code. These configuration files should be placed inside a config directory within your package `(e.g., config/flarion.php)`.

To make your package's configuration usable and customizable, follow these two steps:
## Merge Default Configuration
Inside your service provider’s `register()` method, use the `mergeConfig()` method to combine the package's default config with the user's application config. This ensures that default values are available while allowing overrides.
```php
public function register()
{
    $this->mergeConfig(__DIR__.'/config/flarion.php', 'flarion');
}

```
To make the config file publishable, add this to the boot() method:
```php
public function boot()
{
    $this->publishes([
        __DIR__ . '/config/flarion.php' => config_path('flarion.php'),
    ], 'config');
}
```
To publish the config file, run:
```bash
php pool vendor:publish --tag=config
```
After publishing, users can customize settings in config/flarion.php without modifying the original package files.

Now you can echo your package config name like
```php
echo config('flarion.key', 'default')
```

## Routing
When developing a package in the Doppar framework, routing is an essential part if your package needs to expose web-accessible endpoints. To maintain modularity and clarity, package routes should be placed in a dedicated routes directory—commonly `routes/web.php` for web routes.
```php
public function boot()
{
    $this->loadRoutes(__DIR__ . '/routes/web.php');
}
```
This ensures that when your package is registered, its routes are automatically included and accessible.

## Views
When developing a package in Doppar, it's common to include Odo templates that can be used by the host application. To do this effectively, Doppar allows you to register view paths with a custom namespace, enabling clean, modular usage.

## Loading Package Views
You should store your views inside a views/ directory within your package structure. Then, register them using the loadViews() method inside your service provider’s boot() method:
```php
public function boot()
{
    $this->loadViews(__DIR__ . '/views', 'flarion'); 
    // 'flarion' is the namespace
}
```
With this setup, you can reference your views using the namespace:
```php
return view('flarion::welcome');
```

## Publishing Views
To allow users to override or customize the package’s views, you can make them publishable. Use the publishes() method:
```php
public function boot()
{
    $this->publishes([
        __DIR__ . '/views' => resource_path('views/vendor/flarion'),
    ], 'views');
}
```

This command enables the developer to publish the views to their app's `resources/views/vendor/flarion` directory using:
```bash
php pool vendor:publish --tag=views
```
This setup supports clean view management, namespace isolation, and easy customization for end users.

## Migrations
If your package includes database structure changes (such as tables, columns, or indexes), you can ship migrations directly with it. Doppar supports automatic and publishable migrations to keep both package authors and users flexible and in control.

## Loading Migrations
You should place all migration files in a `database/migrations/` directory within your package. Then register them in your service provider:
```php
public function boot()
{
    $this->loadMigrations(__DIR__ . '/database/migrations');
}
```
This ensures that the migrations are automatically registered and can be executed when the application runs the standard migration command:


## Publishing Migrations
To allow developers to customize database migration files provided by your package, you can make them publishable. This is useful when your package includes default migrations, but the consuming application may want to modify or extend them.

Add the following to your service provider's `boot()` method:
```php
public function boot()
{
   $this->publishes([
        __DIR__ . '/database/migrations' => database_path('migrations'),
    ], 'migrations');
}
```

To publish the migration files to the main application, run:
```bash
php pool vendor:publish --tag=migrations
```

Now you can migrate your database like
```bash
php pool migrate
```

## Publishing Language Files
If you would like to publish your package's language files to the application's `lang/vendor` directory, you may use the service provider's publishes method. The publishes method accepts an array of package paths and their desired publish locations. For example, to publish the language files for the `flarion` package, you may do the following:
```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->loadTranslations(__DIR__.'/lang', 'flarion');

    $this->publishes([
        __DIR__.'/lang' => $this->app->langPath('vendor/flarion'),
    ], 'translations');
}
```
To publish the language files to the main application `lang` directory, run:
```bash
php pool vendor:publish --tag=translations
```

## Public Assets
Your package may have assets such as JavaScript, CSS, and images. To publish these assets to the application's public directory, use the service provider's publishes method. In this example, we will also add a public asset group tag, which may be used to easily publish groups of related assets:
```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    $this->publishes([
        __DIR__.'/public' => public_path('vendor/flarion'),
    ], 'public');
}
```
Now, when your package's users execute the `vendor:publish` command, your assets will be copied to the specified publish location. Since users will typically need to overwrite the assets every time the package is updated, you may use the --force flag:
```bash
php pool vendor:publish --tag=public
```

## Register Commands
Your package may include custom pool commands. To register them, you can use the commands method in your service provider's boot method. Doppar provides several ways to do this:
```php
/**
 * Bootstrap any package services.
 */
public function boot(): void
{
    // Method 1: Register multiple commands using an array (recommended)
    $this->commands([
        TestCommand::class,
        InstallCommand::class,
        PublishCommand::class,
    ]);

    // Method 2: Register multiple commands as arguments
    $this->commands(
        TestCommand::class,
        InstallCommand::class
    );

    // Method 3: Register a single command
    $this->commands(TestCommand::class);
}
```
Once your commands are registered, they will be available through pool. For example:
```bash
php pool your_command_to_run
```

This makes it easy for users of your package to interact with your functionality directly from the command line.

## Publish All Resources
The php pool `vendor:publish` command is used to publish assets, configuration files, views, migrations, translations, and other resources from a package to your application's root directory, making them customizable.

When you run the following command:
```php
php pool vendor:publish --provider="Vendor\PackageName\PackageServiceProvider"
```
> This command becomes available after your package is installed via Composer, such as when it's published on Packagist or required directly.

This command will publish any resources (like views, configurations, migrations, etc.) defined within the `PackageServiceProvider` to the appropriate locations within your application (like the `config`, `resources/views`, or `database/migrations` directories).