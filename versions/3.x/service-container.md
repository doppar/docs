---
title: Service Container
description: Doppar Service Container page
meta:
  - name: keywords
    content: Service Container
---

## Service Container
### Introduction
The Doppar Service Container is a robust and versatile tool designed to streamline dependency management and facilitate dependency injection within application. At its core, dependency injection is a sophisticated concept that simplifies how class dependencies are handled: instead of a class creating or managing its own dependencies, they are "injected" into the class—typically through the constructor or, in certain scenarios, via setter methods. This approach promotes cleaner, more modular, and testable code, making the Doppar framework an ideal choice for modern, scalable web development.
Background

Doppar supported dependency binding through several approaches:
- Parameter-Level binding with `#[Bind]` attribute
- Method-level binding using `#[Resolver]` attribute
- Class-level binding using `#[Resolver]` attribute
- Service provider based binding via `$this->app->bind()` method

The Doppar Service Container simplifies dependency management by providing a clean and intuitive API for binding and resolving services. Whether you're working with regular bindings, singletons, or conditional logic, the container ensures your application remains modular, testable, and scalable.

To register bindings into the container, simply use the `bind()` method provided through the `$this->app` object in a service provider.

Summary of supported binding methods
| Binding Type              | Scope              | Example Syntax                                                    | Singleton Support | Ideal Use Case                        |
| ------------------------- | ------------------ | ----------------------------------------------------------------- | ----------------- | ------------------------------------- |
| **Service Provider**      | Global             | `$this->app->bind(Interface::class, Class::class);`               | Yes             | App-wide, reusable bindings           |
| **Class-Level Resolver**  | Controller-wide    | `#[Resolver(Interface::class, Class::class)]`                     | Yes             | Consistent dependencies in one class  |
| **Method-Level Resolver** | Method-specific    | `#[Resolver(abstract: Interface::class, concrete: Class::class)]` | Yes             | One-off or varied bindings per method |
| **Parameter-Level Bind**  | Parameter-specific | `#[Bind(Class::class)] Interface $param`                    | Yes             | Contextual or fine-grained binding    |

Here’s a basic example of binding using service provider:
```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot(): void
    {
        $this->app->bind('abc', fn() => 'xyz');
    }
}
```

Once bound, you can retrieve the value anywhere in your application using the `app()` helper:
```php
$value = app('abc'); // Returns 'xyz'
```

This is the power of the Doppar container—elegant, minimal, and expressive service registration and resolution. This demonstrates how Doppar allows you to register lightweight bindings using closures, making your application modular and testable by default.

## Utilize the Container in Doppar
You can often type-hint dependencies in your routes, controllers, services, and elsewhere without ever manually interacting with Doppar's container.

For example, you might type-hint the `Phaseolies\Http\Request` object directly in your route handler to access the incoming HTTP request. Even though you never explicitly call the container, Doppar handles the injection of these dependencies behind the scenes:

```php
use Phaseolies\Http\Request;

Route::get('/', function (Request $request) {
    // Use the request object...
});
```

## Attribute-Based Binding
Starting with Doppar’s modern service container implementation, you can bind interfaces to their concrete implementations using attributes directly on controllers or methods. This attribute-based approach offers a clean, declarative way to configure your dependencies without manually binding them in a service provider.

## #[Bind] Attribute
The `#[Bind]` attribute provides a more expressive and localized way to bind interfaces or abstract classes to their concrete implementations directly within controller method parameters.

It enhances readability and reduces boilerplate by allowing you to define bindings exactly where they are used, rather than at the class or service-provider level.

With `#[Bind()]` attribute, you can perform parameter-level bindings, making dependency injection cleaner, context-specific, and easier to maintain. It reduces boilerplate code, improves readability, and makes controller actions more self-contained.

See the example
```php
#[Route(uri: 'user/store', methods: ['POST'])]
public function store(
    #[Bind(UserRepository::class)] UserRepositoryInterface $userRepository
) {
    // #[Bind] resolves UserRepositoryInterface to UserRepository.
}
```

This way, bindings are visible directly where dependencies are injected. The binding applies only to this method’s context, making it explicit and lightweight.

### Singleton Binding

You can also define a binding as a singleton by passing true as the second argument. This ensures the same instance is reused across the request lifecycle.
```php
#[Route(uri: 'api/post', methods: ['POST'])]
public function store(
    #[Bind(PostRepository::class, true)] PostRepositoryInterface $postRepository,
    Request $request
) {
    // #[Bind(..., true)] resolves PostRepositoryInterface to PostRepository as a singleton.
}
```
The second argument `(true)` marks the binding as singleton, meaning Doppar will reuse the same instance for all subsequent resolutions during the request.

