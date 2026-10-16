---
title: Execution Context
description: Doppar execution context - an immutable description of the current request, command or job, with execution-scoped services and lifecycle hooks
meta:
  - name: keywords
    content: execution context, bindExecution, execution scope, request scope, scoped services, tenant, correlation id, request id, log context, lifecycle hooks, Execution facade, stamp, within, runInContext, worker, long-running, state leak
---

## Execution Context

### Introduction

Every piece of work a Doppar application performs happens inside an **execution**: an HTTP request, a console command, or a job handled by a worker. Most of the state your code keeps is meant to live for exactly one of them. The tenant resolved from a header, the user being impersonated, a unit-of-work object, a per-request cache. When that state is held in a plain singleton, it outlives the execution that created it. In a persistent process such as a queue worker or a long-lived server, the next request or job then starts with the previous one's tenant, and nothing tells you it happened.

An **execution context** gives each execution an identity and a lifetime. It is a small immutable object holding a unique id, a type (`request`, `console` or `worker`) and metadata, and the container uses it as a boundary: services bound to the `execution` scope are created inside a context and destroyed with it.

```php
use Phaseolies\Support\Facades\Execution;

Execution::id();               // "9f2c41e07b1d3a58"
Execution::type();             // "request"
Execution::get('path');        // "/orders/10"

// Metadata learned mid-request is stamped onto the context
Execution::stamp(['tenant' => 'acme']);

Log::error('Payment failed'); // the log entry carries the id, type and tenant
```

### Why Use Execution Contexts

- *Leak-proof* → an execution-scoped service can never be seen by the next request, command or job
- *Traceable* → one id ties together every log entry an execution wrote
- *Uniform* → requests, console commands and workers follow the same lifecycle and the same API
- *Immutable* → a context never changes after it is created, so it is safe to pass to listeners and closures
- *No global state to manage* → the container owns the stack of active contexts, and it is cleaned up for you

## How an Execution Is Described

A context is an `ExecutionContext` instance with the following parts:

| Part | Description |
|------|-------------|
| `id` | A unique 16 character identifier. It stays the same when metadata is stamped onto the context |
| `type` | `request`, `console`, `worker`, or any string you choose |
| `metadata` | A key and value array. A nested context inherits its parent's metadata and may override keys |
| `parent` | The context this one was started inside of, or `null` for a root context |
| `startedAt` | The Unix timestamp, with microseconds, at which the context began |

Doppar creates a context for you at each entry point:

| Execution | Type | Metadata | Ends |
|-----------|------|----------|------|
| HTTP request | `request` | `method`, `path`, and `request_id` when an `X-Request-Id` header was sent | After the terminating callbacks, deferred callbacks and request cleanup have run |
| Console command | `console` | `command` | After the command and its deferred callbacks have finished |
| Queue job | `worker` | `queue`, `queue_job_id`, `attempt`, `connection`, then `job` and `job_id` once the job is built | After the job has been recorded as done, retried or failed, and its follow-up work has run |
| Custom runner | Your choice | Your choice | When your callback returns or throws |

## Reading the Current Context

Use the `Execution` facade or the `execution()` helper. Both answer from the container's stack of active contexts.

```php
use Phaseolies\Support\Facades\Execution;

Execution::current();             // ExecutionContext|null
Execution::running();             // true while any context is active
Execution::id();                  // string|null
Execution::type();                // string|null
Execution::get('path');           // a single metadata value
Execution::get('tenant', 'none'); // with a default
Execution::all();                 // every metadata key, inherited ones included
```

The helper returns the same service:

```php
execution()->id();
execution()->get('method');
```

Outside any execution, `current()`, `id()` and `type()` return `null`, `all()` returns an empty array, and `get()` returns the default you pass. A metadata key that exists with the value `null` returns `null`, not the default. If you work with the container directly, `app()->executionContext()` returns the active `ExecutionContext` object itself.

### The ExecutionContext Object

```php
$context = Execution::current();

$context->id();           // "9f2c41e07b1d3a58"
$context->type();         // "request"
$context->get('path');    // "/orders/10"
$context->has('tenant');  // false
$context->all();          // ['method' => 'GET', 'path' => '/orders/10', ...]
$context->parent();       // ExecutionContext|null
$context->depth();        // 0 for a root context
$context->startedAt();    // 1760632461.0412
$context->toArray();      // the log-friendly form, see "Logging"
```

