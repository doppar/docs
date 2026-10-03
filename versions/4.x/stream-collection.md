---
title: Stream Collections
description: Stream Doppar Collections page
meta:
  - name: keywords
    content: Stream Collections, Lazy Collection
---

## Stream Collections
### Introduction
Doppar’s `StreamCollection` provides a lazy, memory-efficient way to handle large or infinite datasets. Built on top of PHP’s powerful generator system, StreamCollection enables data to be processed incrementally, only as needed, instead of materializing the entire dataset in memory. This design makes it exceptionally well suited for working with large files, database cursors, API streams, or any situation where efficiency and scalability are critical.

Unlike a traditional collection, which stores all of its items in memory, a StreamCollection represents a continuous flow of data that can be transformed, filtered, and mapped lazily through an expressive, chainable API. Each operation — such as `map()`, `filter()`, or `chunk()` — returns a new stream without evaluating the underlying data until you explicitly iterate or collect it. This approach makes it possible to build complex data-processing pipelines that remain elegant, composable, and performant even when working with gigabytes of information.

Like the standard `Collection`, `StreamCollection` offers a fluent, expressive API for data manipulation. However, instead of storing all items in memory, each operation (`map`, `filter`, `chunk`, etc.) creates a **new lazy stream** that yields values as they’re needed.

## Creating a Stream
Create a stream with `StreamCollection::make()` or `new StreamCollection()`. The source can be a closure that yields values, a `Generator` or any iterator, or an array:
```php
use Phaseolies\Support\StreamCollection;

StreamCollection::make(function () {
    yield 1;
    yield 2;
});

StreamCollection::make($generator);
StreamCollection::make(new ArrayIterator([1, 2, 3]));
StreamCollection::make([1, 2, 3]);
```

### Ranges and Repeats
`range()` yields numbers one at a time, so even a huge range uses no memory:
```php
StreamCollection::range(1, 5)->all();        // [1, 2, 3, 4, 5]
StreamCollection::range(0, 10, 5)->all();    // [0, 5, 10]
StreamCollection::range(3, 1)->all();        // [3, 2, 1]
StreamCollection::range(1, PHP_INT_MAX)->take(3)->all();   // [1, 2, 3]
```

`times()` calls a callback a number of times, passing 1, 2, 3 and so on. Without a callback the numbers themselves are the items:
```php
StreamCollection::times(3)->all();                         // [1, 2, 3]
StreamCollection::times(3, fn($i) => $i * 10)->all();      // [10, 20, 30]
```

### Reading a File
`lines()` reads a file one line at a time, without the line breaks:
```php
StreamCollection::lines(storage_path('logs/app.log'))
    ->filter(fn($line) => str_contains($line, 'ERROR'))
    ->take(10)
    ->all();
```

The file is opened when you start iterating and closed when the stream ends, or when you stop early, as `take(10)` does above. An exception is thrown straight away if the file does not exist or cannot be read.

## Basic Usage
StreamCollections work by chaining **lazy operations** such as `map`, `filter`, and `take`, which do not execute immediately.

Instead, they build a *streaming pipeline* that processes items only when the stream is consumed — for example, when calling `collect()`, `all()`, or iterating with `foreach`.

Here’s a simple example:

```php
use Phaseolies\Support\StreamCollection;

StreamCollection::make(function () {
    for ($i = 1; $i <= 5; $i++) {
        yield $i;
    }
})
    ->filter(fn($n) => $n > 2)
    ->map(fn($n) => $n * 2)
    ->take(2)
    ->collect();
```

This `collect()` method convert the lazy stream into a regular Collection. Even though the source produces 5 values, only the first 4 are processed — just enough to yield 2 results after filtering.
This makes `StreamCollections` extremely efficient for large or unbounded data source

