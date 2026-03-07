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

This makes it ideal for:
- Reading and transforming large files.
- Iterating over database or API results.
- Building pipelines that process data lazily.
- Performing transformations, filtering, and mapping operations on data streams.

Like the standard `Collection`, `StreamCollection` offers a fluent, expressive API for data manipulation. However, instead of storing all items in memory, each operation (`map`, `filter`, `chunk`, etc.) creates a **new lazy stream** that yields values as they’re needed.

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
->filter(fn($n) => $n > 2) // Keep only numbers greater than 2
->map(fn($n) => $n * 2)    // Multiply each remaining number by 2
->take(2)                  // Take only the first 2 results
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