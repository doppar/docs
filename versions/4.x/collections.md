---
title: Collections
description: Doppar Collections page
meta:
  - name: keywords
    content: Collections
---

## Collections
### Introduction
Doppar provides a convenient and fluent Collection class built on top of the powerful `ramsey/collection` package. Collections are an elegant and flexible way to work with arrays of data, especially when you want to perform transformations, filtering, grouping, and other operations in a chainable and expressive syntax.

In Doppar, the Collection class enhances `Ramsey\Collection\Collection` with additional utility methods and integrations tailored for Doppar's ecosystem, including Entity model support and developer-friendly features like memory usage tracking and deep flattening. To make it easy to work with collections, Doppar provides the global `collect()` helper function:

## Collection Usage
Collections are designed to offer a clean and expressive API to work with arrays or data objects in Doppar. Below are some common methods and how to use them effectively with the `collect()` helper.

### `count()`
The `count()` method returns the total number of items in the collection. This is especially useful when you need to quickly determine the size of a dataset, such as the number of records returned from a query or the number of items in a transformed list.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
    ['id' => 3, 'name' => 'Charlie'],
]);

echo $users->count(); // Output: 3
```

This method internally uses PHP’s native count() function on the collection's internal data array, making it efficient and reliable.

> Tip: Since the Collection class implements Countable, you can also use PHP’s native `count($collection)` function directly:
```php
echo count($users); // Output: 3
```

### `isEmpty()`
The `isEmpty()` method determines whether the collection contains no items. It returns true if the collection has zero elements and `false` otherwise. This is particularly useful when you need to check for the absence of data before performing operations like iteration or filtering.
```php
$users = collect([]);

if ($users->isEmpty()) {
    // $users is empty
}
```

### `isNotEmpty()`
The `isNotEmpty()` method is the inverse of `isEmpty()`. It returns true if the collection contains at least one item, and `false` if the collection is empty. This is useful for conditionally performing operations only when the collection has data.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice']
]);

if ($users->isNotEmpty()) {
    // $users is not empty
}
```

### `all()`
The `all()` method returns all items in the collection as a plain PHP array. This is useful when you need to access the raw underlying data for purposes like debugging, JSON serialization, or interacting with code that expects native arrays.
```php
$products = collect([
    ['id' => 101, 'name' => 'Laptop'],
    ['id' => 102, 'name' => 'Tablet'],
]);

$allProducts = $products->all();
```

Output
```json
[
    {
        "id": 101,
        "name": "Laptop"
    },
    {
        "id": 102,
        "name": "Tablet"
    }
]
```

Unlike the `toArray()` method, `all()` does not perform any transformation or casting on the items. It simply returns the raw `$data` array stored within the collection.

>  Use `all()` when you want untouched items, and use `toArray()` when working with Entity models or objects that implement `toArray()`

### `first()`
The `first()` method retrieves the first item in the collection. If the collection is empty, it returns null. This is helpful when you're only interested in the first element of a dataset—such as the first result of a query, the first matched item, or the initial record in a transformed list.
```php
$tasks = collect([
    ['id' => 1, 'title' => 'Fix bugs'],
    ['id' => 2, 'title' => 'Write tests'],
]);

$firstTask = $tasks->first();
```

Output
```json
{
    "id": 1,
    "title": "Fix bugs"
}
```

If the collection is empty, `first()` will safely return `null`:

You can pass a callback to get the first item that passes a test, and a default for when nothing does. The callback receives the item and its key:
```php
$numbers = collect([1, 2, 3, 4]);

$numbers->first(fn($n) => $n > 2);                  // 3
$numbers->first(fn($n) => $n > 9, 'none');          // 'none'
$numbers->first(fn($n) => $n > 9, fn() => 'lazy');  // 'lazy' (a Closure default is called)
```

### `last()`
The `last()` method works the same way from the other end. Without arguments it returns the last item, or `null` for an empty collection:
```php
$numbers = collect([1, 2, 3, 4]);

$numbers->last();                          // 4
$numbers->last(fn($n) => $n < 4);          // 3
$numbers->last(fn($n) => $n > 9, 'none');  // 'none'
```

