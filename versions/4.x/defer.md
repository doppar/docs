---
title: Defer
description: Doppar defer page
meta:
  - name: keywords
    content: defer, cleanup, guaranteed cleanup, LIFO, execution scope, atomic lock, temporary file, transaction, runInDeferScope
---

## Defer

### Introduction

Code that acquires a temporary resource such as a cache lock, a file handle, a temporary file, a tracing span, or an open transaction must give that resource back no matter how the work ends. Doing it by hand usually means wrapping everything in `try`/`finally`, or remembering which lifecycle hook runs late enough to be safe.

Doppar's `defer()` method lets you register the cleanup **right next to the line that acquires the resource**. The callback is held until the current execution finishes, then runs whether the execution succeeded or threw an exception.

```php
$lock = cache()->locked('report:' . $reportId, seconds: 30);

if (! $lock->get()) {
    throw new \RuntimeException('The report is already being generated.');
}

defer(static function () use ($lock): void {
    $lock->release();
});

// Generate the report. 
// The lock is released when the execution ends.
```

### Why Use defer()

- *Acquire and release stay together* → you cannot forget the cleanup half
- *Guaranteed* → cleanup runs after success and after an exception
- *Ordered* → resources are released in reverse order of acquisition
- *Isolated* → a callback belongs to one execution and never leaks into the next request, job, or command in a long-lived worker

## Registering a Callback

You may register a callback through the application instance, the `App` facade, or the `defer()` helper. All three do the same thing.

```php
use Phaseolies\Support\Facades\App;

app()->defer(function () {
    // cleanup
});

App::defer(function () {
    // cleanup
});

defer(function () {
    // cleanup
});
```

`defer()` returns the application instance, so calls can be chained:

```php
app()
    ->defer(fn () => $first->release())
    ->defer(fn () => $second->release());
```

The callback is invoked without arguments. Capture the values it needs with `use (...)`. Prefer `static` closures so the callback does not accidentally keep `$this` (a controller, for example) alive.

```php
defer(static function () use ($handle): void {
    fclose($handle);
});
```

## Execution Order

Callbacks run in **last-in, first-out (LIFO)** order. The callback registered last runs first. This mirrors how resources are normally acquired and released: the thing you acquired last may depend on what you acquired first, so it must be released first.

```php
defer(fn () => Log::info('1. registered first, runs last'));
defer(fn () => Log::info('2. registered second, runs second'));
defer(fn () => Log::info('3. registered last, runs first'));
```

The log output order is `3`, `2`, `1`.

## When Callbacks Run

A deferred callback runs when the **execution that registered it ends**. What counts as an execution depends on where the code is running.

| Execution | Scope starts | Callbacks run |
|-----------|--------------|---------------|
| HTTP request | `Application::dispatch()` | `DispatchResult::terminate()` |
| Console command | Each command run by the console | When the command finishes |
| Queue job | `QueueWorker` runs every job in its own scope | When the job finishes |
| Custom runner | `app()->runInDeferScope()` | When the callback returns or throws |

### HTTP Requests

In an HTTP request, the scope starts when the application dispatches the request and ends during termination, which is after the response has been sent. The order inside termination is:

```text
Application::dispatch()          -> scope opens
Router, middleware, controller
Response sent
DispatchResult::terminate()
  -> terminating() callbacks
  -> deferred callbacks (LIFO)
  -> request-scoped services dropped
```

Because deferred callbacks run after the terminating callbacks but before request-scoped services are dropped, they can still use the container, the session, and the rest of the request state. See [`Request Lifecycle`](/versions/4.x/request-lifecycle) for the full flow.

Between dispatch and termination the request scope is **suspended**: it is detached from the active scope stack, so it keeps its callbacks but no longer receives `defer()` calls from unrelated code. Termination resumes it, which means a `terminating()` callback can still call `defer()` and have that cleanup run as part of the same request.

This also keeps overlapping requests apart. If a second request is dispatched before the first one is terminated (for example in a persistent server), each request still cleans up only its own callbacks, when its own `DispatchResult::terminate()` runs.