The object has no setters. Everything that appears to change a context returns a new instance.

## Stamping Metadata

Some facts about an execution are only known once it is already running: the tenant resolved from the subdomain, the user an admin is acting as, the order a webhook belongs to. Add them with `stamp()`:

```php
<?php

namespace App\Http\Middleware;

use Closure;
use Phaseolies\Http\Request;
use Phaseolies\Http\Response;
use Phaseolies\Support\Facades\Execution;

class ResolveTenant
{
    public function __invoke(Request $request, Closure $next): Response
    {
        Execution::stamp([
            'tenant' => $request->header('X-Tenant'),
        ]);

        return $next($request);
    }
}
```

`stamp()` returns the updated context and keeps the **same id**, so everything that already used the id (log entries, scoped services) still belongs to the same execution. The instance you held before the call is untouched:

```php
$before = Execution::current();

$after = Execution::stamp(['tenant' => 'acme']);

$before->has('tenant');            // false, contexts are immutable
$after->get('tenant');             // "acme"
$before->id() === $after->id();    // true
```

Log entries written before the call keep the old metadata. Entries written after it carry the new metadata.

> `stamp()` throws a `LogicException` when no context is active, since there is nothing to stamp.

## Execution-Scoped Services

### Binding Scopes

A binding declares how long an instance it produces lives:

| Scope | Lifetime | Declared with |
|-------|----------|---------------|
| `singleton` | The whole application. One instance, shared everywhere | `singleton()` |
| `execution` | One execution context. Shared inside it, destroyed with it | `bindExecution()` |
| `transient` | Not shared. A new instance on every resolution | `bind()` |

Existing `bind()` and `singleton()` calls behave exactly as they always have. The `execution` scope is new, and it is the right choice for any service that holds per-request, per-command or per-job state.

### Registering a Service

Register an execution-scoped service from a launcher, next to your other bindings:

```php
<?php

namespace App\Launchers;

use App\Support\TenantState;
use Phaseolies\Launchers\ServiceLauncher;

class AppLauncher extends ServiceLauncher
{
    public function launch(): void
    {
        $this->app->bindExecution(TenantState::class, fn () => new TenantState());
    }
}
```

A class name works as well. The container builds it and injects its dependencies:

```php
$this->app->bindExecution(TenantState::class);
```

To choose the scope by name, for example from configuration, use `bindWithScope()`:

```php
$this->app->bindWithScope(TenantState::class, fn () => new TenantState(), 'execution');
```

It accepts `singleton`, `execution` or `transient` and throws an `InvalidArgumentException` for anything else.

### Using a Scoped Service

Resolve it the way you resolve any service. Inside one execution every resolution returns the **same instance**, including constructor injection into controllers, middleware and other services:

```php
<?php

namespace App\Http\Controllers;

use App\Support\TenantState;
use Phaseolies\Http\Controller;

class InvoiceController extends Controller
{
    public function __construct(private TenantState $tenant) {}

    public function index()
    {
        // $this->tenant is the instance for this request only
    }
}
```

The next request receives a brand-new instance:

```php
// Request 1
app(TenantState::class)->name = 'acme';
app(TenantState::class)->name;    // "acme"

// Request 2, the same process
app(TenantState::class)->name;    // null, a fresh instance
```

What you can rely on:

- Every execution gets its **own** instance, created on first use and not before.
- The instance is created once per execution, however many times it is resolved.
- It is dropped when the context ends, so no state carries over to the next request, command or job.
- Terminating callbacks and deferred callbacks still resolve the **same** instance the request used, because the context ends after both of them.
- `extend()` keeps the scope of the binding it extends.
- Re-binding a service discards any instance that was already created for it.

### Resolving Outside an Execution

An execution-scoped service needs a context to belong to. Resolving one with no context active throws a `LogicException`:

```text
Cannot resolve execution-scoped service [App\Support\TenantState] outside an
execution context. Resolve it during an HTTP dispatch, a console command or
inside runInContext().
```

Doppar deliberately does **not** hand out an instance that nobody owns, because it would never be destroyed and would quietly turn into a singleton. For a script that bootstraps the application without dispatching a request or running a command, wrap the work in a context:

```php
app()->runInContext('console', ['task' => 'rebuild-index'], function () {
    app(TenantState::class)->name = 'acme';
});
```

### Do Not Hold a Scoped Service in a Singleton