### `pluck()`
The `pluck()` method retrieves all values for a given key from the collection items. It is very useful when you need a simple array of values from a specific field across all items.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
    ['id' => 3, 'name' => 'Charlie'],
]);

$names = $users->pluck('name');

print_r($names);

// Output: ['Alice', 'Bob', 'Charlie']
```

You can also specify an optional second parameter to use as the keys for the resulting array:
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
]);

$namesById = $users->pluck('name', 'id');

print_r($namesById);

// Output: [1 => 'Alice', 2 => 'Bob']
```

Internally, pluck() iterates over each item, retrieving the value for the given key (and optionally using another key as the resulting array’s keys), making it concise and readable.

Use dot notation to reach into nested arrays and objects:
```php
$orders = collect([
    ['id' => 1, 'customer' => ['name' => 'Alice', 'address' => ['city' => 'Paris']]],
    ['id' => 2, 'customer' => ['name' => 'Bob', 'address' => ['city' => 'Oslo']]],
]);

$orders->pluck('customer.name');                // ['Alice', 'Bob']
$orders->pluck('customer.address.city', 'id');  // [1 => 'Paris', 2 => 'Oslo']
```

A path that does not exist gives `null`. Dot notation works the same on objects, models and `ArrayAccess` values, and it is available wherever a method takes a key: `sortBy()`, `groupBy()`, `keyBy()`, `unique()`, `duplicates()`, `sum()`, `min()`, `max()`, `firstWhere()` and the `where*` methods. A key that really contains a dot, such as `'a.b'`, is tried as it is first.

### `groupBy()`
The `groupBy()` method groups the collection’s items by the value of a specified key.

It returns an associative array, where each key corresponds to a unique value from the given property, and each value is an array of items that share that same property value.

This is useful for organizing data into logical groups—for example, grouping users by role, products by category, or records by status.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice', 'role' => 'admin'],
    ['id' => 2, 'name' => 'Bob', 'role' => 'user'],
    ['id' => 3, 'name' => 'Charlie', 'role' => 'admin'],
]);

$grouped = $users->groupBy('role');
```

Output
```json
{
"admin":
    [
        {
            "id": 1,
            "name": "Alice",
            "role": "admin"
        },
        {
            "id": 3,
            "name": "Charlie",
            "role": "admin"
        }
    ],
"user":
    [
        {
            "id": 2,
            "name": "Bob",
            "role": "user"
        }
    ]
}
```

### `sortBy()`
The `sortBy()` method sorts the collection by a given key or callback. It returns a new collection with the items ordered in ascending order by default.
```php
$users = collect([
    ['name' => 'Charlie', 'age' => 30],
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 28],
]);

$sorted = $users->sortBy('age');

return $sorted->all();
```

Using a callback
```php
$sorted = $users->sortBy(fn($user) => strlen($user['name']));
```

The `sortByDesc()` method sorts the collection in descending order by the given key or callback
```php
$sorted = $users->sortByDesc('age');
```
This method behaves exactly like `sortBy()`, but reverses the order.

Sort by a nested value with dot notation:
```php
$sorted = $users->sortBy('profile.city');
```

To sort by several keys, pass an array. Items that tie on the first key are ordered by the next one:
```php
$users = collect([
    ['name' => 'Charlie', 'age' => 30],
    ['name' => 'Alice', 'age' => 25],
    ['name' => 'Bob', 'age' => 28],
    ['name' => 'Dana', 'age' => 30],
]);

$users->sortBy(['age', 'name']);
// Alice (25), Bob (28), Charlie (30), Dana (30)
```

Give each key its own direction as a `[key, direction]` pair, written `'asc'` or `'desc'`:
```php
$users->sortBy([['age', 'desc'], ['name', 'asc']]);
// Charlie (30), Dana (30), Bob (28), Alice (25)
```

The list may also contain callbacks. The sort is stable, so items that tie on every key keep their original order. The sort flags work as well, for example a case-insensitive text sort:
```php
$users->sortBy(['name'], SORT_STRING | SORT_FLAG_CASE);
```

### `chunk()`
The `chunk()` method breaks the collection into multiple smaller collections of a given size. It returns a new collection, where each item is an array representing a chunk.
```php
$numbers = collect([1, 2, 3, 4, 5, 6]);

$chunks = $numbers->chunk(2);