### Bind Attribute in Contructor
In Doppar, you can automatically inject dependencies into your controllers or services using attribute-based binding you already know that. You can also specify which concrete class should be injected for a given interface, directly in the constructor by this following way.

```php
class PostController extends Controller
{
    public function __construct(
        #[Bind(PostRepository::class)] private PostRepositoryInterface $postRepository,
    ) {}

    #[Route('/')]
    public function __invoke(Request $request)
    {
        //
    }
}
```
When the framework instantiates the `PostController`, it looks for all constructor parameters marked with `#[Bind()]`. Doppar automatically creates (or fetches) an instance of the specified class and assigns it to the declared property.

### In summary:

`#[Bind(concrete: ClassName::class)]` binds an interface or abstract type to the specified concrete class. Adding a second argument `(true)` makes it a singleton binding for that request’s lifecycle.

### #[Resolver] Attribute
The `#[Resolver]` attribute allows you to bind an interface to its concrete implementation (optionally as a singleton) directly where it’s needed. Doppar will automatically resolve and inject the dependency into the constructor or method during the request lifecycle.

See the example of class-level binding
```php
<?php

namespace App\Http\Controllers\Admin;

use App\Repository\UserRepositoryInterface;
use App\Repository\UserRepository;
use Phaseolies\Utilities\Attributes\Resolver;
use App\Http\Controllers\Controller;
use Phaseolies\Http\Response;

#[Resolver(UserRepositoryInterface::class, UserRepository::class)]
class UserController extends Controller
{
    public function index(UserRepositoryInterface $user): Response 
    {
        return $user->getUsers();
    }
}
```

In this example, Doppar binds `UserRepositoryInterface` to `UserRepository` at the class level. This means all methods within `UserController` that require `UserRepositoryInterface` will automatically receive an instance of `UserRepository`.

See the example of method-level binding
```php
<?php

namespace App\Http\Controllers\Admin;

use Phaseolies\Utilities\Attributes\Resolver;
use App\Http\Controllers\Controller;
use Phaseolies\Http\Response;
use App\Repository\UserRepositoryInterface as IUser;
use App\Repository\UserRepository;

class UserController extends Controller
{
    #[Resolver(abstract: IUser::class, concrete: UserRepository::class)]
    public function index(IUser $userRepository): Response
    {
        return $userRepository->getUsers();
    }
}
```
This binds the dependency only for the specific method.

> You can pass `true` as the third argument to make it a singleton binding, ensuring it is resolved as a singleton instance

This attribute-driven design offers a modern, lightweight alternative to traditional container configuration and is particularly useful for modular, package-oriented, and clean architecture designs.

## Binding Using Service Provider
In Doppar, most of your service container bindings will be registered within service providers. These providers are the central location for binding services and classes into the container.

Within a service provider, you have access to the container through the `$this->app` property. This gives you the flexibility to bind interfaces or classes to their implementations easily. To register a binding, you can use the bind method. You pass the interface or class name that you want to bind, along with a closure that returns an instance of the concrete implementation.

You can simply bind your abstraction to its concrete implementation like this:
```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;
use App\Repository\UserRepositoryInterface;
use App\Repository\UserRepository;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot(): void
    {
        $this->app->bind(
            abstract: UserRepositoryInterface::class,
            concrete: fn() => new UserRepository()
        );

        // Or
        $this->app->bind(
            abstract: UserRepositoryInterface::class,
            concrete: UserRepository::class
        );
    }
}
```


Take a look some others example, if you want to bind a `NotificationService` to a specific payment provider implementation, you would write something like:

```php
use Phaseolies\Application;
use App\Services\NotificationService;
use App\Services\MailerService;

$this->app->bind(NotificationService::class, function (Application $app) {
    return new NotificationService($app->make(MailerService::class));
});
```
However, if you need to access the container outside of a service provider, you can easily do so using the App facade.
```php
use Phaseolies\Support\Facades\App;
use App\Services\NotificationService;

App::bind(NotificationService::class, function (Application $app) {
    // ...
});
```

Even you can use the app() helper like
```php
use Phaseolies\Application;
use App\Services\NotificationService;

app()->bind(NotificationService::class, function (Application $app) {
    // ...
});
```

### Binding A Singleton
The singleton method registers a class or interface with the container, ensuring it is instantiated only once. After the initial resolution, the same instance of the object is returned each time it is requested from the container.
```php
use Phaseolies\Application;
use App\Services\NotificationService;
use App\Services\MailerService;

$this->app->singleton(NotificationService::class, function (Application $app) {
    return new NotificationService($app->make(MailerService::class));
});
```

