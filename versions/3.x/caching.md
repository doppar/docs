---
title: Cache
description: Doppar cache page
meta:
  - name: keywords
    content: cache, caching, redis, file cache, atomic lock, rate limiter, doppar
---

## Caching

### Introduction

Every production application reaches a point where the same data gets
fetched repeatedly: the same database query, the same API response, the
same computed payload. Caching breaks that cycle. Instead of running the
expensive work on every request, you compute the result once, store it,
and serve it until it expires or is explicitly invalidated.

Doppar's cache system is built on top of **Symfony Cache** and exposes a
PSR-16-compatible store through `Psr\SimpleCache\CacheInterface`. On top
of that base contract, Doppar adds convenience methods such as
`increment`, `decrement`, `add`, `stash`, atomic locks, and a dedicated
rate limiter.

## Configuration

The cache system is configured in `config/caching.php`. This file
controls the default driver, all available store definitions, and the
global key prefix used to prevent collisions between applications or
environments sharing the same backend.

Set your preferred driver in `.env`:
```bash
CACHE_DRIVER=redis
CACHE_PREFIX=myapp_
```

## Supported Drivers

Doppar ships with four first-class cache backends out of the box:

| Driver  | Backed By                  | Best For                                        |
|---------|----------------------------|-------------------------------------------------|
| `file`  | Filesystem                 | Simple deployments, local development           |
| `redis` | Redis (in-memory)          | Production, shared state, high throughput       |
| `array` | PHP array (request-scoped) | Testing, temporary in-memory usage              |
| `apc`   | APCu PHP extension         | Single-server shared memory                     |

All drivers expose the same high-level API. Your application code does
not need to change when you switch from one backend to another.

## Clearing the Cache via CLI

To clear the current application's cache namespace from the command
line:
```bash
php pool cache:clear
```

This is useful during deployments, after configuration changes, or
whenever stale cached data should be discarded.

## Basic Operations

Doppar exposes two equally valid ways to interact with the cache:
the `Cache` facade and the `cache()` global helper. Both resolve the
same underlying `CacheStore` instance.

In the following examples, you can use either style depending on your
preference:

```php
cache()->set('username', 'Alice', 60);
```

Or using the `Phaseolies\Support\Facades\Cache` facades
```php
use Phaseolies\Support\Facades\Cache;

Cache::set('username', 'Alice', 60);
```

### The `cache()` Helper

Use the global helper when you want a concise way to resolve the store,
read a key, or write one or more values.

Resolve the cache store instance:
```php
$store = cache();
```

You can also call cache methods directly on the resolved store:
```php
cache()->set('username', 'Alice', 60);

$username = cache()->get('username', 'Guest');
```

Retrieve a value with an optional default:
```php
$username = cache('username', 'Guest');
```

Store one or more values with an optional TTL:
```php
cache(['username' => 'Alice'], 60);

cache([
    'settings.theme' => 'dark',
    'settings.locale' => 'en',
], 300);
```

### Storing an Item

Store a single value in the cache when you already know the key and the
time it should remain available.

If you prefer the facade style, import `Cache` and call its methods
directly:
```php
use Phaseolies\Support\Facades\Cache;

Cache::set('username', 'Alice', 60);
```

The third argument is the TTL in seconds. You may also pass a
`\DateInterval` instance.

If you omit the TTL, the active adapter's default lifetime is used.
For truly indefinite storage, use `forever()`.

### Retrieving an Item

Retrieve a cached value by key. If the key exists and has not expired,
the stored value is returned immediately.

```php
$username = Cache::get('username');
```

Provide a default value when the key is missing:
```php
$username = Cache::get('username', 'Guest');
$username = cache('username', 'Guest');
$username = cache()->get('username', 'Guest');
```

If the key does not exist or has expired, the default is returned.

### Checking Existence

Use `has()` when you only need to know whether a non-expired cache key
is present.

```php
if (Cache::has('username')) {
    // Key exists and has not expired
}
```

### Deleting an Item

Delete a cached value when you need to invalidate a single key
manually.

```php
Cache::delete('username');
```

`forget()` is an alias that returns `false` when the key was not
present:
```php
Cache::forget('username');
```

## Working with Multiple Items

When you need to read or write several keys in one logical operation,
the batch methods keep the code compact and backend-friendly.

### Storing Multiple Items

Store several related values in a single logical operation when they all
share the same TTL.

```php
Cache::setMultiple([
    'config.theme'    => 'dark',
    'config.language' => 'en',
    'config.timezone' => 'Asia/Dhaka',
], ttl: 3600);

cache([
    'config.theme'    => 'dark',
    'config.language' => 'en',
    'config.timezone' => 'Asia/Dhaka',
], 3600);
```

