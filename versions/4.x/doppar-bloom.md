---
title: Bloom
description: Doppar bloom filter page
meta:
  - name: keywords
    content: bloom filter
---

## Bloom Filter
### Introduction
A Bloom filter is a space-efficient probabilistic data structure, conceived by Burton Howard Bloom in 1970, that is used to test whether an element is a member of a set. False positive matches are possible, but false negatives are not – in other words, a query returns either "possibly in set" or "definitely not in set". Elements can be added to the set, but not removed (though this can be addressed with a "counting" filter); the more elements that are added to the set, the larger the probability of false positives.

Doppar’s Bloom package provides a high-performance, memory-efficient probabilistic set membership test. Instead of storing every item, it keeps a compact bit array and uses hashing to answer: “Is this element possibly in the set?” (false positives possible, false negatives impossible).

In the Doppar, Bloom filters are implemented with support for both `MD5` and `Murmur` hashing algorithms, exclusively for Redis driver, providing high-performance membership testing for large-scale applications.


## Why Bloom Filter

- **Memory efficiency for large sets:** it allows you to test membership of a huge number of elements using a small memory footprint (Redis storage)
- **Speed:** checking or adding is O(k) hashing + bit operations in Redis — very fast
- **Prevent redundant work:** e.g. check before querying DB, avoid duplicate processing
- **Scalable consistency:** as the underlying storage is Redis, it works across distributed instances
- **Flexible hashing:** you can choose MD5 (stable, pure PHP) or Murmur (higher performance)
- **Simple API integration:** built cleanly into Doppar’s design, easy to plug in

## Performance Benchmarks
Doppar Bloom filters are designed for speed and memory efficiency. The following benchmarks show typical performance when using the Redis backend with the default configuration.
| **Operation**           | **100K Items** | **1M Items** | **10M Items** |
| ----------------------- | -------------- | ------------ | ------------- |
| **Memory Usage**        | 12.5 MB        | 12.5 MB      | 12.5 MB       |
| **Check Speed**         | 0.1 ms         | 0.15 ms      | 0.2 ms        |
| **False Positive Rate** | 1%             | 1%           | 1%            |

> ⚡ Even with 10 million items, memory usage stays extremely low (just ~11 MB), and lookups remain sub-millisecond. This efficiency makes Bloom filters a great choice for large-scale applications where you need fast membership checks without the executing database queries.

## Configuration
The Redis connection for Doppar Bloom is configured in the `runtime/config/caching.php` file. Don't forget to set `CACHE_DRIVER = "redis"` in your `env.toml` file. To get started with Doppar Bloom, please update your Redis connection settings accordingly.

## Installation
To get started with Doppar Bloom, use the composer package manager to add the package to your doppar project's dependencies:
```bash
composer require doppar/bloom
```

### Register Launcher
Next, register the Bloom launcher so that Doppar can initialize it properly. Open your `runtime/config/app.php` file and add the `BloomLauncher` to the providers array:
```php
'launchers' => [
    // Other launchers...
    \Doppar\Bloom\BloomLauncher::class,
],
```

## Publish Configuration
Now we need to publish the configuration files by running this pool command
```bash
php pool vendor:publish --launcher="Doppar\Bloom\BloomLauncher"
```

This will publish the `runtime/config/bloom.php` file into your application’s config directory. From there, you can update the configuration to match your requirements — such as filter size, number of hash functions, persistence driver, and hashing algorithm.

## Adding Data
You can add items to the Bloom filter using the `add()` method. Each item is hashed multiple times, and the corresponding bits are set in Redis. This allows you to quickly check if the item probably exists in the set later.
```php
use Doppar\Bloom\Facades\Bloom;

// Define emails to add
$emails = [
    'doppar@doppar.com',
    'mahedi@doppar.com',
    'aliba@doppar.com',
];

// Add each email to the Bloom filter
foreach ($emails as $email) {
    Bloom::key("email")->add($email);
}

// Add another single email
Bloom::key("email")->add('nure@doppar.com');
```

## Checking Membership
After adding items, you can check whether an item is in the filter using `has()`. This operation is very fast, but remember: it may return true for items not actually added (false positives), but it will never return false for items that were added.

```php
// Check if an email exists
if(Bloom::key("email")->has('aliba@doppar.com')){
   // true → this email was added
}

// Check an email that wasn't added
if(Bloom::key("email")->has('not_in_list@test.com')){
   // false → this email was never added
}
```

## Clearing Data
Sometimes you may want to reset the Bloom filter and start fresh. Since Bloom filters don’t support removal of individual items, the only option is to clear the entire filter.
```php
use Doppar\Bloom\Facades\Bloom;

// Clear the Bloom filter associated with "email"
Bloom::key('email')->clear();

// Now, nothing will be in the filter
dd(Bloom::key('email')->has('doppar@doppar.com')); // false
```

> 💡 Be careful: `clear()` removes all data from the filter, not just a single element.

## False Positives
A Bloom filter can sometimes return `true` even when the item was never added. This happens because multiple different values can map to the same bit positions in the array.

However, it will never return `false` for an item that was actually added.
```php
use Doppar\Bloom\Facades\Bloom;

// Add only one email
Bloom::key('email')->add('doppar@doppar.com');

// Checking a completely different email
dd(Bloom::key('email')->has('random@other.com')); 
// Output: might be true (false positive) or false
```

> 💡 The probability of false positives increases as the filter gets more “full.” You can control this trade-off by adjusting filter size and number of hash functions in configuration.