You can conditionally register a service within the container, applying the binding only if a specific condition is met. For example, you might choose to bind a singleton only under certain runtime circumstances:
```php
$this->app->when(fn() => config('cache.enabled'))
    ?->bind(UserRepositoryInterface::class, function ($app) {
        $EntityRepo = $app->make(EntityUserRepository::class);
        return new CachedUserRepository(
            $EntityRepo,
            $app->make('cache')
        );
    });
```

In this example, the NotificationService will be registered as a singleton only if the random condition returns `true`. This provides dynamic control over how and when services are introduced into the container.

### Binding Instances
You can bind an existing instance directly into the container using the instance method. This is useful when you have a pre-configured object that you want to share throughout your application:
```php
use App\Services\ApiClient;

$apiClient = new ApiClient([
    'base_url' => 'https://api.example.com',
    'timeout' => 30
]);

$this->app->instance(ApiClient::class, $apiClient);
```

Once bound, every resolution of `ApiClient` will return the exact same instance:
```php
$client1 = app(ApiClient::class);
$client2 = app(ApiClient::class);

// $client1 === $client2 (same instance)
```
The `share` method is an alias for instance:
```php
$this->app->share(ApiClient::class, $apiClient);
```

## Create Instance Using Container
Doppar provides a powerful global `app()` helper function that gives you access to the Dependency Injection Container. You can use it to create and resolve class instances with zero boilerplate.

### Automatic Resolution
Doppar supports automatic resolution of class dependencies. Just pass the class name to `app()`:
```php
// Creates an instance of SMSService
$object = app(SMSService::class);
$object->sendSms();
```
This will resolve and instantiate the class.

### The make Method
The `make` method is an alias for resolving classes from the container:
```php
$service = $this->app->make(SMSService::class);
```

### Resolving with Parameters
You can pass constructor parameters when resolving:
```php
$service = app(SMSService::class, ['apiKey' => 'your-key-here']);
```

### Binding and Resolving Custom Keys
You can bind any service or class instance to a custom key using the `bind()` method, then resolve it via `app()`:
```php
// Bind 'sms' to an instance of SMSService
$this->app->bind('sms', fn() => new SMSService());

// Retrieve and use the service
app('sms')->sendSms();
```

> If you call `app()` without any arguments, it returns the Application container instance itself. This allows for advanced usage like container inspection, rebinding, and more.

This is the power of Doppar's container: clean, expressive, and ready for modern PHP applications.

## Advanced Container Features
You can extend an existing binding to wrap or modify its resolution:
```php
$this->app->extend(UserRepositoryInterface::class, function ($repository, $app) {
    return new CachedUserRepository($repository, $app->make('cache'));
});
```

This is useful for decorating services or adding middleware-like functionality to existing bindings.

### Aliasing
Create an alias for a binding to make it accessible through multiple keys:
```php
$this->app->bind(NotificationService::class, fn() => new EmailNotificationService());
$this->app->alias(NotificationService::class, 'notifications');

// Both work
$service1 = app(NotificationService::class);
$service2 = app('notifications'); // Same instance
```

### Calling Methods with Dependency Injection
The `call` method allows you to invoke any callable with automatic dependency injection:
```php
$result = $this->app->call(function (Request $request, UserRepository $users) {
    return $users->find($request->input('id'));
});
```

This also works with class methods:
```php
$this->app->call([UserController::class, 'show'], ['id' => 1]);
```

### Resolving Method Dependencies
You can resolve dependencies for a specific method without calling it:
```php
$dependencies = $this->app->resolveMethodDependencies(
    UserController::class,
    'update',
    ['id' => 1]
);
```

### Checking Container State
Check if a binding exists
```php
if ($this->app->has(UserRepositoryInterface::class)) {
    // Binding exists
}
```

Check if an binding is resolved
```php
if ($this->app->hasInstance(CacheService::class)) {
    // Instance has been resolved
}
```

Check if a binding is a singleton
```php
if ($this->app->isSingleton(UserRepositoryInterface::class)) {
    // This binding is registered as a singleton
}
```

Check if currently resolving
```php
if ($this->app->isResolving(SomeService::class)) {
    // This service is currently being resolved
}
```

Check if resolved
```php
if ($this->app->resolved(UserRepositoryInterface::class)) {
    // This service has been resolved at least once
}
```

### Service Provider Registration
You can dynamically register service providers:
```php
$this->app->register(CacheServiceProvider::class);

// Or pass an instance
$provider = new CacheServiceProvider();
$this->app->register($provider);
```

The container will automatically call the provider's `register()` and `boot()` methods with dependency injection support.

### Container Instance
The container itself is a singleton. You can access it globally:
```php
$container = Container::getInstance();
```

Or set a custom instance:
```php
$customContainer = new Container();

Container::setInstance($customContainer);
```

Or forget the instance:
```php
Container::forgetInstance();
```
