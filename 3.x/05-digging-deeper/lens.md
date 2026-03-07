---
title: Lens
description: Doppar Lens page
meta:
  - name: keywords
    content: Lens
---

## Lens
### Introduction
The `Phaseolies\Support\Lens` utility is a powerful and expressive helper for working with PHP arrays. Inspired by functional programming principles and dot notation access patterns, it provides a clean, chainable, and readable way to access, modify, flatten, and filter arrays — including deeply nested structures.

Whether you're working with configuration data, nested API responses, or transforming data collections, `Lens` simplifies your logic and keeps your code expressive.

## Lens Usage
Lens are designed to offer a clean and expressive API to work with arrays or data objects in Doppar. Below are some common methods and how to use them effectively with the collect() helper.

### `grab()`
The `grab` method allows you to retrieve a value from an array using "dot" notation. This means you can easily access nested array values by specifying keys separated by dots.

If the specified key does not exist, it returns a default value.
```php
use Phaseolies\Support\Lens;

$data = [
    'user' => [
        'name' => 'John',
        'email' => 'john@example.com',
        'profile' => [
            'age' => 30,
            'city' => 'New York'
        ]
    ]
];

$name = Lens::grab($data, 'user.name'); // Returns 'John'
$age = Lens::grab($data, 'user.profile.age'); // Returns 30

// Return default if key not found
$country = Lens::grab($data, 'user.profile.country', 'Unknown');

// Returns 'Unknown'

// If key is null, returns entire array
$all = Lens::grab($data, null); // Returns the whole $data array
```

### `put()`
The put method sets a value in an array using "dot" notation. This lets you easily assign values deep inside nested arrays by specifying keys separated by dots. The array is modified by reference.

If intermediate keys don’t exist or aren’t arrays, they will be created as empty arrays automatically.
```php
use Phaseolies\Support\Lens;

$data = [
    'user' => [
        'name' => 'John',
        'profile' => [
            'age' => 30
        ]
    ]
];

// Set a nested value using dot notation
Lens::put($data, 'user.profile.city', 'New York');

// $data now becomes:
[
    'user' => [
        'name' => 'John',
        'profile' => [
            'age' => 30,
            'city' => 'New York'
        ]
    ]
];

// You can also create new nested keys on the fly
Lens::put($data, 'settings.theme.color', 'dark');

// $data now contains:
[
    'user' => [ ... ],
    'settings' => [
        'theme' => [
            'color' => 'dark'
        ]
    ]
];
```

### `got()`
The got method checks if one or more keys exist in an array using "dot" notation. It returns true only if all specified keys exist in the array (including nested keys).

You can pass a single key as a string or multiple keys as an array.
```php
use Phaseolies\Support\Lens;

$data = [
    'user' => [
        'name' => 'John',
        'profile' => [
            'age' => 30,
            'city' => 'New York'
        ]
    ],
    'settings' => [
        'theme' => 'dark'
    ]
];

// Check if a single nested key exists
Lens::got($data, 'user.name'); // true

// Check multiple keys at once
Lens::got($data, ['user.name', 'user.profile.age']); // true

// If any key does not exist, returns false
Lens::got($data, ['user.name', 'user.profile.country']);
// false
```

### `some()`
The `some` method checks if at least one of the given keys exists in an array using "dot" notation. It returns true if any one of the specified keys is found; otherwise, it returns false.

You can pass a single key as a string or multiple keys as an array.
```php
$data = [
    'user' => [
        'name' => 'John',
        'profile' => [
            'age' => 30,
            'city' => 'New York'
        ]
    ],
    'settings' => [
        'theme' => 'dark'
    ]
];

// Check if at least one key exists
$hasOne = Lens::some($data, ['user.email', 'user.name']);
// true (because 'user.name' exists)

// If none exist, returns false
$hasNone = Lens::some($data, ['user.email', 'user.address']); // false
```

### `zap()`
The `zap` method removes one or more items from an array using "dot" notation. It modifies the array by reference, so the original array is updated directly.

