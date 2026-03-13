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
fetched over and over — the same database query, the same API response,
the same computed result. Caching breaks that cycle. Instead of
re-executing expensive operations on every request, you compute the
result once, store it, and serve it instantly until it expires or is
invalidated.

Doppar's cache system is built on top of **Symfony's Cache component**
and implements the **PSR-16 SimpleCache** standard. This means the
entire system is driven by a clean, well-defined interface —
`Psr\SimpleCache\CacheInterface` — so your application code stays
decoupled from the underlying storage mechanism. Swap from `file` to
`redis` to `apc` in a single config line, with zero changes to your
application logic.

## Configuration

The cache system is configured in `config/caching.php`. This file
controls the default driver, all available store definitions, and the
global key prefix used to prevent collisions between applications or
environments sharing the same cache backend.

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
| `redis` | Redis (in-memory)          | Production — high throughput, shared state      |
| `array` | PHP array (request-scoped) | Testing — no persistence, no side effects       |
| `apc`   | APCu PHP extension         | Single-server deployments with shared memory    |

All drivers expose the same API. Your application never needs to know
which driver is active. You can also register a completely custom driver
— see [Custom Cache Drivers](#custom-cache-drivers) below.

---

## Clearing the Cache via CLI

To purge the entire cache store from the command line:
```bash
php pool cache:clear
```

This is useful during deployments, after configuration changes, or
whenever stale cached data needs to be discarded across the board.

## Basic Operations

Doppar exposes two equally valid ways to interact with the cache:
the `Cache` facade and the `cache()` global helper. Both resolve
the same underlying `CacheStore` instance.

### Storing an Item
Store a value for 60 seconds
```php
use Phaseolies\Support\Facades\Cache;

Cache::set('username', 'Alice', 60);
```

The third argument is the TTL — time-to-live in seconds. Pass a
`\DateInterval` instance instead of an integer if you prefer that style.
Omit the TTL entirely to store without expiration (equivalent to
`forever`).

### Retrieving an Item
```php
$username = Cache::get('username');
```

Provide a default value returned when the key is missing
```php
$username = Cache::get('username', 'Guest');
```

If the key does not exist or has expired, the default is returned.
No exception is thrown.

### Checking Existence
You can check item exists or not using `has` function like this way.
```php
if (Cache::has('username')) {
    // Key exists and has not expired
}
```

### Deleting an Item
You can delete item with `delete` function
```php
Cache::delete('username');
```

`forget()` is an alias — returns false if key was not present
```php
Cache::forget('username');
```

## Working with Multiple Items

When you need to read or write several keys in one logical operation,
the batch methods avoid the overhead of multiple round-trips to the
cache backend.

### Storing Multiple Items
```php
Cache::setMultiple([
    'config.theme'    => 'dark',
    'config.language' => 'en',
    'config.timezone' => 'Asia/Dhaka',
], ttl: 3600);
```

All keys share the same TTL. Pass `null` to store without expiration.

### Retrieving Multiple Items
```php
$values = Cache::getMultiple(
    ['config.theme', 'config.language', 'config.timezone'],
    default: 'unknown'
);
```
Returns an associative array:
```php
['config.theme' => 'dark', 'config.language' => 'en', ...]
```
>  Missing keys receive the default value


### Deleting Multiple Items
Delete multiple item using `deleteMultiple` function
```php
Cache::deleteMultiple(['config.theme', 'config.language']);
```

## Numeric Operations

For keys that hold integer values, Doppar provides atomic increment
and decrement operations. These are particularly useful for counters,
hit tracking, and lightweight rate-limiting logic.

Increment by 1 (default)
```php
Cache::increment('page_views');
```

Increment by a custom step
```php
Cache::increment('api_calls', 5);
```

Decrement by 1
```php
Cache::decrement('credits');
```

Decrement by a custom step
```php
Cache::decrement('credits', 10);
```

Both methods return the new value after the operation, or `false` if
the key does not exist. They preserve the original TTL of the item —
incrementing a counter does not reset its expiration.

## Permanent & Conditional Storage

### Store if Absent

`add()` writes the value only when the key does not already exist in
the cache. If the key is present and unexpired, the call is a no-op and
returns `false`.

Set only if 'lock_key' is not already cached
```php
$stored = Cache::add('lock_key', 'processing', ttl: 30);

if ($stored) {
    // We set it — we own this window
}
```

This is useful for lightweight advisory locks or one-time
initialization flags without the overhead of a full `AtomicLock`.

### Store Forever

`forever()` stores a value with no expiration. It will remain in the
cache until explicitly deleted or the store is cleared.
```php
Cache::forever('feature_flags', $flags);
```

Use this for data that is invalidated by an explicit event (a deploy,
an admin action) rather than by the passage of time.

## Stash Helpers

The `stash` family of methods encapsulates the most common caching
pattern — *"give me the cached value, or compute and store it if it
doesn't exist"* — in a single, expressive call. They eliminate the
check-get-set boilerplate that clutters most caching code.

### `stash()` — Cache with TTL

Retrieve the value for `key`. If it does not exist, execute the
callback, cache the result for the given TTL, and return it.
```php
$users = Cache::stash('users.all', ttl: 300, callback: fn() => User::all());
```

| Parameter  | Type                    | Description                                            |
|------------|-------------------------|--------------------------------------------------------|
| `key`      | `string`                | The cache key                                          |
| `ttl`      | `int\|DateInterval`     | How long to cache the result                           |
| `callback` | `Closure`               | Executed only on a cache miss; its return value is stored |

The callback is never executed on a cache hit — only the stored value
is returned. This is the method you will reach for in the vast majority
of caching scenarios.
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

Same as `stash()`, but the result is stored without an expiration time.
Useful for static reference data that only changes on a deliberate
admin action.
```php
$countries = Cache::stashForever(
    'ref.countries',
    fn() => Country::orderBy('name')->get()
);
```

The cache entry persists until `Cache::forget('ref.countries')` or
`Cache::clear()` is called.

### `stashWhen()` — Conditional Caching

Cache the result only when a runtime condition is true. When the
condition is false, the callback is executed and its result returned
directly — nothing is written to the cache.
```php
$results = Cache::stashWhen(
    key: 'search.' . md5($query),
    callback:  fn() => Search::run($query),
    condition: $request->wantsJson() && !$request->has('nocache'),
    ttl:       120
);
```

| Parameter   | Type                        | Description                                                |
|-------------|-----------------------------|------------------------------------------------------------|
| `key`       | `string`                    | The cache key                                              |
| `callback`  | `Closure`                   | The computation to run                                     |
| `condition` | `bool`                      | If `false`, callback runs but result is not cached          |
| `ttl`       | `int\|DateInterval\|null`   | TTL when caching; `null` stores forever when condition is true |

This is ideal for scenarios where caching should be skipped for
authenticated users, debug requests, or specific runtime flags.

## Inspecting the Active Adapter

To confirm which cache adapter is currently running — useful in
debugging or environment verification:
```php
use Phaseolies\Support\Facades\Cache;

ddd(Cache::getAdapter());
```

This returns the raw Symfony `AdapterInterface` instance. For example,
when `redis` is the active driver you will see:
```
Symfony\Component\Cache\Adapter\RedisAdapter {#...}
```

## Custom Cache Drivers

Doppar's `CacheServiceProvider` exposes an `extend()` method that lets
you register a factory closure for any custom driver name. The factory
receives the store config array and must return a Symfony
`AdapterInterface` instance.

In a ServiceProvider boot() method
```php
$cacheProvider = $this->app->make(\Phaseolies\Providers\CacheServiceProvider::class);

$cacheProvider->extend('dynamodb', function (array $config) {
    return new DynamoDbCacheAdapter(
        client: new DynamoDbClient($config),
        table:  $config['table'],
    );
});
```

Then reference it in `config/caching.php`:
```php
'stores' => [
    'dynamodb' => [
        'driver' => 'dynamodb',
        'table'  => env('DYNAMODB_CACHE_TABLE', 'cache'),
        'region' => env('AWS_DEFAULT_REGION', 'us-east-1'),
    ],
],
```

## Direct Store Instantiation

In rare cases — custom scripts, standalone tools, or package
internals — you may want to instantiate a `CacheStore` directly
without going through the service container:
```php
use Phaseolies\Cache\CacheStore;
use Symfony\Component\Cache\Adapter\FilesystemAdapter;

$adapter = new FilesystemAdapter('my_prefix_', 0, '/tmp/cache');
$store   = new CacheStore($adapter, 'my_prefix_');

$store->set('build_id', 'abc123', 3600);
$buildId = $store->get('build_id');
```

The `CacheStore` constructor accepts any `AdapterInterface`-compatible
adapter, making it trivially portable to any Symfony Cache-backed
infrastructure.

---

## Method Reference

| Method                                      | Returns        | Description                                                    |
|---------------------------------------------|----------------|----------------------------------------------------------------|
| `set(key, value, ttl?)`                     | `bool`         | Store a value with optional TTL                                |
| `get(key, default?)`                        | `mixed`        | Retrieve a value; return default on miss                       |
| `has(key)`                                  | `bool`         | Check if a non-expired key exists                              |
| `delete(key)`                               | `bool`         | Remove a key                                                   |
| `forget(key)`                               | `bool`         | Alias for `delete()`; returns `false` if key was absent        |
| `clear()`                                   | `bool`         | Purge all keys from the store                                  |
| `setMultiple(values, ttl?)`                 | `bool`         | Store multiple key-value pairs                                 |
| `getMultiple(keys, default?)`               | `iterable`     | Retrieve multiple keys                                         |
| `deleteMultiple(keys)`                      | `bool`         | Remove multiple keys                                           |
| `increment(key, value?)`                    | `int\|false`   | Increment a numeric value; preserves TTL                       |
| `decrement(key, value?)`                    | `int\|false`   | Decrement a numeric value; preserves TTL                       |
| `add(key, value, ttl?)`                     | `bool`         | Store only if key does not already exist                       |
| `forever(key, value)`                       | `bool`         | Store without expiration                                       |
| `stash(key, ttl, callback)`                 | `mixed`        | Get or compute-and-cache with TTL                              |
| `stashForever(key, callback)`               | `mixed`        | Get or compute-and-cache indefinitely                          |
| `stashWhen(key, callback, condition, ttl?)` | `mixed`        | Conditionally get or compute-and-cache                         |
| `locked(name, seconds, owner?)`             | `AtomicLock`   | Acquire an atomic lock                                         |
| `restoreLock(name, owner)`                  | `AtomicLock`   | Restore a lock by owner token                                  |
| `getAdapter()`                              | `AdapterInterface` | Return the underlying Symfony adapter instance             |