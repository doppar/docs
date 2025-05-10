## Rate Limiter
### Introduction
Rate limiting is a technique used to control the number of requests a client can make to an endpoint within a specified timeframe. It helps prevent abuse, reduce server load, and enhance security by limiting how frequently a user can access a given route.

By applying rate limits to routes or groups of routes, Doppar ensures fair usage and protects your system from potential misuse. In the Doppar PHP framework, you can apply rate limiting to specific routes using middleware.

### Apply Rate Limiter to Route
You can easily apply rate limiting to a route using the throttle middleware. The syntax follows the format throttle:`{max_requests},{minutes}`.

```php
Route::get('show', [PostController::class, 'index'])
    ->middleware(['throttle:10,1']);
```

Here `throttle:10,1` Limits requests to 10 per minute. This means that any client (such as a browser, mobile app, or script) can call the `/show` endpoint up to 10 times per minute. If they exceed this limit, the server will respond with a `429 Too Many Requests` status code.

### Customizing Rate Limits
You can customize the rate limit by changing the parameters:
```php
throttle:<max_requests>,<decay_minutes>
```
For example:
- throttle:5,1 – 5 requests per minute
- throttle:100,60 – 100 requests per hour