You can remove deeply nested keys, and pass either a single key or an array of keys.
```php
$data = [
    'user' => [
        'name' => 'John',
        'email' => 'john@example.com',
        'profile' => [
            'age' => 30,
            'city' => 'New York'
        ]
    ],
    'settings' => [
        'theme' => 'dark',
        'notifications' => true
    ]
];

// Remove a nested key
Lens::zap($data, 'user.profile.city');

// Remove multiple keys at once
Lens::zap($data, ['user.email', 'settings.notifications']);

// $data now looks like:
[
    'user' => [
        'name' => 'John',
        'profile' => [
            'age' => 30
        ]
    ],
    'settings' => [
        'theme' => 'dark'
    ]
];
```

### `pick()`
The `pick` method extracts values from an array of arrays or objects using "dot" notation. It's similar to `array_column()` but more powerful—it supports nested keys and optional custom keys for the result.

You can:
- Just pluck values
- Optionally define a key to index the result by
```php
$users = [
    ['id' => 1, 'profile' => ['name' => 'Alice']],
    ['id' => 2, 'profile' => ['name' => 'Bob']],
    ['id' => 3, 'profile' => ['name' => 'Charlie']],
];

// Pluck just the names
$names = Lens::pick($users, 'profile.name');
// Result: ['Alice', 'Bob', 'Charlie']

// Pluck names using `id` as the key
$namesById = Lens::pick($users, 'profile.name', 'id');
// Result: [1 => 'Alice', 2 => 'Bob', 3 => 'Charlie']
```

### `flat()`
The `flat` method flattens a multi-dimensional array into a single-level array. You can control the depth of flattening—by default, it flattens all levels recursively.

If an item is not an array or the max depth has been reached, it is added as-is.
```php
$data = [
    [1, 2],
    [3, [4, 5]],
    6
];

// Flatten completely (default)
$flat = Lens::flat($data);
// Result: [1, 2, 3, 4, 5, 6]

// Flatten only one level deep
$flatOnce = Lens::flat($data, 1);
// Result: [1, 2, 3, [4, 5], 6]
```

This method is helpful when dealing with deeply nested arrays where you want to reduce complexity or extract values into a simple list.

### `head()`
The `head` method returns the first element of an array, with optional support for filtering through a callback. If no match is found, it returns a default value (which is null by default).

Features:
- Get the first item in a plain array
- Or: Get the first item that matches a condition using a callback
- Return a fallback value if nothing is found
```php
$numbers = [10, 20, 30];

// Get first element
$first = Lens::head($numbers);
// Result: 10

// With filter callback
$firstOver15 = Lens::head($numbers, fn($n) => $n > 15);
// Result: 20

// When nothing matches, return default
$none = Lens::head($numbers, fn($n) => $n > 100, 'Not found');
// Result: 'Not found'
```

### `tail()`
The `tail` method returns the last element of an array, optionally filtered through a callback. If no match is found, it returns a default value (which is null by default).

Features:
- Get the last element from an array
- Or: Get the last element that satisfies a given condition
- Return a default if no match is found

```php
$items = ['a', 'b', 'c'];

// Get last element
$last = Lens::tail($items);
// Result: 'c'

// With filter
$lastB = Lens::tail($items, fn($v) => $v === 'b');
// Result: 'b'

// With no match, return default
$missing = Lens::tail($items, fn($v) => $v === 'x', 'Not found');
// Result: 'Not found'
```
This method is handy when you're looking for the last matching value in an array or simply want the last item safely.

### `squash()`
The `squash` method performs a shallow flatten on a multi-dimensional array. It only removes one level of nesting, unlike `flat()` which can flatten deeply.

Only values that are arrays will be merged into the result; non-array values are ignored.
```php
$data = [
    [1, 2],
    [3, 4],
    5
];

// Shallow flatten
$flat = Lens::squash($data);
// Result: [1, 2, 3, 4]

// Notice: non-array values like `5` are skipped
```

This is useful when you just want to collapse an array of arrays by one level without affecting deeper structures.

### `keep()`
The `keep` method returns a new array containing only the specified keys from the original array. It filters out everything else, preserving only the keys you want.

If a key doesn't exist in the source array, it's simply skipped.
```php
$data = [
    'id' => 1,
    'name' => 'Alice',
    'email' => 'alice@example.com',
    'role' => 'admin'
];

// Keep only 'id' and 'name'
$filtered = Lens::keep($data, ['id', 'name']);
// Result: ['id' => 1, 'name' => 'Alice']

// Missing keys are ignored
$partial = Lens::keep($data, ['name', 'nonexistent']);
// Result: ['name' => 'Alice']
```

