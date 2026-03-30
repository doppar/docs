---
title: Middleware
description: Doppar Middleware page
meta:
  - name: keywords
    content: Middleware
---

## Middleware
### Introduction
Middleware offers a powerful way to inspect, filter, and act on HTTP requests before they reach your application's core logic. For instance, Doppar includes built-in middleware that ensures a user is authenticated. If the user isn't authenticated, the middleware can redirect them to a login page; otherwise, it allows the request to proceed.

Beyond authentication, middleware can handle a wide range of tasks. For example, a logging middleware might record every incoming request, while another could handle input sanitization or API rate limiting. Doppar comes bundled with several useful middleware for tasks like authentication and CSRF protection. Custom middleware can be created and are typically stored in the `app/Http/Middleware` directory of your application.

## Create New Middleware
To create a new middleware, use the `make:middleware` Pool command:
```bash
php pool make:middleware IpBasedUserBlocked
```
This command will place a new `IpBasedUserBlocked` class within your `app/Http/Middleware` directory. In this middleware, we will only allow access to the route if the supplied ip input matches a specified ip. Otherwise, we will redirect the users back to the /home URI:
```php
<?php

namespace App\Http\Middleware;

use Phaseolies\Support\Facades\Auth;
use Phaseolies\Middleware\Contracts\Middleware;
use Phaseolies\Http\Response;
use Phaseolies\Http\Request;
use Closure;

class IpBasedUserBlocked implements Middleware
{
    /**
     * Handle an incoming request
     *
     * @param Request $request
     * @param \Closure(\Phaseolies\Http\Request) $next
     * @return Phaseolies\Http\Response
     */
    public function __invoke(Request $request, Closure $next): Response
    {
        if ($request->ip() === '127.0.0.1') {
            return $next($request);
        }

        return redirect('/home');
    }
}
```

As you can see, if the given `ip` does not match our defined `ip`, the middleware will return an HTTP redirect to the client; otherwise, the request will be passed further into the application.

> All middleware are resolved via the service container, so you may type-hint any dependencies you need within a middleware's constructor.

## Register Middleware
To register middleware as route specific, you need to update `App\Http\Kernel`.php file and update `$routeMiddleware` arrays like that.
```php
<?php

/**
 * The application's route middleware.
 *
 * These middleware may be assigned to groups or used individually.
 *
 * @var array<string, class-string|string>
 */
public array $routeMiddleware = [
    'web' => [
        'blocked' => \App\Http\Middleware\IpBasedUserBlocked::class,
    ],
];
```

## Attribute-Based Middleware Loading
You can now apply middleware directly to controller classes and methods using PHP attributes. This approach provides a cleaner and more declarative way to assign middleware.

#### Method-Level Middleware Attributes
To apply middleware to a specific controller method, use the `#[Middleware(...)]` attribute above the method definition. You can provide either fully qualified class names or aliases (registered in your `Kernel.php`).
```php
use Phaseolies\Http\Request;
use Phaseolies\Http\Response;
use Phaseolies\Routing\Attributes\Middleware;
use App\Http\Middleware\IpBasedUserBlocked;

class PostController extends Controller
{
    #[Middleware([IpBasedUserBlocked::class])]
    public function index(Request $request): Response
    {
        //
    }
}
```
In this example:
- The index method uses both class-based and alias-based middleware.
- Doppar will resolve and apply both forms appropriately before the request hits the method.
- This approach allows concise assignment without requiring route-level middleware declarations.

You can also load middleware using aliases (registered in your `Kernel.php`).

Here’s a basic example using an alias registered in the kernel:
```php
use Phaseolies\Http\Request;
use Phaseolies\Http\Response;
use Phaseolies\Routing\Attributes\Middleware;
use App\Http\Middleware\IpBasedUserBlocked;

class PostController extends Controller
{
    #[Middleware(['blocked'])]
    public function index(Request $request): Response
    {
        //
    }
}
```

### Passing Multiple Middleware
To apply multiple middleware, you can specify an array of middleware within the `#[Middleware(...)]` attribute. Each middleware in the array is executed in the order it is listed, allowing you to apply several checks or actions before the controller method is executed.

Here's an example using multiple middleware aliases:
```php
use Phaseolies\Http\Request;
use Phaseolies\Http\Response;
use Phaseolies\Routing\Attributes\Middleware;

class PostController extends Controller
{
    #[Middleware(['blocked', 'auth', 'is_admin'])]
    public function index(Request $request): Response
    {
        // This method is protected by multiple middleware:
        // 'blocked', 'auth', and 'is_admin'
    }
}
```
Each middleware in the list will be executed in the order provided. If any middleware returns a response (like a redirect or error), the next middleware or the controller method will not be executed.

