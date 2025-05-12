## Caching
### Introduction
Caching is a powerful technique that stores frequently accessed data in a temporary storage layer, reducing the need to repeatedly perform expensive operations (like database queries or API calls). By using caching in your application, you can significantly improve performance and reduce server load.

The Doppar PHP framework provides a flexible caching system that supports multiple drivers and implements the PSR-16 SimpleCache standard for interoperability.

### Configuration
The cache system is configured via the `config/caching.php` file. This file allows you to define The default cache driver,
Available cache stores , a global key prefix to prevent collisions etc. Before starting with cache, configure your cache driver.

### Supported Cache Drivers
Doppar supports multiple cache backends, allowing you to choose the one that best fits your application's performance and scalability needs:
| **Driver** | **Description**                                               |
| ---------- | ------------------------------------------------------------- |
| `file`     | Stores cached data as files on disk                           |
| `redis`    | Uses Redis for fast, in-memory caching                        |
| `array`    | Stores data in memory during the request (useful for testing) |
| `apc`      | Utilizes the APCu PHP extension for memory-based caching      |

Each driver can be configured in the `config/caching.php` file. You can switch drivers without changing your application logic, offering great flexibility across environments.

> 🛠️ You can define custom adapters and drivers to suit your infrastructure needs.

Doppar provides a pool command to completely delete system cache like
```bash
php pool cache:clear
```

### Basic Usage
####  Using the Cache Facade
Doppar provides a convenient caching facade to make working with the cache system simple and expressive within your controllers, services, or any part of your application.
```php
use Phaseolies\Support\Facades\Cache;

Cache::set('name', 'Mahedi', 60); // Stored for 60 seconds
```

Retrieve an item with a default fallback:
```php
$name = Cache::get('name', 'Default Value');
```

You can also check if a key exists in the cache:
```php
$exists = Cache::has('name');
```
To delete a cached item, use the delete method on the Cache facade:
```php
use Phaseolies\Support\Facades\Cache;

Cache::delete('name');
```

### Working with Multiple Items
In many real-world scenarios, you may need to interact with several cache keys at once. Doppar’s caching facade provides convenient methods to handle multiple items efficiently. This includes setting, retrieving, and deleting groups of cached values in a single operation:
```php
// Store multiple items with a 60-second TTL
Cache::setMultiple([
    'key1' => 'value1',
    'key2' => 'value2'
], 60);

// Retrieve multiple items with a default fallback value
$values = Cache::getMultiple(['key1', 'key2'], 'default');

// Remove multiple cached items at once
Cache::deleteMultiple(['key1', 'key2']);
```

### Advanced Cache Operations
For more advanced use cases, Doppar provides several utility methods that allow you to manipulate cache values directly, manage permanent items, and perform atomic operations. These features are especially helpful for counters, configuration flags, and conditional caching logic.

### Increment Cached Values
You can increment or decrement numeric values stored in the cache. This is useful for counters, tracking hits, or managing rate limits.
```php
// Increment or decrement cache values
Cache::increment('counter', 2);
Cache::decrement('counter');
```

### Conditional & Permanent Caching
Doppar provides methods for conditionally setting cache items and storing data without an expiration time.
```php
// Only set the key if it doesn't already exist
Cache::add('user_id_123', 'John Doe', 120);

// Store an item forever (no expiration)
Cache::forever('config_loaded', true);
```
These methods are helpful for one-time initialization or persistent configuration flags.


### Clearing the Cache
You can clear the entire cache store when needed, for example during deployments, configuration updates, or testing.
```php
// Completely clear the cache
Cache::clear();
```

### Check Running Cache Adapter
To determine which cache adapter (driver) is currently active in your Doppar application, you can use the `getAdapter()` method provided by the Cache facade. This is useful for debugging or confirming the environment's cache setup.
```php
use Phaseolies\Support\Facades\Cache;

dd(Cache::getAdapter());
```
This will dump the name of the currently configured cache driver (e.g., file, redis, apc, etc.). If the driver is redis, you will see `Symfony\Component\Cache\Adapter\RedisAdapter` returnd.

### Custom Cache Store Implementation
Under the hood, Doppar uses the `\Phaseolies\Cache\CacheStore` class which implements the PSR `CacheInterface` and works with `Symfony’s AdapterInterface`. This design allows you to easily switch or extend cache drivers while maintaining a consistent API.

```php
$store = new \Phaseolies\Cache\CacheStore($adapter, 'my_prefix_');
$store->set('custom_key', 'value', 300);
```

### Fetch and Store With TTL
To simplify caching logic and improve code readability, we've introduced expressive helpers: `stash`, `stashForever`, and `stashWhen`. These methods provide a clean, fluent way to cache data with fallbacks, time-to-live (TTL), and conditional storage.

Each method wraps common cache usage patterns in a more descriptive and intuitive form—making your codebase easier to maintain and understand.

Use stash() to retrieve a cached item by key. If it doesn’t exist, the given callback is executed, its result is cached for the specified TTL, and returned. Ideal for data that changes periodically but doesn't need to be fetched on every request.
```php
$users = Cache::stash('users_collection', 60, function () {
    return User::all()->toArray();
});
```
Parameters:
- key (string): The cache key used to store the data.
- ttl (int|DateInterval): Duration in seconds or interval the cache should be kept.
- callback (Closure): A function that returns the value to cache if not already stored.

Returns: The cached value

> Remember: Cache will not work with Eloquent collection, you need to convert the collection to array, otherwise you will face unexpected response.

### ### Fetch and Store Forever
Use stashForever() when you want to cache a value indefinitely, until it is manually cleared. This is perfect for rarely changing or configuration-style data where expiration is unnecessary.
```php
$users = Cache::stashForever('users_collection', function () {
    return User::all()->toArray();
});
```
No expiration is set; data remains in the cache until cleared.

### Conditional Caching
The stashWhen() method is a powerful helper for caching data only when certain conditions are met. It gives you control over when caching should occur, rather than always caching by default.

This is especially useful when caching should depend on runtime flags, environment settings, or request context.
```php
$result = Cache::stashWhen(
    'user_'.$userId,
    fn() => User::query()->newest()->get()->toArray(),
    $shouldCache,  // boolean condition
    3600          // optional TTL
);
```
Perfect for situations where caching should only happen under specific conditions.