return $chunks->all();
```

Output
```php
[
    [1, 2],
    [3, 4],
    [5, 6]
]
```

### `partition()`
The `partition()` method splits the collection into two collections based on a condition.

It returns an array containing:
- Items that passed the condition
- Items that failed the condition

```php
[$active, $inactive] = $users->partition(fn($user) => $user['active']);
```

Output
```php
$active->all();   // Users where active === true
$inactive->all(); // Users where active === false
```

### `sole()`
The `sole()` method returns exactly one item from the collection.
```php
$user = $users->sole(fn($user) => $user['email'] === 'john@example.com');
```
This method is ideal when you expect exactly one result and want to fail loudly otherwise.

### `sum()`
The `sum()` method returns the total of the collection’s values.
```php
collect([10, 20, 30])->sum(); // 60
```

Using a key
```php
$users->sum('salary');
```

Using a callback
```php
$users->sum(fn($user) => $user['salary'] * 0.1);
```

### `avg()`
The `avg()` method returns the average (mean) value of the collection.
```php
$users->avg('salary');
```

### `min()`
The `min()` method returns the minimum value in the collection.
```php
$users->min('age');
```

You may also pass a callback:
```php
$users->min(fn($user) => $user['salary']);
```

### `max()`
The `max()` method returns the maximum value in the collection.
```php
$users->max('age');
```
> `max()` supports keys and callbacks just like `min()`.


### `intersect()`
The `intersect()` method returns a new collection containing items that exist in both collections.
```php
$common = collect([1, 2, 3])->intersect([2, 3, 4]);

return $common->all(); // [2, 3]
```

### `diff()`
The `diff()` method returns items that exist in the current collection but not in the given items.
```php
$diff = collect([1, 2, 3])->diff([2, 4]);

return $diff->all(); // [1, 3]
```

### `duplicates()`
The `duplicates()` method returns a new collection containing only duplicate items.
```php
$users = collect([
    ['email' => 'a@test.com'],
    ['email' => 'b@test.com'],
    ['email' => 'a@test.com'],
]);

$duplicates = $users->duplicates('email');

return $duplicates->all();
```

Output
```php
[
    { "email": "a@test.com" }
]
```

### `pipe()`
The `pipe()` method passes the collection into a callback and returns the callback’s result.
```php
$result = $users->pipe(function ($collection) {
    return $collection->filter(fn($u) => $u['active'])->count();
});
```
This is useful for extracting complex logic into reusable pipelines.

### `toArray()`
The `toArray()` method converts the entire collection into a plain PHP array, recursively calling `toArray()` on any items that are instances of Model.
```php
User::all()->toArray();
```

If an item is not an instance of `Model`, it will be returned as-is in the output array.

### `map()`
The `map()` method applies a callback function to each item in the collection and returns a new collection containing the transformed items. This is ideal when you want to transform or reshape data—for example, formatting output, changing values, or extracting specific fields.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
]);

$uppercased = $users->map(fn($user) => [
    ...$user,
    'name' => strtoupper($user['name']),
]);

return $uppercased;
```

Output
```json
[
    {
        "id": 1,
        "name": "ALICE"
    },
    {
        "id": 2,
        "name": "BOB"
    }
]
```
`map()` does not modify the original collection. It returns a new instance with the mapped results. If you're working with models or objects, you can also return transformed data structures like DTOs, strings, or even other collections inside your callback.

### `map()` with Property Shortcut
When you only need to access a property or call a method without arguments on each item, you can use the property shortcut syntax. This makes your code more concise:
```php
$users = collect([
    ['name' => 'Alice'],
    ['name' => 'Bob'],
    ['name' => 'Charlie'],
]);

$names = $users->map->name;

// Collection: ['Alice', 'Bob', 'Charlie']
```

This makes data transformation in Doppar collections both expressive and concise.

### Higher-Order Access
The same shortcut works for many other methods. Read a property, or call a method on each item, without writing a closure:
```php
$users->filter->isActive();    // keep the users for whom isActive() is true
$users->filter->active;        // keep the users whose active property is truthy
$users->reject->active;        // the opposite
$users->sum->points;           // total of the points property
$users->max->age;
$users->sortBy->age;
$users->groupBy->country;
$users->each->save();          // call save() on every item that has it
```

