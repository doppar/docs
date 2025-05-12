## Middleware
### Introduction
Middleware offers a powerful way to inspect, filter, and act on HTTP requests before they reach your application's core logic. For instance, Doppar includes built-in middleware that ensures a user is authenticated. If the user isn't authenticated, the middleware can redirect them to a login page; otherwise, it allows the request to proceed.

Beyond authentication, middleware can handle a wide range of tasks. For example, a logging middleware might record every incoming request, while another could handle input sanitization or API rate limiting. Doppar comes bundled with several useful middleware for tasks like authentication and CSRF protection. Custom middleware can be created and are typically stored in the `app/Http/Middleware` directory of your application.

### Create New Middleware
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

### Register Middleware
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

#### Assigning Middleware to Routes
If you would like to assign middleware to specific routes, you may invoke the `middleware` method when defining the route:
```php
<?php

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\ProfileController;

Route::get('profile', [ProfileController::class,'index'])->middleware('blocked');
```
The ProfileController index method is now protected by the `blocked` middleware. Update your middleware configuration to ensure authorization is required before accessing this method.

### Middleware Parameters
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

Route::get('/', [UserController::class, 'index'])->middlewar('role:admin');
```
In this example, the middleware named `role` will receive `admin` as a parameter. This `role` middleware must be registered in your `Kernel.php` file. It can then use the parameter to determine if the request should proceed.

#### Multiple Middleware Parameters
We can define multiple route middleware parameters. To define route middleware, add a : after the middleware name. If there are multiple parameters, separate them with a , comma. See the example
```php
<?php

use Phaseolies\Route;
use App\Http\Controllers\UserController;

Route::get('/', [UserController::class, 'index'])->middleware(['auth:admin,editor,publisher', 'is_subscribed:premium']);
```

In this example:
- The auth middleware receives three parameters: admin, editor, and publisher.
- The is_subscribed middleware receives one parameter: premium.

#### Accept Parameters in Middleware

In the middleware class, define the handle method and accept the parameters as function arguments:
```php
/**
 * Handle an incoming request
 *
 * @param Request $request
 * @param \Closure(\Phaseolies\Http\Request) $next
 * @return Phaseolies\Http\Response
 */
public function __invoke(Request $request, Closure $next, $admin, $editor, $publisher): Response
{
    // Parameters received:
    // $admin = 'admin'
    // $editor = 'editor'
    // $publisher = 'publisher'

    // Middleware logic goes here

    return $next($request);
}
```

### Cache Control Middleware
Doppar includes a `cache.headers` middleware, which may be used to quickly set the Cache-Control header for a group of routes. Directives should be provided using the "snake case" equivalent of the corresponding cache-control directive and should be separated by a semicolon. If etag is specified in the list of directives, an MD5 hash of the response content will automatically be set as the ETag identifier:
```php
Route::get('privacy', function (Request $request) {
    return view('page.privacy');
})->middleware('cache.headers:public;max_age=2628000;etag');
```
This setup allows efficient caching by making your headers both consistent and easy to manage.