A singleton lives for the whole process, so a scoped service injected into its constructor is resolved once and then kept. After the first execution ends the singleton is left holding an instance that is no longer current:

```php
// Wrong: the singleton keeps request 1's TenantState forever
$this->app->singleton(ReportBuilder::class, fn ($app) => new ReportBuilder(
    $app->get(TenantState::class)
));
```

Resolve the scoped service at the moment you use it instead:

```php
class ReportBuilder
{
    public function build(): array
    {
        $tenant = app(TenantState::class);   // the current execution's instance

        return $this->queryFor($tenant->name);
    }
}
```

Alternatively give `ReportBuilder` the same scope, with `bindExecution()`, so it is rebuilt together with the state it depends on.

### Flushing the Container

`flush()` (and `reset()`) first ends every execution context that is still open, innermost first and suspended ones last, so their `ending` listeners get the chance to clean up. Only then are the bindings, instances and listeners cleared. This is mainly relevant in tests and in code that rebuilds the container; a normal request or command never needs it.

### Inspecting a Binding

```php
app()->scopeOf(TenantState::class);   // "execution"
app()->scopeOf('log');                // "singleton"
app()->scopeOf('not-bound');          // null
```

### Relationship to Request-Scoped Cleanup

Doppar already forgets the resolved instances of services that were bound **after** boot when a request is terminated, so request services do not leak between requests. That sweep is implicit, it only covers HTTP requests, and it depends on *when* a binding was registered. The `execution` scope is the explicit counterpart: you state the lifetime on the binding itself, and it holds for requests, console commands and workers alike.

## Nested Contexts

Contexts nest. A child inherits its parent's metadata, may override individual keys, and never changes the parent:

```php
use Phaseolies\Support\Facades\Execution;

// Inside an HTTP request with tenant "acme" stamped on it
Execution::within('worker', ['job' => 'SendInvoice', 'tenant' => 'globex'], function ($context) {
    $context->type();             // "worker"
    $context->get('tenant');      // "globex", overridden
    $context->get('path');        // "/orders/10", inherited
    $context->parent()->type();   // "request"
    $context->depth();            // 1
});

Execution::get('tenant');          // "acme", the parent is untouched
```

A nested context gets its **own** instance of every execution-scoped service. The outer context's instance is left alone and is returned again as soon as the nested context ends.

```php
$outer = app(TenantState::class);

$inner = Execution::within('worker', [], fn () => app(TenantState::class));

$outer === $inner;                    // false
$outer === app(TenantState::class);   // true
```

When an outer context ends while a nested one is still open, the nested context is ended first, each with its own end listeners, so nothing is left behind.

## Running Code in a Context

### within()

`within()` begins a context, runs your callback, and **always** ends the context afterwards, whether the callback returned or threw. The callback receives the new context, the return value is passed through, and exceptions are rethrown after teardown.

```php
use Phaseolies\Support\Facades\Execution;

$total = Execution::within('worker', ['job' => 'ImportOrders'], function ($context) {
    Log::info('Import started');    // carries the worker context

    return $this->import();
});
```

This is the building block for any long-lived process. Run each unit of work in its own context and scoped services and metadata can never carry over from one unit to the next:

```php
foreach ($messages as $message) {
    Execution::within('worker', ['message' => $message->id], function () use ($message) {
        $this->consume($message);
    });
}
```

The container exposes the same operation as `runInContext($type, $metadata, $callback)`.

### Manual Control

When the beginning and the end of the work are in different places, begin and end the context yourself. Always end it in a `finally` block.

```php
$context = app()->beginContext('worker', ['job' => 'Reindex']);

try {
    $this->reindex();
} finally {
    app()->endContext($context);
}
```

- Ending a context also ends every context that was opened inside it and never closed.
- Ending a context that has already ended does nothing.
- `beginContext()` nests inside the current context, if there is one.

### Suspending and Resuming

Most applications never need this. It exists for work that spans two phases which must not share the active context, which is exactly how Doppar handles an HTTP request. The context is open while the response is produced, then detached until termination.

`suspendContext()` detaches a context from the active stack **without ending it**. While suspended it keeps its scoped instances, but no longer counts as the current context. `resumeContext()` makes it current again, and `endContext()` ends it, resumed or not.

```php
$context = app()->beginContext('request');

app()->suspendContext($context);
app()->executionContext();          // null, nothing is current

app()->resumeContext($context);
app()->executionContext();          // the request context again

app()->endContext($context);
```