All keys share the same TTL. Pass `null` to use the adapter default
lifetime.

### Retrieving Multiple Items

Read multiple keys at once when you want a grouped result array instead
of separate `get()` calls.

```php
$values = Cache::getMultiple(
    ['config.theme', 'config.language', 'config.timezone'],
    default: 'unknown'
);
```

Returns an associative array:
```php
[
    'config.theme' => 'dark',
    'config.language' => 'en',
    'config.timezone' => 'Asia/Dhaka',
]
```

Missing keys receive the provided default value.

### Deleting Multiple Items

Remove several cached keys together when you want to invalidate a whole
group of values.

```php
Cache::deleteMultiple(['config.theme', 'config.language']);
```

## Numeric Operations

For keys that hold integer values, Doppar provides increment and
decrement operations.

Increment by 1:
```php
Cache::increment('page_views');
```

Increment by a custom step:
```php
Cache::increment('api_calls', 5);
```

Decrement by 1:
```php
Cache::decrement('credits');
```

Decrement by a custom step:
```php
Cache::decrement('credits', 10);
```

Both methods return the new value, or `false` if the key does not
exist. They preserve the original expiration time.

## Permanent & Conditional Storage

### Store if Absent

Use `add()` when the value should only be written if the key is not
already present.

`add()` stores a value only if the key does not already exist:
```php
$stored = Cache::add('lock_key', 'processing', ttl: 30);

if ($stored) {
    // Value was written
}
```

### Store Forever

Use `forever()` for values that should remain cached until you delete
them explicitly.

`forever()` stores a value without expiration:
```php
Cache::forever('feature_flags', $flags);
```

The value remains until you explicitly delete it or clear the cache
namespace.

## Stash Helpers

The `stash` family wraps the common pattern of "get the cached value or
compute and store it on a miss".

### `stash()` — Cache with TTL

Use `stash()` when you want Doppar to compute a value only on a cache
miss and remember it for a fixed amount of time.

```php
$users = Cache::stash('users.all', ttl: 300, callback: fn() => User::all());
```

| Parameter  | Type                | Description                          |
|------------|---------------------|--------------------------------------|
| `key`      | `string`            | The cache key                        |
| `ttl`      | `int\|DateInterval` | How long to cache the computed value |
| `callback` | `Closure`           | Executed only on a cache miss        |

Example:
```php
#[Route(uri: 'products', methods: ['GET'])]
public function index(): mixed
{
    $products = Cache::stash(
        'products.active',
        ttl: 600,
        callback: fn() => Product::where('active', true)->get()
    );

    return response()->json($products);
}
```

### `stashForever()` — Cache Indefinitely

Use `stashForever()` when the cached value should be computed once and
kept until you explicitly invalidate it.

```php
$countries = Cache::stashForever(
    'ref.countries',
    fn() => Country::orderBy('name')->get()
);
```

### `stashWhen()` — Conditional Caching

Use `stashWhen()` when whether a value should be cached depends on a
runtime condition.

Cache only when a condition is true:
```php
$results = Cache::stashWhen(
    key: 'search.' . md5($query),
    callback: fn() => Search::run($query),
    condition: $request->wantsJson() && !$request->has('nocache'),
    ttl: 120
);
```

When `condition` is `false`, the callback still runs, but nothing is
written to cache.

## Atomic Locks

For cross-request coordination, Doppar provides an atomic lock API on
top of the cache store. It ensures that a specific resource or process is accessed by only one execution thread (or request) at a time — preventing race conditions, inconsistent states, and data corruption.

Atomic locks are built on doppar default caching abstraction, which integrates with Symfony’s cache adapters. This allows locks to be distributed across multiple servers while remaining efficient and consistent.

In simpler terms, Atomic Lock ensures that only one worker can perform a specific task at a given time — all others must wait or fail gracefully.

### Create and Acquire a Lock

Create a lock when a block of code must run in only one request, worker,
or process at a time.

```php
$lock = Cache::locked('reports:daily', 30);

if ($lock->get()) {
    try {
        // Critical section...
    } finally {
        $lock->release();
    }
}
```

The second argument is the lock lifetime in seconds.

### Block Until Available

Use `block()` when you are willing to wait for a lock instead of failing
immediately.

```php
$lock = Cache::locked('reports:daily', 30);

if ($lock->block(5)) {
    try {
        // Lock acquired within 5 seconds
    } finally {
        $lock->release();
    }
}
```