If handling the request throws before a response can be built, or an error page cannot be rendered, the scope is closed immediately and the callbacks run before the exception continues to propagate.

### Console Commands

Every command the console executes is wrapped in its own scope, so `defer()` works the same way inside a command. A command that is run through the console application from inside another command gets its own independent scope as well, and cleans up when it finishes rather than when the outer command does. Calling another command object's `run()` method directly does not go through the console, so that command shares the caller's scope.

Because the scope wraps the command itself, deferred callbacks run **before** Symfony renders an uncaught exception, and a custom reporter sees the exception that failed the command (see [`Custom Failure Reporting`](#custom-failure-reporting)).

```php
public function handle(): int
{
    $tmp = tempnam(sys_get_temp_dir(), 'export_');

    defer(static fn () => @unlink($tmp));

    // ... build the export ...

    return self::SUCCESS;
}
```

### Queue Jobs

The queue worker runs **every job in its own scope**, so `defer()` works directly inside a job's `handle()` method. Callbacks run when the job finishes, after success or after an exception, and always before the worker records the result (completing, releasing, or failing the job). Nothing registered by one job can reach the next job the worker handles.

```php
public function handle(): void
{
    $lock = cache()->locked('invoice:' . $this->invoiceId, seconds: 60);

    if (! $lock->get()) {
        throw new \RuntimeException('Invoice is already being sent.');
    }

    defer(static fn () => $lock->release());

    $this->sendInvoice();
}
```

Throwing from `handle()` fails the attempt; whether the worker retries the job depends on the job's `$tries` setting (which defaults to `1`). Cleanup runs after every attempt either way.

This holds both for jobs run in-process and for jobs run in a timeout child process: the callbacks run in the same process that ran the job.

> Jobs run synchronously through `dispatchSync()` or `queueAsSync()` execute inside the caller's execution, so their `defer()` callbacks belong to the caller's scope (the HTTP request or command) instead of getting a scope of their own.

### Custom Runners

For any other repeated or long-running unit of work, wrap it in `runInDeferScope()`. It opens a scope, runs your callback, then runs the deferred callbacks whether the callback returned or threw. The callback's return value is passed through, and exceptions are rethrown after cleanup.

```php
$result = app()->runInDeferScope(function () use ($message) {
    $lock = cache()->locked('message:' . $message->id, seconds: 60);

    if (! $lock->get()) {
        return null;
    }

    defer(static fn () => $lock->release());

    return $this->consume($message);
});
```

## Success and Failure

### Cleanup After Success and After an Exception

Deferred callbacks run on both paths. If the execution throws, the exception is still delivered to the caller after cleanup completes.

```php
try {
    app()->runInDeferScope(function () {
        defer(fn () => Log::info('cleanup always runs'));

        throw new \RuntimeException('Something went wrong.');
    });
} catch (\RuntimeException $e) {
    // 'cleanup always runs' was logged first, then this block runs.
}
```

### When a Cleanup Callback Throws

Every registered callback is attempted, even if an earlier one throws. A failing callback never stops the remaining cleanup.

```php
app()->runInDeferScope(function () {
    defer(fn () => Log::info('runs last'));
    defer(fn () => throw new \RuntimeException('cleanup failed'));
    defer(fn () => Log::info('runs first'));
});
```

Both log lines are written, and the failure is reported.

Cleanup exceptions are **reported, not thrown**:

- They never replace the original exception when the execution failed.
- They do not turn a successful execution into a failure.
- By default they are written to the `log` service as an error, or to PHP's `error_log` when no logger is available.

### Custom Failure Reporting

Use `reportDeferredExceptionsUsing()` to send cleanup failures somewhere else, such as an error tracker. The reporter receives the cleanup exception and the exception that ended the execution, if any (`null` after a successful execution).

```php
app()->reportDeferredExceptionsUsing(
    function (\Throwable $failure, ?\Throwable $original): void {
        report_to_tracker($failure, ['original' => $original?->getMessage()]);
    }
);
```

If your reporter itself throws, the exception is swallowed so that remaining cleanup still runs. Pass `null` to restore the default reporting.

## Execution Scopes

### Isolation Between Executions

HTTP requests and console commands also run inside an [`Execution Context`](/versions/4.x/execution-context), which gives the execution an id and lets services be bound to its lifetime. Deferred callbacks run before that context ends, so they can still use it.

A callback belongs only to the scope that was current when `defer()` was called, and `defer()` throws if none is current, for example after a request was dispatched and before it is terminated. When the scope ends, its callbacks run once and are discarded. Nothing carries over to the next request, job, or command, so long-lived workers (such as a queue worker or a persistent server) do not accumulate callbacks or keep captured values alive.

### Nested Scopes

Scopes can be nested. Each has its own independent stack, and `defer()` always registers into the innermost active scope.

```php
app()->runInDeferScope(function () {
    defer(fn () => Log::info('outer cleanup'));

    app()->runInDeferScope(function () {
        defer(fn () => Log::info('inner cleanup'));
    });

    Log::info('after inner scope');
});
```

The log output order is `inner cleanup`, `after inner scope`, `outer cleanup`.

If an outer scope ends while an inner scope was opened but never closed, the inner scope is ended first so that no callback is left behind.

### Registering During Cleanup

A deferred callback may itself call `defer()`. The new callback is added to the scope that is currently running and executes before cleanup finishes.

```php
defer(function () {
    Log::info('first cleanup');

    defer(fn () => Log::info('follow-up cleanup'));
});
```

### Suspending and Resuming a Scope

Most applications never need this; `runInDeferScope()` covers the common case. The lower-level methods exist for code whose work spans two phases that must not share the active scope, which is exactly how Doppar handles an HTTP request: the scope is open while the request is handled, then detached until termination.

`suspendDeferScope()` detaches a scope from the active stack **without running its callbacks**. While suspended, the scope keeps what it has registered, but `defer()` no longer targets it, and scopes started or ended in the meantime cannot affect it. `resumeDeferScope()` makes it current again.

```php
$scope = app()->beginDeferScope();

defer(fn () => Log::info('cleanup for phase one and two'));

// Phase one is done. Detach the scope; nothing runs yet.
app()->suspendDeferScope($scope);

// Other work can now use its own scopes without touching $scope.
app()->runInDeferScope(function () {
    defer(fn () => Log::info('independent scope'));
});

// Phase two: make the scope current again so it can receive more cleanup.
app()->resumeDeferScope($scope);
defer(fn () => Log::info('registered in phase two'));

// Run everything registered, last in, first out.
app()->endDeferScope($scope);
```

The log output order is `independent scope`, `registered in phase two`, `cleanup for phase one and two`.

Rules to keep in mind:

- A suspended scope must always be ended with `endDeferScope()`, otherwise its callbacks never run. `endDeferScope()` works on a suspended scope directly, so resuming first is only needed when you want to register more cleanup into it.
- If you suspend a scope that still has inner scopes that were never ended, those inner scopes are ended first.
- Resuming a scope that has already ended does nothing.
- While a scope is suspended and no other scope is active, `defer()` throws a `LogicException`.

## Calling defer() Outside a Scope

`defer()` needs an active execution scope. Called outside one, for example from a plain script that bootstraps the application without dispatching a request or running a command, it throws a `LogicException`:

```text
defer() was called outside an active execution scope. Run the code through
runInDeferScope(), an HTTP dispatch or a console command.
```

Doppar deliberately does **not** fall back to running the callback at the end of the process, because that would silently hold resources far longer than intended. When you do want application-lifetime behavior, use `terminating()` explicitly, or wrap the code in `runInDeferScope()`.

To check whether a scope is active:

```php
if (app()->hasDeferScope()) {
    defer($cleanup);
}
```

## Examples

### Cache Lock

Doppar cache stores expose atomic locks through `locked()`. Acquire the lock with `get()` (or `block()` to wait for it) and release it with `release()`. Deferring the release guarantees the lock is freed even if the work throws.

```php
$lock = cache()->locked('report:' . $reportId, seconds: 30);

if (! $lock->get()) {
    throw new \RuntimeException('The report is already being generated.');
}

defer(static function () use ($lock): void {
    $lock->release();
});

$report = $this->generateReport($reportId);
```

To wait for a busy lock instead of failing:

```php
$lock = cache()->locked('report:' . $reportId, seconds: 30);

if (! $lock->block(5)) {
    throw new \RuntimeException('Timed out waiting for the report lock.');
}

defer(static fn () => $lock->release());
```

See [`Cache`](/versions/4.x/caching) for the full atomic lock API.

### Resource Cleanup

Close file handles, streams, and sockets without a surrounding `try`/`finally`:

```php
$handle = fopen($path, 'rb');

if ($handle === false) {
    throw new \RuntimeException("Unable to open {$path}.");
}

defer(static function () use ($handle): void {
    if (is_resource($handle)) {
        fclose($handle);
    }
});

while (($line = fgets($handle)) !== false) {
    $this->process($line);
}
```

### Transaction

Open a transaction and register a safety net that rolls it back if the work never committed:

```php
use Phaseolies\Support\Facades\DB;

DB::beginTransaction();

defer(static function (): void {
    if (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});

$order = Order::create($data);
OrderItem::saveMany(
    array_map(fn (array $item) => $item + ['order_id' => $order->id], $items)
);

DB::commit();
```

If an exception is thrown before `DB::commit()`, the transaction is rolled back during cleanup. If the work commits, the level is back to zero and the callback does nothing. For simple cases where the whole unit of work lives in one closure, `DB::transaction()` is still the more direct tool.

### Multiple Resources in Order

Because callbacks run last-in, first-out, resources acquired later are released first, which is what you want when they depend on each other:

```php
$lock = cache()->locked('import', 120);
$lock->get();
defer(static fn () => $lock->release());     // runs last

DB::beginTransaction();
defer(static function (): void {             // runs first
    if (DB::transactionLevel() > 0) {
        DB::rollBack();
    }
});
```

The transaction is rolled back first, and only then is the lock released.

## API Reference

| Method | Returns | Description |
|--------|---------|-------------|
| `defer(callable $callback)` | `Application` | Register a cleanup callback for the current execution (LIFO) |
| `runInDeferScope(callable $callback)` | `mixed` | Run a callback inside its own scope and clean up afterwards |
| `hasDeferScope()` | `bool` | Determine whether a scope is currently active |
| `beginDeferScope()` | `DeferScope` | Open a new scope manually (lower level) |
| `suspendDeferScope(DeferScope $scope)` | `void` | Detach a scope from the active stack without running it |
| `resumeDeferScope(DeferScope $scope)` | `void` | Make a suspended scope the current scope again |
| `endDeferScope(DeferScope $scope, ?Throwable $exception = null)` | `void` | Run and discard a scope, ending any unclosed inner scopes first |
| `reportDeferredExceptionsUsing(?callable $reporter)` | `Application` | Customize how cleanup exceptions are reported |

`beginDeferScope()`, `suspendDeferScope()`, `resumeDeferScope()` and `endDeferScope()` are the building blocks behind HTTP dispatch and `runInDeferScope()`. Prefer `runInDeferScope()` in application code; if you do open a scope by hand, always pair it with `endDeferScope()` in a `finally` block.

## Defer vs terminating()

| | `defer()` | `terminating()` |
|---|-----------|-----------------|
| Registered | Anywhere, at the point of acquiring a resource | Once, during application bootstrap |
| Lifetime | The current execution only | Every request |
| Order | Last-in, first-out | Registration order |
| Receives | Nothing (use `use (...)`) | `$request`, `$response`, `$exception` |
| Typical use | Releasing locks, files, spans, transactions | Logging, metrics, post-response work |

Use `defer()` for local resource ownership and `terminating()` for application-wide post-response behavior.