A suspended context must always be ended, otherwise its end listeners never run and its scoped instances are never released.

What you can rely on:

- Resuming a context restores its **latest** metadata, including anything stamped after you obtained the object. Termination callbacks and `ending` listeners of a request therefore see the tenant that a middleware stamped.
- Resuming a context that has already ended does nothing. An ended context can never be brought back, and neither can one that was ended by `flush()`.
- Resuming a context that is already current does nothing.

## Lifecycle Hooks

Register listeners when the application boots, from the `launch()` method of a service launcher, in the same way as signal listeners:

```php
<?php

namespace App\Launchers;

use Phaseolies\Launchers\ServiceLauncher;
use Phaseolies\Support\ExecutionContext;
use Phaseolies\Support\Facades\Execution;

class AppLauncher extends ServiceLauncher
{
    public function launch(): void
    {
        // Every execution
        Execution::beginning(function (ExecutionContext $context) {
            // ...
        });

        // Only HTTP requests
        Execution::beginning(function (ExecutionContext $context) {
            metrics()->increment('http.requests');
        }, 'request');

        // Only console commands
        Execution::ending(function (ExecutionContext $context) {
            metrics()->timing('console.duration', microtime(true) - $context->startedAt());
        }, 'console');
    }
}
```

| Hook | Runs | Receives |
|------|------|----------|
| `Execution::beginning($listener, $type = null)` | Right after a context becomes current | The new `ExecutionContext` |
| `Execution::ending($listener, $type = null)` | When a context ends, **before** its scoped services are dropped | The ending `ExecutionContext` |

The optional second argument limits a listener to one context type. Without it, the listener runs for every type.

What you can rely on:

- Listeners run in the order they were registered.
- An `ending` listener can still resolve the context's scoped services, so it can flush or persist them.
- When a nested context ends, its listeners run before its parent's.
- If a listener throws, the remaining listeners still run and the context is still torn down completely. The first failure is rethrown afterwards.
- A listener that throws from `beginning` stops the context from starting: the context is ended again and the exception propagates.

For cleanup that belongs next to the line that acquired a resource, prefer [`defer()`](/versions/4.x/defer). Deferred callbacks run before the context ends, so they can still use the context and its scoped services.

## The Request Lifecycle

For an HTTP request the context brackets the whole lifecycle:

```text
Application::dispatch()          -> request context begins, defer scope opens
Router, middleware, controller   -> Execution::stamp() may enrich the context
Response sent
                                 -> context and defer scope suspended
DispatchResult::terminate()      -> context and defer scope resumed
  -> terminating() callbacks
  -> deferred callbacks (LIFO)
  -> request services dropped
  -> request context ends        -> ending listeners run, scoped services dropped
```

If handling the request throws before a response can be built, or an error page cannot be rendered, the context is ended immediately and the exception continues to propagate. If a second request is dispatched before the first is terminated, each request still ends its own context. See [`Request Lifecycle`](/versions/4.x/request-lifecycle) for the full flow.

## Console Commands

Every command the console runs gets a `console` context carrying the command name:

```php
public function handle(): int
{
    Execution::type();            // "console"
    Execution::get('command');    // "orders:export"

    Log::info('Export started');  // carries the command context

    return self::SUCCESS;
}
```

The context is opened around the command's defer scope, so deferred callbacks run first and the context ends last. A command run through the console application from inside another command gets its own nested context.

## Logging

While a context is active, every log entry gets an `execution` key added to its context data. It holds the id, the type, the parent id and the metadata, so one search finds everything a request or job did.

```php
Execution::stamp(['tenant' => 'acme']);

Log::error('Payment failed', ['order' => 10]);
```

```text
[2026-10-16T21:19:21.116749+06:00] stack.ERROR: Payment failed {"order":10,"execution":{"id":"9f2c41e07b1d3a58","type":"request","parent_id":null,"metadata":{"method":"POST","path":"/checkout","request_id":"abc-123","tenant":"acme"}}}
```

- The data comes from the active context at the moment you log. Nothing is stored on the logger, so there is no global state to reset between executions.
- It applies to every handler, because it travels in the standard log context.
- Outside any context, log entries are exactly as before.
- An `execution` key you pass yourself is never overwritten.
- If the caller sends an `X-Request-Id` header, it appears as `request_id` and lets you correlate a log line with the load balancer or the calling service.

### Personal Data in Logs