In this example, we'll demonstrate how `StreamCollection` can process structured data lazily — filtering, transforming, and extracting values efficiently without building large arrays in memory.
```php
StreamCollection::make(function () {
    yield ['id' => 1, 'name' => 'Alice', 'age' => 24];
    yield ['id' => 2, 'name' => 'Bob', 'age' => 30];
    yield ['id' => 3, 'name' => 'Charlie', 'age' => 24];
})
    ->filter(fn($u) => $u['age'] > 24)
    ->map(fn($u) => strtoupper($u['name']))
    ->unique()
    ->values()
    ->take(1)
    ->all();
```

Output
```php
['BOB']
```
Even if the generator contained thousands of users, only the minimal number needed to produce the first matching record would ever be processed — keeping memory usage extremely low.

## Streaming Large Files
One of the most powerful use cases for `StreamCollection` is handling large files such as CSVs or logs — where loading the entire file into memory would be inefficient or impossible.  
Because streams are processed lazily, each line is read, transformed, and handled as it becomes available.

The quickest way is `lines()`:
```php
StreamCollection::lines('product.csv')
    ->map(fn($line) => str_getcsv($line))
    ->each(function ($row) {
        // Process or inspect each row as it streams
    });
```

You can also write the generator yourself when you need full control over how the file is read:

```php
StreamCollection::make(function () {
    $handle = fopen('product.csv', 'r');
    while (($row = fgets($handle)) !== false) {
        yield str_getcsv($row);
    }
    fclose($handle);
})
->each(function ($row) {
    // Process or inspect each row as it streams
});
```
This approach can handle gigabyte-sized files smoothly, since it uses constant memory regardless of file size. This pattern turns large file processing into a clean, expressive, and memory-safe workflow — perfect for ETL jobs, log parsing, CSV imports, and any other bulk data tasks in Doppar.

## Chunked Stream Processing

When working with large files or continuous data streams, it’s often more efficient to process data in **batches** rather than one record at a time.

With `StreamCollection`, you can easily use the `chunk()` method to split a lazy stream into smaller `Collection` chunks — each processed independently.

```php
StreamCollection::make(function () {
    $handle = fopen('product.csv', 'r');
    while (($row = fgets($handle)) !== false) {
        yield str_getcsv($row);
    }
    fclose($handle);
})
->chunk(2)
->each(function ($row) {
    //
});
```
The `chunk(2)` call groups every two rows into a Collection object. This allows you to process batches efficiently (for example, inserting into a database or sending to an API). 

At any given time, only the current chunk is in memory — no matter how large the file is. The `each()` method consumes each chunk as it streams, triggering your processing logic (e.g. transforming, validating, or storing).

## Reading Only What You Need
A stream only reads from its source when something asks for the next item, and it stops as soon as it has what it needs. `take()` reads exactly as many items as it returns, `first()` stops at the first match, and `every()`, `some()` and `takeWhile()` stop at the first item that decides the result:
```php
$readings = StreamCollection::make(function () {
    foreach (range(1, 1000000) as $n) {
        yield $n;
    }
});

$readings->take(3)->all();                         // reads 3 items
$readings->first(fn($n) => $n > 10);               // reads 11 items
$readings->some(fn($n) => $n === 5);               // reads 5 items
$readings->takeWhile(fn($n) => $n < 4)->all();     // reads 4 items
```

`unique()` is lazy as well. It passes each new value on as soon as it is read and keeps the first of each, so it works on a stream that never ends:
```php
$endless = StreamCollection::make(function () {
    $i = 0;

    while (true) {
        yield $i++ % 5;
    }
});

$endless->unique()->take(3)->all();   // [0, 1, 2]
```

Other lazy methods are `reject()`, `skip()`, `skipWhile()`, `pluck()`, `flatten()`, `flatMap()`, `concat()`, `values()` and `chunk()`. `filter()` without a callback removes every falsy item.

