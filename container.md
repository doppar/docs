## Service Container
### Introduction
The Doppar Service Container is a robust and versatile tool designed to streamline dependency management and facilitate dependency injection within application. At its core, dependency injection is a sophisticated concept that simplifies how class dependencies are handled: instead of a class creating or managing its own dependencies, they are "injected" into the class—typically through the constructor or, in certain scenarios, via setter methods. This approach promotes cleaner, more modular, and testable code, making the Doppar framework an ideal choice for modern, scalable web development.

The Doppar Service Container simplifies dependency management by providing a clean and intuitive API for binding and resolving services. Whether you're working with regular bindings, singletons, or conditional logic, the container ensures your application remains modular, testable, and scalable.

Let's see a simple example
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use App\Service\PaymentServiceInterface as Payment;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function __construct(protected User $user) {}

    public function index(Payment $payment, Request $request)
    {
        $status = $payment->processPayment();
        $users = $this->user->all();
    }
}
```

In this example, the `App\Service\PaymentServiceInterface` from controller index method and `User` class from constructor will automatically injected in Doppar's request life cycle and will be automatically instantiated.

### When to Utilize the Container in Doppar
You can often type-hint dependencies in your routes, controllers, services, and elsewhere without ever manually interacting with Doppar's container.

For example, you might type-hint the `Phaseolies\Http\Request` object directly in your route handler to access the incoming HTTP request. Even though you never explicitly call the container, Doppar handles the injection of these dependencies behind the scenes:

```php
use Phaseolies\Http\Request;

Route::get('/', function (Request $request) {
    // Use the request object...
});
```

In many scenarios—thanks to automatic dependency injection and tools like facades or service classes—you can build an entire Doppar application without manually binding or resolving anything from the container.

First, if you write a class that implements an interface and you wish to type-hint that interface on a route or class constructor, you must tell the container how to resolve that interface. Secondly, if you are writing a Doppar package that you plan to share with other Doppar developers, you may need to bind your package's services into the container.

### Binding
In Doppar, most of your service container bindings will be registered within service providers. These providers are the central location for binding services and classes into the container.

Within a service provider, you have access to the container through the `$this->app` property. This gives you the flexibility to bind interfaces or classes to their implementations easily. To register a binding, you can use the bind method. You pass the interface or class name that you want to bind, along with a closure that returns an instance of the concrete implementation.

For example, if you want to bind a `NotificationService` to a specific payment provider implementation, you would write something like:
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
use Phaseolies\Application;
use App\Services\NotificationService;
use App\Services\MailerService;

$this->app->when(fn() => rand(0, 1) === 1)?->singleton(NotificationService::class, function (Application $app) {
    return new NotificationService($app->make(MailerService::class));
});
```

In this example, the NotificationService will be registered as a singleton only if the random condition returns `true`. This provides dynamic control over how and when services are introduced into the container.

### Create Instance Using Container
Doppar allows you to create object of a class using global `app()` function. If you want to create object that works like a singleton object, you can use the app() function
```php
$singletonObject = app(SMSService:class); // this is the object of SMSService class
$singletonObject->sendSms();
```

But if you call just `app()`, it will return the Application instance.