They chain like any other call:
```php
$names = $users->filter->isActive()->map->name;
```

This is available for `map`, `each`, `filter`, `reject`, `sum`, `avg`, `min`, `max`, `sortBy`, `sortByDesc`, `groupBy`, `keyBy`, `unique`, `every`, `some` and `flatMap`.

> **Note:** If the collection has a key with the same name as one of these methods (other than `map` and `each`), `$collection->filter` returns that value instead of the shortcut.

## Advanced Grouping & Keying
When working with collections, organizing data efficiently is key to writing clean, maintainable code. Doppar Collections provide powerful methods like `groupBy()` and `keyBy()` (underlying implementations: `mapAsGroup()` and `mapAsKey()`) that allow you to categorize, transform, and index your data in flexible ways.

> **Note:** A string you pass to `groupBy()`, `keyBy()`, `sortBy()` and the other key-based methods is always the name of a key, even when it is also the name of a PHP function such as `date` or `count`. To compute the key yourself, pass a closure.

With these methods, you can:
- Group items by a property, computed value, or custom logic.
- Transform values while grouping, keeping only the fields you need.
- Key collections by unique identifiers, composite keys, or dynamic conditions.
- Easily work with nested, relational, or large datasets while keeping your code expressive and concise.

In this section, we’ll explore practical examples that demonstrate how to leverage these capabilities to structure your data for real-world scenarios, such as:

- Grouping employees by department, status, or salary range.
- Creating associative arrays keyed by IDs, emails, or custom composite keys.
- Applying transformation callbacks to extract only the relevant data from each item.

By mastering advanced grouping and keying, you can make your collection handling both powerful and readable, keeping your code organized and scalable.

### `mapAsGroup()`
The `mapAsGroup()` method allows you to group items in the collection by a specific key or a callback.
This is useful when you want to categorize data — for example, grouping employees by their department, status, or role.

In the following example, we group a list of employees by their department:
```php
$collection = collect([
    [
        "id" => 1,
        "name" => "John Doe",
        "email" => "john@example.com",
        "department" => "IT",
        "salary" => 50000,
        "active" => true,
    ],
    [
        "id" => 2,
        "name" => "Jane Smith",
        "email" => "jane@example.com",
        "department" => "HR",
        "salary" => 45000,
        "active" => true,
    ],
    [
        "id" => 3,
        "name" => "Bob Johnson",
        "email" => "bob@example.com",
        "department" => "IT",
        "salary" => 55000,
        "active" => false,
    ],
    [
        "id" => 4,
        "name" => "Alice Brown",
        "email" => "alice@example.com",
        "department" => "Finance",
        "salary" => 60000,
        "active" => true,
    ],
]);

$groupedByDept = $collection->mapAsGroup("department");

print_r($groupedByDept);
```

Output will be like that
```json
[
    "IT" => [
        [
            "id" => 1,
            "name" => "John Doe",
            "email" => "john@example.com",
            "department" => "IT",
            "salary" => 50000,
            "active" => true,
        ],
        [
            "id" => 3,
            "name" => "Bob Johnson",
            "email" => "bob@example.com",
            "department" => "IT",
            "salary" => 55000,
            "active" => false,
        ],
    ],
    "HR" => [
        [
            "id" => 2,
            "name" => "Jane Smith",
            "email" => "jane@example.com",
            "department" => "HR",
            "salary" => 45000,
            "active" => true,
        ],
    ],
    "Finance" => [
        [
            "id" => 4,
            "name" => "Alice Brown",
            "email" => "alice@example.com",
            "department" => "Finance",
            "salary" => 60000,
            "active" => true,
        ],
    ],
];
```
Here, all employees are automatically grouped under their department keys (IT, HR, Finance).

You can also pass a second callback to `mapAsGroup()` to transform each item before it’s added to the group. This is useful when you only want to keep certain fields or apply some transformation.
```php
$groupedWithMap = $collection->mapAsGroup(
    "department",
    fn($user) => ["name" => $user["name"], "email" => $user["email"]],
);

$groupedByActive = $collection->mapAsGroup(
    fn($user) => $user["active"] ? "active" : "inactive",
    fn($user) => $user["name"],
);

$groupedBySalary = $collection->mapAsGroup(
    fn($user) => $user["salary"] >= 55000
        ? "high"
        : ($user["salary"] >= 45000
            ? "medium"
            : "low"),
    fn($user) => ["name" => $user["name"], "salary" => $user["salary"]],
);
```