### Restore a Lock by Owner Token

Restore a lock when you need to pass lock ownership between different
parts of your application or across process boundaries.

```php
$lock = Cache::locked('reports:daily', 30);

if ($lock->get()) {
    $owner = $lock->getOwner();
}

$restored = Cache::restoreLock('reports:daily', $owner);

if ($restored->isOwnedByCurrentProcess()) {
    $restored->release();
}
```

### Lock Inspection Helpers

These helper methods let you inspect the current lock state and its
ownership details.

```php
$lock->owner();
$lock->getOwner();
$lock->getName();
$lock->getSeconds();
$lock->isOwned();
$lock->isOwnedByCurrentProcess();
$lock->isRestored();
$lock->getRemainingTime();
```

Disable auto-release if you need to keep the lock alive beyond object
destruction:
```php
$lock->preventRelease();

// Later...
$lock->allowRelease();
```

## Rate Limiting

Doppar ships with a dedicated `RateLimiter` service and a `throttle()`
global helper:

```php
$limiter = throttle();
```

### Record an Attempt and Inspect the Limit

Use `attempt()` when you want to increment a limiter and immediately
inspect the remaining capacity in the current window.

```php
$limit = throttle()->attempt('login:' . request()->ip(), 5, 60);

if ($limit->remaining === 0) {
    // No attempts left
}
```

The returned `RateLimit` object exposes:

| Property    | Type  | Meaning                              |
|-------------|-------|--------------------------------------|
| `limit`     | `int` | Maximum allowed attempts             |
| `remaining` | `int` | Attempts left in the current window  |
| `resetAt`   | `int` | Unix timestamp for reset             |

### Increment Without Returning a `RateLimit`

Use `hit()` when you only need to increment the counter and retrieve the
current attempt count.

```php
$attempts = throttle()->hit('password-reset:' . request()->ip(), 300);
```

### Inspect and Reset Limiter State

Use these methods to inspect the current limiter state or clear it when
your application needs to reset the window manually.

```php
if (throttle()->tooManyAttempts('api:' . request()->ip(), 100)) {
    $retryAfter = throttle()->availableIn('api:' . request()->ip());
}

$attempts = throttle()->attempts('api:' . request()->ip());

throttle()->resetAttempts('api:' . request()->ip());
throttle()->clear('api:' . request()->ip()); // removes counter and timer
```

### Built-in Middleware

If you prefer middleware-driven throttling, Doppar includes a request
throttling middleware built on the same limiter service.

The `ThrottleRequests` middleware uses this limiter internally and adds
`X-RateLimit-*` headers to the response.

## Inspecting the Active Adapter

To confirm which cache adapter is currently active:
```php
use Phaseolies\Support\Facades\Cache;

ddd(Cache::getAdapter());
```

Example output:
```text
Symfony\Component\Cache\Adapter\RedisAdapter {#...}
```

## Notes on Clear Behavior

`Cache::clear()` clears the keys owned by the current cache
prefix/namespace. It does not flush every key from a shared Redis
database or shared filesystem cache root.

## Custom Cache Drivers

Doppar's `CacheServiceProvider` exposes an `extend()` method for custom
drivers. The factory closure receives the store config array and must
return a Symfony `AdapterInterface`.

In a service provider:
```php
$cacheProvider = $this->app->make(\Phaseolies\Providers\CacheServiceProvider::class);

$cacheProvider->extend('dynamodb', function (array $config) {
    return new DynamoDbCacheAdapter(
        client: new DynamoDbClient($config),
        table: $config['table'],
    );
});
```

Then reference it in `config/caching.php`:
```php
'stores' => [
    'dynamodb' => [
        'driver' => 'dynamodb',
        'table' => env('DYNAMODB_CACHE_TABLE', 'cache'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],
],
```

## Direct Store Instantiation

For custom scripts or package internals, you can instantiate
`CacheStore` directly:
```php
use Phaseolies\Cache\CacheStore;
use Symfony\Component\Cache\Adapter\FilesystemAdapter;

$adapter = new FilesystemAdapter('my_prefix_', 0, '/tmp/cache');
$store = new CacheStore($adapter, 'my_prefix_');

$store->set('build_id', 'abc123', 3600);
$buildId = $store->get('build_id');
```

## Method Reference

### Cache Store

These are the main methods exposed by Doppar's cache store and cache
facade.