Every log entry written during an execution carries its metadata, so what a context holds is written to your logs many times over. For that reason Doppar only puts in what is needed to correlate entries: the method, the path without its query string, and the request id. The client IP address, the user, and anything else that identifies a person are **not** added automatically.

Add them deliberately, once you have decided the data belongs in your logs:

```php
Execution::stamp([
    'ip' => $request->ip(),
    'user_id' => $request->user()?->id,
]);
```

Prefer opaque identifiers such as a user id over names or email addresses. Remember that the path itself can contain identifiers, for example `/users/10/invoices`.

See [`Logging`](/versions/4.x/logging) for channels and handlers.

## Failure Behavior

| Situation | Result |
|-----------|--------|
| Resolving a scoped service with no active context | `LogicException`, nothing is created |
| `stamp()` with no active context | `LogicException` |
| An empty context type | `InvalidArgumentException` |
| An unknown scope passed to `bindWithScope()` | `InvalidArgumentException` |
| The callback given to `within()` throws | The context is ended, then the exception is rethrown |
| An `ending` listener throws | Every other listener still runs, the context is fully torn down, then the first failure is rethrown |
| A `beginning` listener throws | The context is ended again and the exception propagates |
| Ending a context twice | Nothing happens |
| Resuming a context that has ended | Nothing happens, the context stays ended |
| `flush()` while contexts are open | Every open context is ended first and its `ending` listeners run, then all state is cleared. If a listener throws, the state is still cleared and the first failure is rethrown |

## Queue Jobs