> Note on `mapAsGroup()` vs `groupBy()`

The `mapAsGroup()` method is the underlying implementation for `groupBy()`. This means that any use of `mapAsGroup()` can be replaced with `groupBy()`, which is the preferred, developer-friendly method to group collection items.

Using `mapAsGroup()`
```php
$grouped = $collection->mapAsGroup('department');
```

Equivalent using `groupBy()`
```php
$grouped = $collection->groupBy('department');
```

### `mapAsKey() / keyBy()`
The `mapAsKey()` method allows you to transform a collection into an associative array where the keys are derived from a property or callback.
The `keyBy()` method is the user-friendly alias for `mapAsKey()` — both work identically.

You can also pass an optional callback to transform the value for each key.

Usages Example
```php
$keyedById = $collection->mapAsKey("id");
```

Equivalent using `keyBy()`
```php
$keyed = $collection->keyBy("id");
```

Output
```json
[
    1 => [ /* John Doe data */ ],
    2 => [ /* Jane Smith data */ ],
    3 => [ /* Bob Johnson data */ ],
    4 => [ /* Alice Brown data */ ],
]
```

See the others advanced example using callback
```php
$keyedByEmail = $collection->mapAsKey(
    "email",
    fn($user) => [
        "name" => $user["name"],
        "department" => $user["department"],
    ],
);

$keyedByComposite = $collection->mapAsKey(
    fn($user) => $user["department"] .
        "_" .
        str_replace(" ", "_", strtolower($user["name"])),
    fn($user) => $user["salary"],
);

$keyedByStatus = $collection->mapAsKey(
    fn($user) => $user["active"] ? "active_user" : "inactive_user",
);

$keyedWithMap = $collection->keyBy(
    'email',
    fn($user) => $user['name']
);
```

### `mapToGroups()`
The `mapToGroups()` method allows you to group items into multiple categories simultaneously. Unlike `groupBy()`, which groups by a single key, `mapToGroups()` lets each item belong to multiple groups at once — making it perfect for scenarios where your data needs to be classified in more than one way.

See the basic example of group employees by department and active status
```php
$multipleGroups = $collection->mapToGroups(function ($user) {
    $groups = [];

    // Group by department
    $groups[$user['department']] = $user['name'];

    // Also group by active status
    $status = $user['active'] ? 'active' : 'inactive';
    $groups[$status] = $user['name'];

    return $groups;
});

print_r($multipleGroups);
```

Output
```php
[
    "IT" => ["John Doe", "Bob Johnson"],
    "HR" => ["Jane Smith"],
    "Finance" => ["Alice Brown"],
    "active" => ["John Doe", "Jane Smith", "Alice Brown"],
    "inactive" => ["Bob Johnson"],
]
```

`mapToGroups()` is ideal for tagging or categorizing data along multiple dimensions without needing to run multiple grouping operations separatel.

### `mapWithKeys()`
The `mapWithKeys()` method transforms a collection into an associative array using a callback that returns custom key-value pairs.
Unlike `keyBy()`, which produces a single key per item, `mapWithKeys()` allows each item to generate multiple keys — giving you fine-grained control over the resulting array structure.

See the basic example of creating multiple keys per user
```php
$multipleKeys = $collection->mapWithKeys(function ($user) {
    return [
        'user_' . $user['id'] => $user['name'],
        'email_' . $user['id'] => $user['email'],
        'dept_' . $user['id'] => $user['department'],
    ];
});

print_r($multipleKeys);
```

Output
```php
[
    "user_1" => "John Doe",
    "email_1" => "john@example.com",
    "dept_1" => "IT",
    "user_2" => "Jane Smith",
    "email_2" => "jane@example.com",
    "dept_2" => "HR",
    "user_3" => "Bob Johnson",
    "email_3" => "bob@example.com",
    "dept_3" => "IT",
    "user_4" => "Alice Brown",
    "email_4" => "alice@example.com",
    "dept_4" => "Finance",
]
```
`mapWithKeys()` is ideal when a simple key mapping isn’t enough, and you need multiple keys derived from each item.