This method is useful for whitelisting specific keys in a payload, config, or input array.

### `drop()`
The `drop` method removes the specified keys from an array and returns the filtered result. It is essentially the inverse of `keep()`.

If a key doesn't exist in the array, it’s ignored silently.
```php
$data = [
    'id' => 1,
    'name' => 'Alice',
    'email' => 'alice@example.com',
    'role' => 'admin'
];

// Drop 'email' and 'role'
$filtered = Lens::drop($data, ['email', 'role']);
// Result: ['id' => 1, 'name' => 'Alice']

// Dropping a non-existent key does nothing
$partial = Lens::drop($data, ['nonexistent']);
// Result: original array unchanged
```
This is helpful when you want to blacklist certain keys from being included in the final output.

### `assoc()`
The `assoc` method checks if an array is associative, meaning it has non-sequential or string keys. It returns true if the array is associative, and false if it's a standard, numerically indexed (sequential) array.

An empty array is considered not associative.
```php
// Sequential array
$data1 = ['a', 'b', 'c'];
$isAssoc1 = Lens::assoc($data1);
// Result: false

// Associative array
$data2 = ['name' => 'Alice', 'age' => 30];
$isAssoc2 = Lens::assoc($data2);
// Result: true

// Mixed keys
$data3 = [0 => 'a', 2 => 'b', 3 => 'c'];
$isAssoc3 = Lens::assoc($data3);
// Result: true

// Empty array
$data4 = [];
$isAssoc4 = Lens::assoc($data4);
// Result: false
```

### `whr()`
The `whr` method filters an array using a custom callback, returning only the items for which the callback returns true. It’s similar to array_filter(), but the callback receives both the value and the key, and all matching keys are preserved in the result.
```php
$data = [
    'a' => 10,
    'b' => 25,
    'c' => 5,
    'd' => 30
];

// Filter items greater than 20
$result = Lens::whr($data, fn($value) => $value > 20);
// Result: ['b' => 25, 'd' => 30]

// You can also filter using the key
$startsWithB = Lens::whr($data, fn($v, $k) => $k === 'b');
// Result: ['b' => 25]
```

This method is great when you need fine-grained control over filtering, including conditions based on keys.

### `wrap()`
The `wrap` method ensures the given value is an array. If the value is already an array, it returns it as-is. If the value is null, it returns an empty array. Otherwise, it wraps the value inside a new array.
```php
Lens::wrap(5);        // Returns [5]
Lens::wrap([1, 2]);   // Returns [1, 2]
Lens::wrap(null);     // Returns []
Lens::wrap('hello');  // Returns ['hello']
```
Use this method to normalize inputs, especially when you want to treat single values and arrays uniformly.

### `dot()`
The `dot` method flattens a multi-dimensional associative array into a single-level array using dot notation for nested keys.

- It recursively traverses the array.
- Nested keys are concatenated with dots (.).
- Only associative arrays are recursively flattened; sequential arrays are treated as values.
```php
$array = [
    'user' => [
        'name' => 'Alice',
        'profile' => [
            'age' => 25,
            'city' => 'Boston'
        ]
    ],
    'active' => true
];

$flattened = Lens::dot($array);
```
Result:
```php
[
    'user.name' => 'Alice',
    'user.profile.age' => 25,
    'user.profile.city' => 'Boston',
    'active' => true
]
```

### `undot()`
The undot method converts a flattened array using dot notation back into a nested associative array.

- It iterates over each dot-notated key in the flat array.
- It uses the previously defined put() method to set the nested values accordingly.
- This reconstructs the original multi-dimensional array structure.
```php
$flattened = [
    'user.name' => 'Alice',
    'user.profile.age' => 25,
    'user.profile.city' => 'Boston',
    'active' => true
];

$nested = Lens::undot($flattened);
```
Result:
```php
[
    'user' => [
        'name' => 'Alice',
        'profile' => [
            'age' => 25,
            'city' => 'Boston'
        ]
    ],
    'active' => true
]
```
Use this method to restore nested arrays from flat representations, such as those used in configuration files or database storage.

### `rand()`
The `rand` method returns a new array with the values shuffled in random order.
```php
$items = [1, 2, 3, 4, 5];
$shuffled = Lens::rand($items);
// $shuffled might be [3, 1, 5, 2, 4] or any other random permutation
```
Use this method when you want a random order of the array elements without modifying the original.