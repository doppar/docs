---
title: Request lifecycle
description: Doppar request lifecycle page
meta:
  - name: keywords
    content: request lifecycle, dispatch, terminate, middleware, routing, response, doppar
---

## Request Lifecycle

### Introduction

Understanding the request lifecycle makes the framework easier to reason
about. When you know where the request enters, how it becomes a
response, and what runs after the response is sent, debugging and
structuring your application code becomes much more predictable.

Doppar's HTTP lifecycle has two clear phases:
request handling and request termination. First, the framework captures
the incoming request, resolves it through routing and middleware, and
sends the final response. After that, Doppar can still run registered
termination callbacks for logging, cleanup, metrics, or other
post-response work.

## High-Level Flow

### Lifecycle Overview

At a high level, a Doppar web request moves through this flow:

```text
HTTP Request
  -> public/index.php
  -> bootstrap/app.php
  -> Application::dispatch()
  -> Application::handle()
  -> Router and middleware pipeline
  -> Controller or route closure
  -> Response preparation and send
  -> DispatchResult::terminate()
  -> Application terminating callbacks
```

This split is important. The response is sent before the termination
callbacks run.

## Entry Point

### `public/index.php`

Every web request enters through `public/index.php`. Your web server
points incoming requests to this file, making it the front controller
for the application.

The file loads Composer, bootstraps the application, captures the HTTP
request, dispatches it, and then explicitly completes the termination
lifecycle:

```php
use Phaseolies\Http\Request;

require __DIR__ . '/../vendor/autoload.php';
require __DIR__ . '/../bootstrap/app.php';

$response = $app->dispatch(Request::capture());

$response->terminate();
```

This means Doppar now has an explicit post-response lifecycle step
instead of ending the process immediately after the response is sent.

## Bootstrapping

### `bootstrap/app.php`

The `bootstrap/app.php` file creates the application instance and
configures the framework before request handling begins. This is where
you define base paths, relaxed CSRF paths, and lifecycle hooks such as
termination callbacks.

Because the application object is built before the request is handled,
this file is the right place to register logic that should always run at
the end of a request.

## Request Handling

### Dispatching the Request

When `dispatch()` is called, Doppar resolves the request into a response
and sends that response to the client.

Internally, `Application::dispatch()`:

1. calls `Application::handle($request)`
2. lets the router resolve the request
3. receives the final response from the route or controller
4. prepares and sends the response
5. returns a `DispatchResult` instance for termination handling

That returned `DispatchResult` is what makes this front-controller style
possible:

```php
$response = $app->dispatch(Request::capture());
$response->terminate();
```

## Routing and Middleware

### Request Resolution

Inside `Application::handle()`, Doppar passes the request into the
router. The router is responsible for matching the request to the
correct route and running any middleware that belongs to that route or
the wider HTTP pipeline.

Middleware can inspect or modify the request before your controller
runs, and it can also inspect or modify the outgoing response on the way
back out.

In practical terms, this stage is where features such as
authentication, CSRF validation, request filtering, and response
transformation usually happen.

## Response Phase

### Sending the Response

Once the controller or route closure returns, Doppar prepares the final
response and sends it to the client.

This is the point where the browser or API client receives the output.
For most frameworks, this feels like the end of the request, but Doppar
now keeps a small lifecycle window open for post-response termination
work.

## Termination Phase

### Explicit Termination

After the response has already been sent, the returned `DispatchResult`
can run the application termination lifecycle:

```php
$response = $app->dispatch(Request::capture());
$response->terminate();
```

Calling `terminate()` triggers all callbacks that were registered
through `Application::terminating(...)`.

If the dispatch result is ignored, Doppar still has a safe fallback that
ensures termination runs automatically. Even so, the explicit
`->terminate()` call is the clearest and recommended flow for the front
controller.

### Using `terminating(...)`

Register a termination callback in `bootstrap/app.php` when you need
logic to run after the response is sent but before the framework fully
finishes the request lifecycle.

```php
use Phaseolies\Http\Request;
use Phaseolies\Http\Response;

return $app
    ->withBasePath(basePath: $basePath)
    ->setRelaxablePaths(relaxablePaths: [
        // The paths listed below will bypass CSRF token verification.
    ])
    ->terminating(function (
        Request $request,
        ?Response $response,
        ?\Throwable $exception = null
    ) {
        // Runs after the response is sent, during application termination.
    })
    ->configure(app: $app)
    ->build();
```

This hook is useful for:

- request-end logging
- metrics collection
- cleanup tasks
- audit writes
- deferred side effects that should not block the main response body

### Callback Context

Termination callbacks may receive the current request, the resolved
response, and an exception when the request ended through an HTTP
exception path.

Typical callback signature:

```php
->terminating(function (
    Request $request,
    ?Response $response,
    ?\Throwable $exception = null
) {
    //
})
```

Important behavior to know:

- `$request` is the incoming request that was dispatched
- `$response` is the resolved response when one exists
- `$response` may be `null` when the request ends through an HTTP exception path
- `$exception` is available when termination happens after an exception-based response flow

## Exception Handling

### Requests That End with an HTTP Exception

If request handling throws an `HttpException`, Doppar still sends the
exception response and returns a `DispatchResult`. That means the
termination lifecycle still runs, but the callback will receive the
exception context instead of a normal resolved response.

This keeps termination behavior consistent across both successful and
exception-driven request endings.

## Summary

### Final Mental Model

The Doppar request lifecycle is best understood like this:

1. The request is captured in `public/index.php`.
2. The application is bootstrapped from `bootstrap/app.php`.
3. The request is dispatched through routing and middleware.
4. The response is prepared and sent to the client.
5. The dispatch result terminates the lifecycle.
6. Registered `terminating(...)` callbacks run after the response is sent.

That final termination step is what makes Doppar's current lifecycle
different from the older "send response and stop immediately" model.