## `filter()`
The `filter()` method allows you to selectively include items in the collection by applying a callback function that returns true for items you want to keep.

It returns a new collection containing only the items that passed the condition.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice', 'active' => true],
    ['id' => 2, 'name' => 'Bob', 'active' => false],
    ['id' => 3, 'name' => 'Charlie', 'active' => true],
]);

return $users->filter(fn($user) => $user['active']);
```

Output
```json
[
    {
        "id": 1,
        "name": "Alice",
        "active": true
    },
    {
        "id": 3,
        "name": "Charlie",
        "active": true
    }
]
```

Without a callback, `filter()` removes every "falsy" item (`null`, `false`, `0`, `''` and `[]`):
```php
collect([0, 1, null, 'a', false, '', []])->filter();

// [1, 'a']
```

### `reject()`
The `reject()` method is the opposite of `filter()`. It keeps the items for which the callback returns `false`:
```php
collect([1, 2, 3, 4])->reject(fn($n) => $n % 2 === 0);

// [1, 3]
```

## `each()`
The `each()` method executes a callback function on each item in the collection. Unlike `map()`, `each()` does not return a new collection. Instead, it returns the original collection after executing the callback.

You can break out of the loop early by returning false from within the callback.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
    ['id' => 3, 'name' => 'Charlie'],
]);

$users->each(function ($user, $index) {
    echo "User #$index: {$user['name']}" . PHP_EOL;
});
```

Output
```php
User #0: Alice
User #1: Bob
User #2: Charlie
```

With early termination:
```php
$users->each(function ($user) {
    echo $user['name'] . PHP_EOL;
    return $user['id'] < 2; // stop when id is 2 or more
});

// Output: Alice
```
> `each()` is for iteration, not transformation. If you need to transform data, use map() instead.

## `take()` and `takeLast()`
The `take()` method returns a new collection containing only the first N items from the current collection.

Passing a positive `$limit` takes items from the beginning. Passing a negative `$limit` is treated as a request to take items from the end, and is automatically forwarded internally to `takeLast()`.

This method never modifies the original collection—it always returns a new instance.
```php
$collection = collect([10, 20, 30, 40, 50]);

$firstTwo = $collection->take(2);
// [10, 20]

$lastTwo = $collection->take(-2);
// [40, 50]
```

The `takeLast()` method returns a new collection containing only the last N items from the current collection.

If `$limit` is 0 or negative, an empty collection is returned. If `$limit` exceeds the size of the collection, the entire collection is returned.

This method also returns a new instance and never mutates the original collection.
```php
$collection = collect([10, 20, 30, 40, 50]);

$lastThree = $collection->takeLast(3);
// [30, 40, 50]

$none = $collection->takeLast(0);
// []
```

## `push()`
The `push()` method adds a single item to the end of the collection. It modifies the current collection in-place and returns the updated collection instance, making it suitable for method chaining if needed.

```php
$fruits = collect([
    'Apple',
    'Banana',
]);

$fruits->push('Cherry');

return $fruits->all();
```

Output
```php
[
    "Apple",
    "Banana",
    "Cherry"
]
```

You can also chain multiple operations with push():
```php
$numbers = collect([1, 2]);

$numbers->push(3)->push(4);

print_r($numbers->all()); // [1, 2, 3, 4]
```

## `values()`
The `values()` method returns a new collection with the keys reset to consecutive numeric indexes starting from zero. This is especially useful after operations like `filter()` or `unique()` that may leave "holes" in the array keys, which can cause unexpected behavior in loops or when using certain array functions
```php
$collection = collect([
    2 => ['id' => 1, 'name' => 'Alice'],
    5 => ['id' => 2, 'name' => 'Bob'],
]);

$reset = $collection->values();

return $reset->all();

// Output:
// [
//     0 => ['id' => 1, 'name' => 'Alice'],
//     1 => ['id' => 2, 'name' => 'Bob'],
// ]
```
Internally, this method uses PHP’s array_values() function on the collection's internal data array to reindex the items.