## Totals and Searching
These methods read the stream one item at a time and return a single result, so they use constant memory:
```php
$orders = StreamCollection::make($ordersFromADatabaseCursor);

$orders->sum('total');                        // add up one key
$orders->avg('total');                        // null for an empty stream
$orders->min('total');
$orders->max('customer.age');                 // dot notation reaches nested values
$orders->sum(fn($order) => $order['total'] * 1.2);
$orders->reduce(fn($carry, $order) => $carry + $order['total'], 0);
$orders->count();
$orders->last();
$orders->last(fn($order) => $order['paid']);
```

`first()` and `last()` accept a test and a default, like on `Collection`. `every()` and `some()` check a test against the items.

## Iterating a Stream More Than Once
A stream made from a closure can be iterated as often as you like, because the closure runs again each time. A stream made from a `Generator` or an iterator object can only be read once, and reading it a second time throws `Cannot traverse an already closed generator`.

Call `remember()` to keep the items as they are read, so the stream can be iterated again:
```php
$stream = StreamCollection::make($generator)->remember();

$stream->count();
$stream->all();    // the source is not read a second time
```

`remember()` is still lazy: the source is read only as far as the stream is consumed, and `take(2)` reads two items. It keeps the items it has read in memory, so use it for streams that fit.

## Keys
`all()`, `toArray()` and `collect()` keep the keys the stream yields. When a stream yields the same key more than once, which is what `yield from` does with several arrays, later items replace earlier ones. Call `values()` first to number the items:
```php
$stream = StreamCollection::make(function () {
    yield from [1, 2];
    yield from [3, 4];
});

$stream->all();                  // [3, 4]
$stream->values()->all();        // [1, 2, 3, 4]
```

`flatten()`, `flatMap()`, `concat()`, `unique()`, `values()` and `lines()` number their items, so they are safe to combine this way. `chunk()` keeps the keys inside each chunk and adds an item at the next free position when a key repeats, so no item is lost.

## Looping
`each()` calls a callback for every item and returns the stream. Return `false` from the callback to stop:
```php
StreamCollection::lines('import.csv')->each(function ($line, $key) {
    if ($line === 'END') {
        return false;
    }

    // import the line
});
```

## Available Methods
These methods return a new stream and do no work until you iterate it:

| Method | Description |
|--------|-------------|
| `map(callback)` | Transform each item |
| `filter(callback?)` | Keep items that pass; without a callback, drop falsy items |
| `reject(callback)` | Drop items that pass |
| `take(count)` | The first items, reading no more than that |
| `takeWhile(callback)` | Items until the test first fails |
| `skip(count)` | Skip the first items |
| `skipWhile(callback)` | Skip items while the test passes |
| `unique(key?, strict?)` | Keep the first of each value |
| `pluck(key, by?)` | One key of each item, dot notation allowed |
| `flatten(depth?)` | Join nested arrays, collections and streams |
| `flatMap(callback)` | Map each item to a list and join the lists |
| `concat(items)` | Add items at the end |
| `values()` | Number the items |
| `chunk(size)` | Group items into `Collection` batches |
| `push(item)` | Add an item to this stream |
| `remember()` | Keep items so the stream can be read again |

These methods read the stream and return a result:

| Method | Description |
|--------|-------------|
| `first(test?, default?)` | First item, stops at a match |
| `last(test?, default?)` | Last item |
| `each(callback)` | Run a callback; return `false` to stop |
| `reduce(callback, initial?)` | Reduce to one value |
| `sum(key?)`, `avg(key?)`, `min(key?)`, `max(key?)` | Totals; the key may be a callback |
| `every(test)`, `some(test)` | Check a test, stopping early |
| `count()`, `isEmpty()`, `isNotEmpty()` | Size (`isEmpty()` reads one item) |
| `tap(callback)`, `pipe(callback)` | Pass the stream to a callback |
| `collect(model?)`, `all()`, `toArray()`, `toJson()` | Read everything into memory |

Methods that need every item at once, such as `groupBy()`, `sortBy()` and `keyBy()`, are on `Collection`. Call `collect()` first to use them.

```php
StreamCollection::lines('orders.csv')
    ->map(fn($line) => str_getcsv($line))
    ->collect()
    ->groupBy(0);
```