The queue worker runs every job in its own `worker` context, inside the same scope as its [`defer()`](/versions/4.x/defer#queue-jobs) callbacks. Nothing needs to be configured. The context carries identifiers only, never the payload:

| Metadata | Description |
|----------|-------------|
| `queue` | The queue the job was taken from |
| `queue_job_id` | The id of the reserved job in the queue backend |
| `attempt` | The attempt number, starting at `1` |
| `connection` | The queue connection of the worker, or `null` for the default |
| `job` | The job class, added once the job has been built |
| `job_id` | The job's own id, added once the job has been built |

```php
public function handle(): void
{
    Execution::type();          // "worker"
    Execution::get('attempt');  // 2

    Log::info('Sending invoice');   // the log entry names the queue, job and attempt
}
```

Because every job gets a fresh context, an execution-scoped service is created again for each job and is dropped when the job finishes. The same context also covers the code the worker runs around `handle()`: `failed()`, the retry handling, a chained job's dispatch and the processed hook. Stamp extra metadata, such as a tenant stored on the job, as soon as the job starts:

```php
public function handle(): void
{
    Execution::stamp(['tenant' => $this->tenantId]);

    // ...
}
```

A job does not inherit the context of the request that dispatched it, see [`Limitations`](#limitations). A worker running with an older version of the framework that has no execution contexts keeps working, it simply runs without a context.

## Long-Running Workers

A persistent process keeps its container between executions. The context is what lets that be safe: scoped instances are keyed by context id and removed when the context ends, so a worker that runs thousands of jobs holds no more state than one that ran a single job.

The queue worker already does this for queue jobs. For your own workers, consumers and loops, run each unit of work in its own context:

```php
use Phaseolies\Support\Facades\Execution;

while ($message = $this->next()) {
    Execution::within('worker', ['message' => $message->id, 'queue' => 'billing'], function () use ($message) {
        $this->handle($message);
    });
}
```

Each message then has its own id in the logs, its own scoped services, and its own `ending` hooks.

## Execution Context vs Other Tools

| | `Execution` scope | `defer()` | `terminating()` | `singleton()` |
|---|-------------------|-----------|-----------------|---------------|
| Purpose | Holding per-execution state and identity | Releasing a resource you acquired | Post-response work for every request | Sharing one instance everywhere |
| Lifetime | One execution | One execution | Every request | The process |
| Registered | At boot, on the binding | Anywhere, at the point of acquisition | Once, at bootstrap | At boot |
| Runs in | Requests, commands, workers | Requests, commands, workers | HTTP requests only | Everywhere |

They work together. A request's deferred callbacks run first, then request services are dropped, then the context ends and its scoped services are released.

## Limitations

- A context describes **one process**. It is not carried across a process boundary, so a job pushed onto a queue does not inherit the dispatching request's context. Put the values the job needs, such as the tenant, in the job itself and stamp them when the job runs.
- Doppar creates contexts for HTTP requests, console commands and queue jobs. Your own long-running loops need `Execution::within()`, as shown in [`Long-Running Workers`](#long-running-workers).
- A job that is stopped by force, for example by a timeout, is killed together with its child process. Anything created inside that process, scoped instances included, disappears with it, and its deferred callbacks do not run. The worker's own context for the job still ends normally when the worker records the failure.
- A context is not a place to store application data. Keep it for identity and small correlation values, and keep state in a scoped service.
- Doppar does not guess a tenant or a user for you. Metadata contains what Doppar can read from the request itself and whatever you stamp.
- Metadata values should be scalars or small arrays, because they are written to every log entry. See [`Personal Data in Logs`](#personal-data-in-logs).

## API Reference

### Execution Facade and Helper

| Method | Returns | Description |
|--------|---------|-------------|
| `Execution::current()` | `ExecutionContext\|null` | The active context |
| `Execution::running()` | `bool` | Whether any context is active |
| `Execution::id()` | `string\|null` | The active context's id |
| `Execution::type()` | `string\|null` | The active context's type |
| `Execution::get(string $key, mixed $default = null)` | `mixed` | One metadata value |
| `Execution::all()` | `array` | All metadata, inherited keys included |
| `Execution::stamp(array $metadata)` | `ExecutionContext` | Add metadata to the active context, keeping its id |
| `Execution::within(string $type, array $metadata, callable $callback)` | `mixed` | Run a callback inside a nested context |
| `Execution::beginning(callable $listener, ?string $type = null)` | `void` | Run a listener when a context begins |
| `Execution::ending(callable $listener, ?string $type = null)` | `void` | Run a listener when a context ends |
| `execution()` | `ExecutionService` | Get the execution service, the same object the facade proxies |

### Container

| Method | Returns | Description |
|--------|---------|-------------|
| `bindExecution(string $abstract, callable\|string\|null $concrete = null)` | `void` | Bind a service to the `execution` scope |
| `bindWithScope(string $abstract, callable\|string\|null $concrete, string $scope)` | `void` | Bind with `singleton`, `execution` or `transient` |
| `scopeOf(string $abstract)` | `string\|null` | The scope a binding was declared with |
| `executionContext()` | `ExecutionContext\|null` | The active context |
| `hasExecutionContext()` | `bool` | Whether any context is active |
| `beginContext(string $type, array $metadata = [])` | `ExecutionContext` | Begin a context nested in the current one |
| `endContext(ExecutionContext $context)` | `void` | End a context, and any open contexts inside it |
| `runInContext(string $type, array $metadata, callable $callback)` | `mixed` | Run a callback in a context that always ends |
| `suspendContext(ExecutionContext $context)` | `void` | Detach a context from the stack without ending it |
| `resumeContext(ExecutionContext $context)` | `void` | Make a suspended context current again |
| `stampContext(array $metadata)` | `ExecutionContext` | Add metadata to the active context |
| `contextBeginning(callable $listener, ?string $type = null)` | `void` | Register a begin listener |
| `contextEnding(callable $listener, ?string $type = null)` | `void` | Register an end listener |

### ExecutionContext

| Method | Returns | Description |
|--------|---------|-------------|
| `ExecutionContext::create(string $type, array $metadata = [], ?ExecutionContext $parent = null)` | `ExecutionContext` | Create a context, inheriting the parent's metadata |
| `id()` | `string` | The unique id |
| `type()` | `string` | The type |
| `get(string $key, mixed $default = null)` | `mixed` | One metadata value |
| `has(string $key)` | `bool` | Whether a metadata key exists |
| `all()` | `array` | All metadata |
| `parent()` | `ExecutionContext\|null` | The enclosing context |
| `depth()` | `int` | Number of ancestors |
| `startedAt()` | `float` | When the context began |
| `withMetadata(array $metadata)` | `ExecutionContext` | A copy with the same id and merged metadata |
| `toArray()` | `array` | `id`, `type`, `parent_id` and `metadata` |

The type constants `ExecutionContext::REQUEST`, `ExecutionContext::CONSOLE` and `ExecutionContext::WORKER` name the built-in types.

| Scope Constant | Value |
|----------------|-------|
| `Container::SCOPE_SINGLETON` | `singleton` |
| `Container::SCOPE_EXECUTION` | `execution` |
| `Container::SCOPE_TRANSIENT` | `transient` |