## `unique()`
The `unique()` method returns a new collection containing only unique items. You can optionally specify a key to determine uniqueness based on a specific field within each item. You can also specify whether to use strict comparison `(===)` for uniqueness checks.
```php
$users = collect([
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
    ['id' => 3, 'name' => 'Alice'],
]);

$uniqueUsers = $users->unique('name');

return $uniqueUsers->all();

/*
[
    ['id' => 1, 'name' => 'Alice'],
    ['id' => 2, 'name' => 'Bob'],
]
*/
```
Internally, this method builds a set of serialized values (optionally using strict comparison) to track which items have already been included. This makes it flexible and robust when dealing with arrays, objects, and mixed types.

The key may use dot notation, as in `unique('profile.city')`.

By default the comparison is loose, so `'1'`, `1` and `true` count as the same value. Different numbers always stay different, including decimals such as `1.5` and `1.2`. Pass `true` as the second argument to compare strictly:
```php
collect([1, '1', 1.5, 1.2])->unique();         // [1, 1.5, 1.2]
collect([1, '1'])->unique(null, true);         // [1, '1']
```


## `flatten()`
The `flatten()` method reduces a multi-dimensional collection into a single-level array. It recursively merges nested arrays or collections into one flat structure.

You can optionally specify the `$depth` to control how deep the flattening should go. By default, it flattens all levels.
Example – Fully flatten:
```php
$nested = collect([
    [1, 2],
    [3, [4, 5]],
    6,
]);

$flat = $nested->flatten();

echo $flat;

// Output: [1, 2, 3, 4, 5, 6]
```

### Example – Limit depth:
```php
$limited = collect([
    [1, 2],
    [3, [4, 5]],
]);

$flat = $limited->flatten(1);

print_r($flat->all());
```

Output
```php
Array
(
    [0] => 1
    [1] => 2
    [2] => 3
    [3] => Array
        (
            [0] => 4
            [1] => 5
        )
)
```
`flatten()` works only on nested arrays or array-like structures. If you store objects with nested collections or models, flattening won't traverse inside them unless they are cast to arrays first.

## `pluck()` with `flatten()` and `unique()`
The combination of `pluck()`, `flatten()`, and `unique()` allows you to efficiently extract and deduplicate deeply nested data structures within a collection. This is particularly useful when working with `Entity relationships` or nested arrays, such as retrieving a list of unique post titles from a collection of users, each having multiple posts.

Example
```php
User::query()
    ->pluck('posts')  // Extract the 'posts' array from each user
    ->flatten()       // Merge all post arrays into a single flat collection
    ->unique('title') // Keep only posts with unique titles
    ->toArray();      // Convert the result to a plain array
```

Visual Flow:
```
Users → [Posts, Posts, Posts] → Flatten → [Post1, Post2, Post3...] → Unique by 'title' → Array
```
This technique showcases the power and expressiveness of Doppar collections when working with nested and relational data.

## Searching, Slicing and Conditions
The following methods cover everyday tasks that would otherwise need a closure or a loop.

### `firstWhere()`
The `firstWhere()` method returns the first item whose value for a key matches. With one argument it checks that the value is truthy, with two it checks equality, and with three it compares with an operator (`=`, `==`, `===`, `!=`, `<>`, `!==`, `<`, `>`, `<=`, `>=`):
```php
$users = collect([
    ['name' => 'Alice', 'age' => 30, 'active' => true],
    ['name' => 'Bob', 'age' => 25, 'active' => false],
]);

$users->firstWhere('active');          // Alice
$users->firstWhere('age', 25);         // Bob
$users->firstWhere('age', '>=', 30);   // Alice
$users->firstWhere('age', 99);         // null
```

### `whereIn()`, `whereNotIn()`, `whereNull()` and `whereNotNull()`
Filter by the value of a key:
```php
$users->whereIn('age', [25, 30]);
$users->whereNotIn('name', ['Bob']);
$users->whereNotNull('email');
$users->whereNull('deleted_at');
```

`whereIn()` and `whereNotIn()` compare loosely. Pass `true` as the third argument to compare strictly.

### `every()` and `some()`
`every()` is true when all items pass the test, and `some()` is true when at least one does:
```php
$ages = collect([18, 25, 40]);

$ages->every(fn($age) => $age >= 18);   // true
$ages->some(fn($age) => $age > 30);     // true
```

