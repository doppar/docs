---
title: Rate Limiter
description: Doppar Rate Limiter page
meta:
  - name: keywords
    content: Rate Limiter
---

## Rate Limiter
### Introduction
Rate limiting is a technique used to control the number of requests a client can make to an endpoint within a specified timeframe. It helps prevent abuse, reduce server load, and enhance security by limiting how frequently a user can access a given route.

By applying rate limits to routes or groups of routes, Doppar ensures fair usage and protects your system from potential misuse. In the Doppar PHP framework, you can apply rate limiting to specific routes using middleware.

## Apply Rate Limiter to Route
You can easily apply rate limiting to a route using the throttle middleware. The syntax follows the format throttle:`{max_requests},{minutes}`.

```php
class LoginController extends Controller
{
    #[Route(uri: 'login', middleware: ['throttle:10,1'])]
    public function login()
    {
        //
    }
}
```

If you define your route as file-based using the `Route` facade, you can set the rate limit in the following way.
```php
Route::post('login', [LoginController::class, 'login'])
    ->middleware(['throttle:10,1']);
```

Here `throttle:10,1` Limits requests to `10` per minute. This means that any client (such as a browser, mobile app, or script) can call the `/login` endpoint up to `10` times per minute. If they exceed this limit, the server will respond with a `429 Too Many Requests` status code.

You can directly pass the `rateLimit` and `rateLimitDecay` in your attribute based routing system to define rate limit in the following way.
```php
#[Route(uri: 'login', rateLimit: 10, rateLimitDecay: 1)]
public function login()
{
    //
}
```

## Customizing Rate Limits
You can customize the rate limit by changing the parameters:
```php
throttle:<max_requests>,<decay_minutes>
```
For example:
- `throttle:5,1 – 5` requests per minute
- `throttle:100,60 – 100` requests per hour

## Annotation-Based
In addition to applying rate limits via middleware, Doppar also supports annotation-based rate limiting. This allows you to define rate limits directly within your controller methods, keeping the configuration close to the logic it applies to.

You can add the `@RateLimit` annotation above a controller method. The format is:
```php
@RateLimit <max_requests>/<decay_minutes>
```
- `max_requests →` maximum number of requests allowed
- `decay_minutes →` time window (in minutes) for the limit

Example usage
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;

class PostController extends Controller
{
    /**
     * @RateLimit 10/1
     */
    public function show(Request $request)
    {
        //
    }
}
```

In the example above, the `show` method is limited to `10 requests` per minute per client. If a client exceeds this limit, Doppar automatically responds with a `429` Too Many Requests status code.

## Global Throttle
In addition to route and annotation-based rate limiting, Doppar provides a global throttle helper that allows you to manually control request limits anywhere in your application. This is especially useful for rate limiting custom logic, API calls, background jobs, or service-level operations where middleware is not applicable.

### Basic Usage
The global `throttle()` helper lets you check whether a specific action has exceeded its allowed number of attempts within a given time window.

Example: Limiting password reset attempts per user.
```php
$userId = auth()->id();
$key = 'password-reset:' . $userId;

if (throttle()->tooManyAttempts($key, 3)) {
    $seconds = throttle()->availableIn($key);

    return response()->json([
        'message' => "Too many password reset attempts. Try again in {$seconds} seconds."
    ], 429);
}

// Record the attempt (3 attempts per 10 minutes)
throttle()->hit($key, 600);
```
The throttle key should uniquely identify the action you want to limit.