#### Class-Level Middleware Attributes
You can also apply middleware globally to all methods within a controller by placing the `#[Middleware(...)]` attribute above the class declaration:
```php
use Phaseolies\Routing\Attributes\Middleware;

#[Middleware([IpBasedUserBlocked::class])]
class PostController extends Controller
{
    public function index()
    {
        // Global class-level middleware applied
    }

    public function store()
    {
        // Same middleware applied here
    }
}
```

You may mix both class-level and method-specific middleware. In such cases, both sets will be executed, with class-level middleware running first.

> Middleware specified using attributes must be registered in `\App\Http\Kernel.php`

## Assigning Middleware Using Routes
If you would like to assign middleware to specific routes, you may invoke the `middleware` method when defining the route:
```php
<?php

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\ProfileController;

Route::get('profile', [ProfileController::class,'index'])
    ->middleware('blocked');
```
The ProfileController index method is now protected by the `blocked` middleware. Update your middleware configuration to ensure authorization is required before accessing this method.

Or if your route is attribute based, then you can define middleware like this way
```php
class ProfileController extends Controller
{
    #[Route('profile', middleware: ['blocked'])]
    public function index()
    {
        //
    }
}
```

## Middleware Parameters
We can define multiple route middleware parameters. To define route middleware, add a : after the middleware name. If there are multiple parameters, separate them with a , comma. See the example.

Middleware can also receive additional parameters. For example, if your application needs to verify that the authenticated user has a given "role" before performing a given action, you could create an `EnsureUserHasRole` middleware that receives a role name as an additional argument.

Additional middleware parameters will be passed to the middleware after the $next argument:
```php
<?php

use Phaseolies\Support\Facades\Auth;
use Phaseolies\Middleware\Contracts\Middleware;
use Phaseolies\Http\Response;
use Phaseolies\Http\Request;
use Closure;

class EnsureUserHasRole implements Middleware
{
   /**
     * Handle an incoming request
     *
     * @param Request $request
     * @param \Closure(\Phaseolies\Http\Request) $next
     * @return Phaseolies\Http\Response
     */
    public function __invoke(Request $request, Closure $next, $role): Response
    {
        if (! $request->user()->hasRole($role)) {
            abort(403, 'Unauthorized.');
        }

        return $next($request);
    }
}
```

Middleware parameters may be specified when defining the route by separating the middleware name and parameters with a :
```php
<?php

use Phaseolies\Route;
use App\Http\Controllers\UserController;

Route::get('/', [UserController::class, 'index'])
    ->middlewar('role:admin');
```
In this example, the middleware named `role` will receive `admin` as a parameter. This `role` middleware must be registered in your `Kernel.php` file. It can then use the parameter to determine if the request should proceed.

### Multiple Middleware Parameters
We can define multiple route middleware parameters. To define route middleware, add a : after the middleware name. If there are multiple parameters, separate them with a , comma. See the example
```php
<?php

use Phaseolies\Route;
use App\Http\Controllers\UserController;

Route::get('/', [UserController::class, 'index'])
    ->middleware(['auth:admin,editor,publisher', 'is_subscribed:premium']);
```

In this example:
- The auth middleware receives three parameters: admin, editor, and publisher.
- The is_subscribed middleware receives one parameter: premium.

### Accept Parameters in Middleware

In the middleware class, define the handle method and accept the parameters as function arguments:
```php
/**
 * Handle an incoming request
 *
 * @param Request $request
 * @param \Closure(\Phaseolies\Http\Request) $next
 * @return Phaseolies\Http\Response
 */
public function __invoke(
    Request $request,
    Closure $next,
    string $admin,
    string $editor,
    string $publisher): Response
{
    // Parameters received:
    // $admin = 'admin'
    // $editor = 'editor'
    // $publisher = 'publisher'

    // Middleware logic goes here

    return $next($request);
}
```

## HTTP Cache Headers Middleware
Doppar provides a flexible and powerful middleware system that includes support for managing HTTP cache headers. This is useful for optimizing client-side caching and reducing unnecessary server load.

Doppar includes a `http.cache.headers` middleware, which may be used to quickly set the Cache-Control header for a group of routes. Directives should be provided using the `snake case` equivalent of the corresponding cache-control directive and should be separated by a semicolon. If etag is specified in the list of directives, an MD5 hash of the response content will automatically be set as the ETag identifier:
```php
Route::get('about', function () {
    return view('about');
})->middleware('http.cache.headers:public;max_age=3600;etag');
```
In this example:
- public allows the response to be cached by both browsers and intermediate caches.
- max_age=3600 tells clients to keep the response in cache for 3600 seconds (1 hour).
- etag adds an ETag header for cache validation, so clients can avoid downloading unchanged data

When using `http.cache.headers:public;max_age=3600;etag`, the response will include:
```php
Cache-Control: max-age=3600, public
ETag: "81c82a8ab1b8b063b642c1219e782a71"
```
`ETag` is automatically generated based on the response body using an MD5 hash. The middleware removes default PHP cache headers like Pragma: no-cache and Expires when applicable.