`every()` returns `true` for an empty collection and `some()` returns `false`.

### `search()`
The `search()` method returns the key of a value, or of the first item that passes a test, and `false` when there is none:
```php
$fruits = collect(['a' => 'apple', 'b' => 'banana']);

$fruits->search('banana');                          // 'b'
$fruits->search(fn($fruit) => str_starts_with($fruit, 'a'));  // 'a'
$fruits->search('cherry');                          // false
```

Pass `true` as the second argument for a strict comparison. A string is always searched for as a value, never called as a function.

### `skip()` and `slice()`
```php
$numbers = collect([1, 2, 3, 4, 5]);

$numbers->skip(2);        // [3, 4, 5]
$numbers->slice(1, 2);    // [2, 3]
$numbers->slice(-2);      // [4, 5]
```

### `reverse()`, `keys()`, `only()` and `except()`
```php
collect([1, 2, 3])->reverse();                       // [3, 2, 1]
collect(['a' => 1, 'b' => 2])->keys();               // ['a', 'b']
collect(['a' => 1, 'b' => 2, 'c' => 3])->only(['a', 'c']);   // ['a' => 1, 'c' => 3]
collect(['a' => 1, 'b' => 2, 'c' => 3])->except(['a']);      // ['b' => 2, 'c' => 3]
```

### `merge()` and `concat()`
`merge()` combines the collection with other collections, arrays or iterators into a new one. Numeric keys are renumbered, and a string key from a later one replaces an earlier one. `concat()` always adds the values at the end:
```php
collect([1, 2])->merge([3], collect([4]));      // [1, 2, 3, 4]
collect(['a' => 1])->merge(['a' => 2]);         // ['a' => 2]
collect([1, 2])->concat([3, 4]);                // [1, 2, 3, 4]
```

### Adding Items
Append with `push()`, `add()` or the array syntax:
```php
$items = collect([1]);

$items->push(2);
$items->add(3);
$items[] = 4;

// [1, 2, 3, 4]
```

### `collapse()` and `flatMap()`
`collapse()` joins an array of arrays (or collections) into a single level. `flatMap()` maps every item and collapses the result:
```php
collect([[1, 2], [3], [4]])->collapse();                    // [1, 2, 3, 4]
collect(['ab', 'c'])->flatMap(fn($text) => str_split($text));  // ['a', 'b', 'c']
```

### `countBy()`
The `countBy()` method counts how often each value occurs and returns an array. Without an argument it counts the items themselves; pass a key or a callback to count something else:
```php
collect(['a', 'b', 'a'])->countBy();                          // ['a' => 2, 'b' => 1]
$users->countBy('age');                                       // [30 => 1, 25 => 1]
collect([1, 2, 3])->countBy(fn($n) => $n % 2 ? 'odd' : 'even');  // ['odd' => 2, 'even' => 1]
```

### `implode()`
Join the items into a string, or one key of each item:
```php
collect([1, 2, 3])->implode(', ');      // '1, 2, 3'
$users->implode(', ', 'name');          // 'Alice, Bob'
```

### `median()`
The `median()` method returns the middle value, ignoring `null` values, or `null` when there are none. Pass a key to take the values from each item:
```php
collect([5, 1, 3])->median();      // 3
collect([1, 2, 3, 4])->median();   // 2.5
$users->median('age');
```

### `when()` and `unless()`
Run a callback only when a condition holds, so a chain does not have to be broken with an `if`:
```php
$users = collect($rows)
    ->when($request->has('active'), fn($users) => $users->filter->active)
    ->unless($request->has('all'), fn($users) => $users->take(10));
```

The condition can be a closure, which receives the collection. A third argument runs when the condition is not met. If the callback returns nothing, the collection itself is passed on.

## `withMemoryUsage()`
The `withMemoryUsage()` method provides insight into PHP memory usage at the time the method is called. It helps with profiling, debugging, and performance monitoring, especially during collection-heavy operations.

You can choose to return the memory usage as a formatted string (default) or as a detailed array of metrics.

Example – Default (string output):
```php
$collection = collect(range(1, 10000));

echo $collection->withMemoryUsage();
```

Output:
> Memory usage: 1.5 MB, Peak: 2 MB