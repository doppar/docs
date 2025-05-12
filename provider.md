## Service Providers
### Introduction
Service providers are the central place of all Doppar application bootstrapping. Your own application, as well as all of Doppar's core services, are bootstrapped via service providers.

But, what do we mean by "bootstrapped"? In general, we mean registering things, including registering service container bindings, configuration initialization, middleware, and even routes and all the facades. Service providers are the central place to configure your application.

Doppar uses service providers internally to bootstrap its core services, such as the mailer, cache, facades and others.

All user-defined service providers are registered in the `config/app.php` file. In the following documentation, you will learn how to write your own service providers and register them with your Doppar application.

### Creating Service Providers
Doppar provides `make:provider` pool command to create a new service provider.
```bash
php pool make:provider MyOwnServiceProvider
```

### The Register Method
As mentioned previously, within the register method, you should only bind things into the service container. You should never attempt to register other services. Otherwise, you may accidentally use a service that is provided by a service provider which has not loaded yet.

Let's take a look at a basic service provider. Within any of your service provider methods, you always have access to the $app property which provides access to the service container:
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
        $this->app->singleton(Connection::class, function (Application $app) {
            return new Connection('test');
        });
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
        // Register services
    }
}
```