| Method                                      | Returns              | Description                                             |
|---------------------------------------------|----------------------|---------------------------------------------------------|
| `set(key, value, ttl?)`                     | `bool`               | Store a value with optional TTL                         |
| `get(key, default?)`                        | `mixed`              | Retrieve a value; return default on miss                |
| `has(key)`                                  | `bool`               | Check if a non-expired key exists                       |
| `delete(key)`                               | `bool`               | Remove a key                                            |
| `forget(key)`                               | `bool`               | Alias for `delete()`                                    |
| `clear()`                                   | `bool`               | Clear keys for the current cache namespace/prefix       |
| `setMultiple(values, ttl?)`                 | `bool`               | Store multiple key-value pairs                          |
| `getMultiple(keys, default?)`               | `iterable`           | Retrieve multiple keys                                  |
| `deleteMultiple(keys)`                      | `bool`               | Remove multiple keys                                    |
| `increment(key, value?)`                    | `int\|false`         | Increment a numeric value; preserves TTL                |
| `decrement(key, value?)`                    | `int\|false`         | Decrement a numeric value; preserves TTL                |
| `add(key, value, ttl?)`                     | `bool`               | Store only if key does not already exist                |
| `forever(key, value)`                       | `bool`               | Store without expiration                                |
| `stash(key, ttl, callback)`                 | `mixed`              | Get or compute-and-cache with TTL                       |
| `stashForever(key, callback)`               | `mixed`              | Get or compute-and-cache indefinitely                   |
| `stashWhen(key, callback, condition, ttl?)` | `mixed`              | Conditionally get or compute-and-cache                  |
| `locked(name, seconds, owner?)`             | `AtomicLock`         | Create a lock instance                                  |
| `restoreLock(name, owner)`                  | `AtomicLock`         | Restore a lock by owner token                           |
| `getAdapter()`                              | `AdapterInterface`   | Return the underlying Symfony adapter                   |

### Global Helpers

These helpers provide a lightweight way to work with the cache store and
rate limiter.

| Helper                    | Returns            | Description                                           |
|---------------------------|--------------------|-------------------------------------------------------|
| `cache()`                 | `CacheStore`       | Resolve the cache store                               |
| `cache()->method(...)`    | `mixed`            | Call cache store methods directly on the resolved object |
| `cache('key', default?)`  | `mixed`            | Retrieve a cached value                               |
| `cache(['k' => 'v'], ttl?)` | `bool`           | Store one or more values                              |
| `throttle()`              | `RateLimiter`      | Resolve the rate limiter service                      |

### Atomic Lock

These methods are available on the `AtomicLock` instance returned by
`Cache::locked()`.

| Method                     | Returns          | Description                                           |
|----------------------------|------------------|-------------------------------------------------------|
| `get()`                    | `bool`           | Attempt to acquire the lock immediately               |
| `block(seconds)`           | `bool`           | Wait for the lock to become available                 |
| `release()`                | `bool`           | Release an owned lock                                 |
| `preventRelease()`         | `AtomicLock`     | Disable automatic release on destruction              |
| `allowRelease()`           | `AtomicLock`     | Re-enable automatic release                           |
| `owner()`                  | `string`         | Get the current owner token from cache                |
| `getOwner()`               | `string`         | Get this instance's owner token                       |
| `getName()`                | `string`         | Get the lock name                                     |
| `getSeconds()`             | `int`            | Get the configured lock lifetime                      |
| `isOwned()`                | `bool`           | Check whether this instance is marked as owner        |
| `isOwnedByCurrentProcess()`| `bool`           | Verify cached ownership against this instance         |
| `isRestored()`             | `bool`           | Check whether the lock came from `restoreLock()`      |
| `getRemainingTime()`       | `int\|null`      | Seconds until lock expiry                             |

### Rate Limiter

These methods are available on the `RateLimiter` service returned by
the `throttle()` helper.

| Method                                    | Returns        | Description                                           |
|-------------------------------------------|----------------|-------------------------------------------------------|
| `attempt(key, maxAttempts, decaySeconds)` | `RateLimit`    | Record an attempt and return current limit state      |
| `hit(key, decaySeconds)`                  | `int`          | Increment a limiter counter                           |
| `tooManyAttempts(key, maxAttempts)`       | `bool`         | Check whether a key is at or above the limit          |
| `attempts(key)`                           | `int`          | Get the current number of attempts                    |
| `availableIn(key)`                        | `int`          | Seconds until the limiter window resets               |
| `availableAt(seconds)`                    | `int`          | Unix timestamp for a future reset time                |
| `resetAttempts(key)`                      | `void`         | Delete only the attempt counter                       |
| `clear(key)`                              | `void`         | Delete both the attempt counter and timer             |
