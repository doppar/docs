---
title: Entity Builder
description: Doppar Entity Builder page
meta:
  - name: keywords
    content: Entity Builder
---

  - [Query Using Builder](#query-using-builder)
  - [Insertion and Update](#insertion-and-update)
  - [Aggregation](#aggregation)
  - [Querying Date Columns](#querying-date-columns)
  - [Transform Entity Collection](#transform-entity-collection)
  - [Pagination](#pagination)
  - [Cursor Pagination](#cursor-pagination)
  - [Database Transactions](#database-transactions)
  - [Entity Join](#entity-join)
  - [DB Query](#db-query)
  - [Handling Multiple Database Connection](#handling-multiple-database-connection)

## Entity Builder
### Introduction
Doppar introduces Entity Builder, a powerful, flexible, and model-free query builder designed for developers who want full control over their database interactions — without the need for predefined data models.

With Entity Builder, you can construct, execute, and manage complex SQL queries using a clean, expressive, and chainable interface. Whether you're fetching data, inserting records, updating fields, or performing intricate joins and filters, Entity Builder brings the full power of SQL into an intuitive, object-oriented syntax that feels natural and effortless.

While Doppar Entity ORM excels at model-driven workflows, Entity Builder is perfect for dynamic scenarios, raw data manipulation, or rapid prototyping — letting you query any table directly. It supports nearly all ORM-level query capabilities, minus relationship-based operations, while maintaining the same elegant syntax and high performance that Doppar is known for.

Built entirely within Doppar’s ecosystem, Entity Builder operates with zero third-party dependencies, ensuring unmatched speed, consistency, and reliability across all your data-driven applications.

With Doppar Entity Builder, you get the freedom of raw SQL — with the clarity, power, and safety of Doppar.

## Query Using Builder
### Retrieving Data
Doppar provides the `db()` helper to interact with your database connection according to the currently active driver and bucket configuration.
Using this helper, you can easily query any table without relying on predefined models.

For example, to retrieve all records from the post table:
```php
db()->bucket('post')->get();
```
This simple and expressive syntax leverages the Entity Builder, allowing you to construct queries fluently while maintaining Doppar’s consistency and performance across different database drivers.

### Expending Queries
While the `get()` method retrieves all records from a specified table, Entity Builder allows you to extend and refine your queries with powerful, chainable methods — giving you full control over the data you fetch.
```php
db()->bucket('post')
    ->where('status', true)
    ->orderBy('title')
    ->limit(10)
    ->get();
```

### Measure Memory Usage
You can measure memory consumption of a query result by chaining `->withMemoryUsage()` after data retrieval:
```php
db()->bucket('post')->limit(10)->get()->withMemoryUsage();

// Memory usage: 2 MB, Peak: 2 MB
```
What It Does:
- Outputs the memory used to fetch and hold the result.
- Helps profile large datasets or optimize memory-sensitive operations.

> Note: These methods are non-intrusive and designed for development use only.

### Collections
As observed, Entity builder methods `get` are designed to fetch multiple records from the database. Rather than returning a standard PHP array, these methods yield an instance of `Phaseolies\Support\Collection`, providing a rich set of tools to work with the retrieved records efficiently.

If you want to convert collection to array, you can use `toArray()` method.
```php
db()->bucket('post')
    ->where('status', true)
    ->orderBy('title')
    ->limit(10)
    ->get()
    ->toArray();
```

### Dynamic `where` Conditions
Doppar supports dynamic query building using expressive method calls, allowing you to easily filter results using field names directly in the method. like
```php
db()->bucket('user')->whereName('Nure')->get();
```
This is equivalent to:
```php
db()->bucket('user')->where('name', 'Nure')->get();
```
The method name must follow `camelCase` or `StudlyCase` based on the column name. For example:
```php
db()->bucket('user')
    ->whereEmail('user@example.com')
    ->whereCountryCode('880')
    ->get();
```
Only works with existing columns in your table.

### Fetch the First Records
To fetch the first records you can use `first()` function as like below
```php
db()->bucket('post')->first();
```

Actually this above query returns a collection object so you can directly fetch its attributes like
```php
db()->bucket('post')->first()?->title ?? 'defult';
```

### GroupBy and OrderBy
You can also use `groupBy()` and `orderBy()` to handle your records like
```php
db()->bucket('users')
    ->orderBy('id', 'desc')
    ->groupBy('name')
    ->get();
```

### toSql()
The toSql() method in Entity Builder is a useful tool when you want to inspect the raw SQL query that would be executed for a given Entity query. Instead of retrieving the results from the database, `toSql()` returns the SQL statement as a string, allowing you to debug or log the query before it's run. This is especially helpful during development when you need to understand how your query builder chain is being translated into SQL.

Here's an example:
```php
db()->bucket('users')->where('status', 'active')->toSql();
```
This will output:
```sql
select * from `users` where `status` = ?
```
Using `toSql()` helps you optimize and troubleshoot queries by giving insight into what Doppar is sending to the database under the hood.

### count()
To see how many records your table has, you can use count() function as like
```php
db()->bucket('users')->count();
db()->bucket('users')->orderBy('id', 'desc')->groupBy('name')->count();
db()->bucket('users')->where('active', true)->count();
```

### Newest and Oldest Records
Doppar makes it convenient to retrieve the most recent or earliest records from a database using the `newest()` and `oldest()` query methods. These methods sort the results based on a specified column, with id being the default field if none is provided.

To fetch the latest records (in descending order):
```php
db()->bucket('users')->newest()->get();
```
This retrieves the most recently added users based on the id column by default. If you want to sort by a different column, you can pass it as an argument:
```php
db()->bucket('users')->newest('name')->get();
```
Similarly, to retrieve the oldest records (in ascending order):
```php
db()->bucket('users')->oldest()->get();
```
Again, you can specify a custom column to determine what "oldest" means in your context:
```php
db()->bucket('users')->oldest('id')->get();
```

### Selecting Specific Columns
In many cases, you may not need to retrieve every column from a table—especially when working with large datasets. Entity's `select()` method allows you to specify exactly which columns you want to fetch, helping optimize performance and reduce memory usage.

Here are a few examples. Selecting specific columns using an array:
```php
db()->bucket('users')->select(['name', 'email'])->get();
```

Selecting specific columns using multiple arguments:
```php
db()->bucket('users')->select('name', 'email')->get();
```

Both of these will return only the name and email fields for each user, excluding all other columns.

You can also use `select()` in more advanced queries, such as when grouping results:
```php
db()->bucket('post')
    ->select('title', 'COUNT(*) as count')
    ->groupBy('category_id')
    ->get();
```
Using `select()` helps tailor your queries to fetch only the data you actually need, improving both efficiency and clarity in your code.


### Excludes specific columns
Excludes specific columns from the `SELECT` query. This method is useful when you want to retrieve all columns except certain ones, such as timestamp or metadata fields.
```php
db()->bucket('post')
    ->omit('created_at', 'updated_at', 'excerpt')
    ->get();
```
In the example above, all columns will be selected `except`, `created_at` and `updated_at`.

You can also pass an array instead of variadic arguments:
```php
db()->bucket('post')->omit(['deleted_at', 'archived_at'])->get();
```
If the query initially selects all columns using *, `omit()` will first resolve the actual column names via the table schema and then exclude the specified ones.

## selectRaw()
The `selectRaw()` method in Doppar's query builder provides a powerful way to include raw SQL expressions in your queries. It's especially useful when you need to perform calculations, use SQL functions, or include complex expressions that go beyond Entity's default capabilities.

### Basic Usage

You can use `selectRaw()` to write a custom SQL expression directly in the SELECT clause. For example, to count the total number of orders:

```php
db()->bucket('order')
    ->selectRaw('COUNT(*) as order_count')
    ->first();

echo $total->order_count;
```
This will execute a query similar to:
```sql
SELECT COUNT(*) as order_count FROM orders;
```

### Chaining Multiple selectRaw() Calls
You can chain multiple `selectRaw()` calls to build a SELECT clause that includes several raw expressions. Each call adds to the final SQL:
```php
db()->bucket('order')
    ->selectRaw('SUM(price) as total_sales')
    ->selectRaw('AVG(quantity) as average_quantity')
    ->selectRaw('MAX(price) as highest_price')
    ->first();
```
This returns an object with aggregated values such as total sales, average quantity, and the highest price.
> ⚠️ Security Tip: When using raw expressions, especially with user input, always bind parameters or sanitize values to protect against SQL injection.

### Using Bindings with selectRaw()
Scenario:
You're running an e-commerce platform. You store orders in an orders table with these columns:

- price: base price of an item
- category_id: the category the product belongs to
- status: whether the order is completed (true) or not
- created_at: timestamp of the order

You want to:
- Get the maximum order value after tax (e.g. 8%) and after VAT (e.g. 20%)
- Only consider completed orders
- Group results by category

Now assume
"For each product category, what's the highest order value including tax and VAT, from completed orders?"

```php
db()->bucket('product')
    ->selectRaw('MAX(price * ?) AS total_with_tax', [1.08]) // Apply 8% tax
    ->selectRaw('MAX(price * ?) AS total_with_vat', [1.20]) // Apply 20% VAT
    ->where('status', true)
    ->groupByRaw('category_id')
    ->get();
```

Example output
| category\_id | total\_with\_tax | total\_with\_vat |
| ------------ | ---------------- | ---------------- |
| 1            | 108.00           | 120.00           |
| 2            | 216.00           | 240.00           |
| 3            | 162.00           | 180.00           |

Why Use selectRaw Here?
- You need to run math operations (price * taxRate) directly in SQL.
- You want to bind the tax rate dynamically (?) to avoid hardcoding values.
- You’re leveraging SQL’s aggregate function (MAX()) for reporting.

Bindings are passed as an array to selectRaw() and must match the number of ? placeholders in the SQL expression.

### whereLike()
The `whereLike()` method provides a convenient way to perform SQL LIKE pattern matching on a column. It’s especially useful for building search filters or partial text matching queries without writing raw SQL.

Basic LIKE condition
```php
db()->bucket('users')->whereLike('name', 'john')->get();
```
SQL Equivalent:
```sql
SELECT * FROM users WHERE name LIKE '%john%';
```
Combining with other conditions
```php
db()->bucket('users')
    ->where('status', 'active')
    ->whereLike('email', 'example.com')
    ->get();
```

SQL Equivalent:
```sql
SELECT * FROM users
WHERE status = 'active'
  AND email LIKE '%example.com%';
```

### orWhereLike()
The `orWhereLike()` method adds a logical `OR` condition with a `LIKE` clause.
```php
db()->bucket('users')
    ->where('status', 'active')
    ->orWhereLike('name', 'john')
    ->get();
```

### whereRaw()
The `whereRaw()` method allows you to write a raw SQL WHERE clause directly into an Entity query. This is useful when your query requires SQL features that aren't easily expressed using Entity's fluent methods — such as complex conditions, custom SQL functions, or case-insensitive matching.

You pass a raw SQL string as the first argument, and an optional array of bindings as the second. This helps maintain parameter binding and protects against SQL injection.

Here’s an example:
```php
db()->bucket('users')
    ->whereRaw('LOWER(name) LIKE LOWER(?)', ['%jewel%'])
    ->get();
```
This query returns all users whose name matches “jewel” in a case-insensitive way, by applying the `LOWER()` SQL function to both the column and the search term.

### orderByRaw()
The `orderByRaw()` method allows you to write a raw SQL ORDER BY clause directly into an Entity query. This is useful when you need advanced sorting logic that can't be easily represented using Entity’s fluent orderBy() method — such as ordering by multiple columns with different sort directions or using SQL functions.

You pass a raw SQL string as the first argument, which defines the custom sorting logic. Because the input is raw SQL, it gives you full control over the order clause — but it’s important to ensure the syntax is correct and secure.

Here’s an example:
```php
db()->bucket('employee')
    ->orderByRaw('salary DESC, name ASC')
    ->get()
```

Use `orderByRaw()` when your ordering requirements exceed the capabilities of standard Entity methods.

### groupByRaw()
The `groupByRaw()` method allows you to define a raw SQL `GROUP BY` clause in an Entity query. It’s especially useful when you need to group results by SQL functions or multiple columns that can't be easily handled using Doppar’s fluent `groupBy()` method.

You pass a raw SQL string as the argument, which gives you full control over the grouping logic.

Here’s an example:
```php
db()->bucket('users')
    ->selectRaw('COUNT(*) as total, YEAR(created_at) as year, MONTH(created_at) as month')
    ->groupByRaw('YEAR(created_at), MONTH(created_at)')
    ->orderByRaw('year DESC, month DESC')
    ->get();
```

This query:
- Counts how many users were created in each month,
- Groups the results by year and month using the YEAR() and MONTH() SQL functions,
- Orders the grouped results from most recent to oldest.

This is useful for generating reports, monthly user activity, or analytics dashboards.

### exists()
To check whether a specific row exists in your database, you can use the `exists()` function. This method returns a boolean value (true or false) based on whether the specified condition matches any records. Here's an example:
```php
db()->bucket('users')->where('id', 1)->exists();
```
Returns `true` if a matching row exists, otherwise `false`.

### whereIn()
The `whereIn()` method filters records where a column's value matches any value in the given array.
```php
db()->bucket('users')->whereIn('id', [1, 2, 4])->get();
```
This retrieves all users with id values of 1, 2, or 4.

### orWhereIn()
The `orWhereIn()` method filters records where a column's value optionally matches any value in the given array.
```php
db()->bucket('users')->orWhereIn('id', [1, 2, 4])->get();
```

### whereBetween()
The `whereBetween()` method in Doppar lets you filter records where a column’s value falls within a specified range. It’s particularly useful for working with numeric values or date/time columns, such as filtering records created within a certain time frame.

#### Example: Filter Users by Date Range
```php
db()->bucket('users')
    ->whereBetween('created_at', ['2025-02-29', '2025-04-29'])
    ->get();
```
This query will retrieve all users whose created_at timestamp falls between February 29, 2025 and April 29, 2025, inclusive.

You can use `whereBetween()` with any comparable column — such as prices, IDs, scores, or dates — to make your queries more expressive and efficient.
```php
db()->bucket('users')->whereBetween('id', [1, 10])->get();
```
This query retrieves all user records where the id is between 1 and 10, inclusive. That means it will return users with id values of 1, 2, 3, ..., 10.

### whereNotBetween()
The whereNotBetween() method allows you to filter records where a given column's value do not falls within a specified range. This is commonly used for date or numerical ranges.
```php
db()->bucket('users')
    ->whereNotBetween('created_at', ['2025-02-29', '2025-04-29'])
    ->get();
```

### orWhereBetween()
Doppar allows you to construct expressive, flexible queries using methods like orWhereBetween(). This method is particularly useful when you want to retrieve records that match either a specific condition or fall within a certain range.

Example: Combine exact match with a range
```php
db()->bucket('post')
    ->where('status', 'published')
    ->orWhereBetween('views', [100, 500])
    ->get();
```
This will return all posts that are either published or have between 100 and 500 views, inclusive.

### orWhereNotBetween()
Similarly, orWhereNotBetween() is used when you want to retrieve records that match a condition or fall outside a given range. This adds more control to your filtering logic.
```php
db()->bucket('post')
    ->where('status', 'published')
    ->orWhereNotBetween('views', [100, 500])
    ->get();
```
This query retrieves posts that are either published or have a view count less than 100 or greater than 500.

### whereNull()
In Doppar, the `whereNull()` method is used to filter records where a specific column has no value—i.e., it contains NULL. This is useful when you're identifying incomplete or pending data.

Example: find users without a created_at
```php
db()->bucket('users')
    ->whereNull('created_at')
    ->get();
```
This query retrieves all posts where the created_at field is NULL, which might indicate drafts or records that haven't been finalized yet.

### whereNotNull()
Conversely, `whereNotNull()` filters records where a given column does contain a value. It's ideal for fetching entries that are already completed or published.

Example: Fetch Published Posts
```php
db()->bucket('post')
    ->whereNotNull('published_at')
    ->get();
```
This retrieves all posts that have a value in the published_at field, meaning they’ve been published.

### orWhereNull()
The `orWhereNull()` method in Entity is used to add an OR condition to the query, checking if a column is NULL. In your example
```php
db()->bucket('post')
    ->where('status', 'draft')
    ->orWhereNull('reviewed_at')
    ->get();
```

### orWhereNotNull()
The `orWhereNotNull()` method in Entity is used to add an OR condition to the query, checking if a column is not NULL. In your example:
```php
db()->bucket('post')
    ->where('status', 'draft')
    ->orWhereNotNull('reviewed_at')
    ->get();
```

### DB::sql()
The `DB::sql()` method is used to insert raw SQL expressions into an Entity query. It’s useful when you need to perform calculations, use SQL functions, or alias complex expressions directly within a `select()`, `orderBy()`, or other query builder methods.

This is especially handy when Entity’s query builder doesn’t cover a specific SQL feature or function.

Here’s an example:

```php
use Phaseolies\Support\Facades\DB;

db()->bucket('employee')
    ->select(
        'name',
        'salary',
        DB::sql("ROUND(salary * 7.5 / 100, 2) as tax_amount")
    )
    ->get();
```
In this case, `DB::sql()` adds a computed column named `tax_amount` that represents 7.5% of each employee’s salary, rounded to two decimal places.

## Querying Date Columns

### whereDate()
The `whereDate()` method filters records by matching only the date part (`ignoring time`) of a column against a given value. It supports custom comparison operators.

Where date equals a specific date
```php
db()->bucket('users')
    ->whereDate('created_at', '2023-01-01')
    ->get();
```
Where date is greater than a specific date
```php
db()->bucket('users')
    ->whereDate('created_at', '>', '2023-01-01')
    ->get();
```

### whereMonth()
The `whereMonth()` method filters records by matching only the month part of a date or datetime column against a given value. The month can be given as a number (1 for January, 12 for December) and supports custom comparison operators.

Where month is January (month 1)
```php
db()->bucket('order')
    ->whereMonth('order_date', 1)
    ->get();
```

Where month is greater than March
```php
db()->bucket('order')
    ->whereMonth('order_date', '>', 3)
    ->get();
```

### whereYear()
The `whereYear()` method filters records by matching only the year part of a date or datetime column against a given value. Supports custom comparison operators.

Where year is 2023
```php
db()->bucket('posts')
    ->whereYear('published_at', 2023)
    ->get();
```

Where year is greater than 2020
```php
db()->bucket('posts')
    ->whereYear('published_at', '>', 2020)
    ->get();
```

### whereDay()
The `whereDay()` method filters records by matching only the day of the month (1–31) from a date or datetime column. Supports custom comparison operators.

Where day is the 15th
```php
db()->bucket('event')
    ->whereDay('event_date', 15)
    ->get();
```

Where day is less than 10
```php
db()->bucket('event')
    ->whereDay('event_date', '<', 10)
    ->get();
```

### whereTime()
The `whereTime()` method filters records by matching only the time part (`HH:MM:SS`) of a datetime or time column. Supports custom comparison operators.

Where time is after 14:00:00
```php
db()->bucket('appointment')
    ->whereTime('start_time', '>', '14:00:00')
    ->get();
```

Where time equals 09:30:00
```php
db()->bucket('appointment')
    ->whereTime('start_time', '09:30:00')
    ->get();
```

### whereToday()
The `whereToday()` method filters records where the date part of a column matches today’s date.

Records created today
```php
db()->bucket('order')
    ->whereToday('created_at')
    ->get();
```

### whereYesterday()
The `whereYesterday()` method filters records where the date part of a column matches yesterday’s date.

Records from last year
```php
db()->bucket('statistics')
    ->whereYesterday('recorded_at')
    ->get();
```

### whereThisMonth()
The `whereThisMonth()` method filters records where the month part of a column matches the current month.

Records from this month
```php
db()->bucket('sale')
    ->whereThisMonth('sale_date')
    ->get();
```

### whereLastMonth()
The `whereLastMonth()` method filters records where the month part of a column matches the previous month.

Records from last month
```php
db()->bucket('invoice')
    ->whereLastMonth('invoice_date')
    ->get();
```

### whereThisYear()
The `whereThisYear()` method filters records where the year part of a column matches the current year.

Records from this year
```php
db()->bucket('report')
    ->whereThisYear('report_date')
    ->get();
```

### whereLastYear()
The `whereLastYear()` method filters records where the year part of a column matches the previous year.

Records from last year
```php
db()->bucket('statistics')
    ->whereLastYear('recorded_at')
    ->get();
```

### whereDateBetween()
The `whereDateBetween()` method filters records where a date or datetime column falls between two given values (inclusive). By default, only the date part is compared. You can enable time comparison with the `$includeTime` parameter.

Between two dates (date only comparison)
```php
db()->bucket('result')
    ->whereDateBetween('test_date', '2023-01-01', '2023-01-31')
    ->get();
```

### whereDateTimeBetween()
The `whereDateTimeBetween()` method filters records where a datetime column falls between two given datetime values (inclusive of both boundaries). This method includes time comparison by default and is ideal for precise timestamp ranges.

Basic datetime range (inclusive of both boundaries)
```php
db()->bucket('users')
    ->whereDateTimeBetween('created_at', '2025-01-01 00:00:00', '2025-10-31 13:59:59')
    ->get();
```

Using DateTime objects
```php
$start = new DateTime('2025-01-01 00:00:00');
$end = new DateTime('2025-10-31 13:59:59');
db()->bucket('users')
    ->whereDateTimeBetween('created_at', $start, $end)
    ->get();
```

Date strings with automatic time handling. Start becomes '2025-01-01 00:00:00', End becomes '2025-01-31 23:59:59'
```php
db()->bucket('users')
    ->whereDateTimeBetween('created_at', '2025-01-01', '2025-01-31')
    ->get();
```

Specific time window within a day
```php
db()->bucket('users')
    ->whereDateTimeBetween('created_at', '2025-01-15 09:00:00', '2025-01-15 17:00:00')
    ->get();
```

## Nested conditions using callbacks
The `where()` and `orWhere()` methods allow you to define precise filtering conditions for your queries.
They can be chained for complex logical grouping, and both accept closures for nested conditions — providing expressive, readable query composition.

The following query fetches products that are active (status = true) and satisfy one of the following conditions:
- Have a price greater than 5000, or
- Have exactly 100 units in stock and belong to categories 8 or 9.
```php
db()->bucket('product')
    ->where('status', true)
    ->where(function ($query) {
        $query->where('price', '>', 5000)
            ->orWhere(function ($q) {
                $q->where('stock', 100)
                    ->whereIn('category_id', [8, 9]);
            });
    })
    ->get();
```
This structure produces a clean, readable way to express complex, nested conditions without worry.

## Conditional Query Execution
In Doppar, the `if()` method provides a powerful and elegant way to conditionally add query constraints based on a given condition. This allows you to build more flexible and dynamic queries, improving code readability and reducing unnecessary complexity.

Doppar ORM's `if()` method allows you to conditionally add query constraints based on a given condition. If the condition evaluates to true, the corresponding query modification is applied; otherwise, it is skipped.

Basic usage of `if()`
```php
db()->bucket('post')
    ->if($request->has('date'), function($query) use ($request) {
        $query->where('date', $request->has('date'));
    })
    ->get();
```
In this example, the `if()` method checks if the `date` is present in the request. If it is, the query will filter the posts based on the provided `date`. If not, the `where()` condition is skipped.

### Conditional Query Modifications with a Default Case
You can also provide a default case to be applied when the condition is false or not provided:
```php
db()->bucket('post')
    ->if($request->input('search'),
        // If search is provided, filter by title
        fn($q) => $q->where('title', 'LIKE', "%{$request->search}%"),

        // If no search is provided, filter by featured status
        fn($q) => $q->where('is_featured', '=', true)
    )
    ->get();
```
In this example:

- If a search parameter is provided, the query will search posts by title.
- If no search parameter is found, it defaults to filtering posts where is_featured is true.

### How if() Works with Different Conditions
Will execute when the condition is truthy:
```php
db()->bucket('post')
    // Executes because true
    ->if(true, fn($q) => $q->where('active', true))

    // Executes because 'text' is truthy
    ->if('text', fn($q) => $q->where('title', 'text'))

    // Executes because 1 is truthy
    ->if(1, fn($q) => $q->where('views', 1))

    // Executes because [1] is truthy
    ->if([1], fn($q) => $q->whereIn('id', [1]))
    ->get();
```

Will NOT execute when the condition is falsy:
```php
db()->bucket('post')
    // Does not execute because false
    ->if(false, fn($q) => $q->where('active', false))

    // Does not execute because 0 is falsy
    ->if(0, fn($q) => $q->where('views', 0))

    // Does not execute because empty string is falsy
    ->if('', fn($q) => $q->where('title', ''))

    // Does not execute because null is falsy
    ->if(null, fn($q) => $q->where('deleted_at', null))

    // Does not execute because empty array is falsy
    ->if([], fn($q) => $q->whereIn('id',  []))
    ->get();
```
This makes the `if()` method powerful for dynamically building queries based on various conditions. It allows for more concise and flexible query building without having to manually check each condition before applying the relevant query changes.

## Insertion and Update
### Inserts

Of course, when using Doppar, working with data isn't limited to just retrieving records — inserting new entries is just as important. Fortunately, Doppar makes this process straightforward. To add a new record to the database, just call the `insert()` method:
```php
db()->bucket('users')
    ->insert([
        'name' => 'John Doe',
        'email' => 'john@example.com',
        'password' => bcrypt('password'),
        'created_at' => now(),
        'updated_at' => now()
    ]);
```

This inserts a new row into the users table with the specified name and email. The `insert()` method takes care of generating and executing the appropriate SQL insert query behind the scenes, making data insertion clean and intuitive.

This is particularly useful when you need to ensure a record exists without accidentally updating existing data.

### Batch Insert
The `insertMany()` method allows you to insert multiple rows into a table efficiently — typically in a single query or in chunks (to avoid hitting database parameter limits).

Its signature:
```php
public function insertMany(array $rows, int $chunkSize = 100): int
```
Here’s a simple example inserting multiple posts:
```php
db()->bucket('posts')->insertMany([
    [
        'title' => 'Introducing Doppar',
        'status' => true,
        'author_id' => 1,
    ],
    [
        'title' => 'Entity Builder Deep Dive',
        'status' => true,
        'author_id' => 2,
    ],
    [
        'title' => 'Working with Multiple Connections',
        'status' => false,
        'author_id' => 3,
    ],
]);
```
If you have a very large dataset (e.g., 100000 records), you can use chunking to process inserts in batches:
```php
db()->bucket('logs')->insertMany($logData, 500);
```

### Updates
To update a record, modify the attributes you want to change, and then call `update()`. Doppar will handle generating the appropriate UPDATE statement.
```php
db()->bucket('users')
    ->where('id', 1)
    ->update([
        'name' => 'Jane Swinger',
        'updated_at' => now()
    ]);
```

## Upsert
Upsert (a combination of Insert and Update) is a database operation that inserts new records if they don't exist or updates existing records if they do. It's especially useful when you want to ensure data consistency without writing separate logic for checking existence first.

In SQL terms, this is typically handled using statements like INSERT ... ON `DUPLICATE KEY UPDATE (MySQL)`.

Basic Upsert by Unique Email
```php
$users = [
    [
        "email" => "mahedi@doppar.com",
        "name" => "Mahedi",
        "password" => bcrypt("password"),
    ],
    [
        "email" => "aliba@doppar.com",
        "name" => "Aaliba",
        "password" => bcrypt("password"),
    ],
];

$affectedRows = db()->bucket('users')
->upsert(
    $users,
    "email",  // Unique constraint
    ["name"], // Only update the "name" column if exists
    true      // Ignore errors (e.g., duplicate key)
);

return $affectedRows;
```

Result:
- Inserts new users if the email doesn't exist.
- Updates only the name if the user with that email already exists.

Upsert with Multiple Unique Keys
```php
$records = [
    [
        "email" => "unique@example.com",
        "phone" => "1234567890",
        "name" => "Update Me",
    ],
];

$affectedRows = db()->bucket('users')->upsert(
    $records,
    ["email", "phone"],  // Combination must be unique
    ["name"],            // Only update the name
    false
);
```
Result:
- Matches on both email and phone.
- If both match, updates name; otherwise inserts.

### Deleting Models
To delete a record, you may call the delete method like this way.
```php
db()->bucket('product')->where('id', 1)->delete();
```

## Aggregation
The Doppar ORM provides a rich set of aggregation functions that allow you to perform statistical and summary operations directly through your builder queries. These methods help extract meaningful insights from your data without writing raw SQL, making your code clean, expressive, and efficient.

### Total Sum of a Column
To calculate the sum of all values in a specific column (e.g., total views across all posts):
```php
db()->bucket('post')->sum('views');
```

### Average Value of a Column
To compute the average value of a column (e.g., average number of views):
```php
db()->bucket('post')->avg('views');
```

### Maximum Value in a Column
To get the highest value in a column (e.g., the most expensive product):
```php
db()->bucket('post')->max('price');
```

### Minimum Value in a Column
To get the lowest value in a column (e.g., cheapest product):
```php
db()->bucket('post')->min('price');
```

### Standard Deviation Calculation
To compute the standard deviation of values in a column (useful for measuring variability):
```php
db()->bucket('post')->stdDev('price');
```

### Variance Calculation
To calculate the variance of values (a squared measure of data spread):
```php
db()->bucket('post')->variance('price');
```

### Multiple Aggregations in One Query
To retrieve count, average, min, and max in a single grouped query:
```php
db()->bucket('product')
    ->select([
        'COUNT(*) as count',
        'AVG(price) as avg_price',
        'MIN(price) as min_price',
        'MAX(price) as max_price'
    ])
    ->groupBy('variants')
    ->first();
```
This groups data by variants and returns aggregated stats for each group.

### Fetching Distinct Rows
To get unique values from a column (e.g., all unique user IDs who posted):
```php
db()->bucket('posts')->distinct('user_id');
```

### Total Sales by Category
To aggregate sales per category by multiplying price and quantity:
```php
db()->bucket('sales')
    ->select(['category_id', 'SUM(price * quantity) as total_sales'])
    ->groupBy('category_id')
    ->get();
```

## Transform Entity Collection
The Doppar Builder offers expressive methods like `map()`, `filter()`, and `each()` to transform, filter, and iterate over Entity collections with ease. These methods provide fine control over the shape and content of your data after fetching it from the database.

### map() – Transforming Collection Items
Use the `map()` method to transform each item in a collection. This is especially useful when you want to reformat or limit the fields returned in your response.

Example: Return Only the name of Each User
```php
User::all()
    ->map(function ($item) {
        return [
            'name' => $item['name']
        ];
    });
```

> Use case: Customize API responses or prepare data for front-end consumption by trimming down unnecessary fields.

### `map()` with Property Shortcut
When you only need to access a property or call a method without arguments on each item, you can use the property shortcut syntax. This makes your code more concise:
```php
$users = db()->bucket('users')->get();

$names = $users->map->name;
```

The above code is equivalent to:
```php
$users->map(fn($user) => $user->name)
```

### filter() – Conditional Filtering
After transforming a collection, you can use filter() to return only items that match a given condition.

Example: return posts where `status = 1`, with only title and status
```php
db()->bucket('post')
    ->get()
    ->map(function ($item) {
        return [
            'title' => $item['title'],
            'status' => $item['status']
        ];
    })
    ->filter(function ($item) {
        return $item['status'] === 1;
    });
```

### each() – Iterating Through Items Without Changing Structure
The `each()` method is used to perform actions on each item in the collection without modifying the collection itself. It's great for side effects like logging, notifications, or conditional updates.

See a basic example
```php
db()->bucket('user')
    ->get()
    ->each(function ($user) {
        //
    });
```

These methods make Doppar’s Entity collection manipulation concise, readable, and highly adaptable for API responses, data formatting, and conditional logic. See more from [collections](collections).

## Pagination
Pagination is an essential feature for working with large datasets, enabling you to fetch and display data in manageable chunks. This improves both performance and user experience, especially in applications that deal with lists like users, posts, products, or logs.

Doppar makes pagination seamless through the built-in `paginate()` method, which handles all the behind-the-scenes logic for splitting your results into pages.

### Basic Usage
To paginate a dataset, simply call the `paginate()` method like as follows:
```php
db()->bucket('users')->paginate(1);
```

This will give you the response like this
```json
{
    "data": [
        {
            "id": 4,
            "name": "Caden Jerde",
            "email": "stokes.ludwig@hotmail.com",
            "created_at": "2025-05-06 09:59:57",
            "updated_at": "2025-05-06 09:59:57"
        }
    ],
    "first_page_url": "http://example.com/users?page=1",
    "last_page_url": "http://example.com/users?page=21",
    "next_page_url": "http://example.com/users?page=2",
    "previous_page_url": null,
    "path": "http://example.com/users",
    "from": 1,
    "to": 1,
    "total": 21,
    "per_page": 1,
    "current_page": 1,
    "last_page": 21
}
```

## Display Pagination in Views
When working with paginated data in your views, Doppar provides two convenient methods to render pagination links. These methods allow you to display navigation controls for moving between pages, ensuring a smooth user experience.

### Available Methods:
- **linkWithJumps()** method generates pagination links with additional "jump" options, such as dropdown with paging. It is ideal for datasets with a large number of pages, as it allows users to quickly navigate to the beginning or end of the paginated results.
- **links()** This method generates standard pagination links, including "Previous" and "Next" buttons, along with page numbers. It is suitable for most use cases and provides a clean and simple navigation interface.

Now call the pagination for views
```html
#foreach ($data['data'] as $user)
    <tr>
        <td>[[ $user->id ]]</td>
        <td>[[ $user->name ]]</td>
        <td>[[ $user->username ]]</td>
        <td>[[ $user->email ]]</td>
    </tr>
#endforeach

<!-- "Previous" and "Next" buttons, along with page numbers. -->
[!! paginator($data)->links() !!]

<!-- "Previous" and "Next" buttons, along with page jump options. -->
[!! paginator($data)->linkWithJumps() !!]
```

## Customize Default Pagination
Doppar provides a Bootstrap 5 pagination view by default. However, you can also customize this view to suit your needs. To customize the pagination view, Doppar offers the publish:pagination pool command. Running this command will create two files, `jump.Odo.php` and `number.Odo.php`, inside the `resources/views/vendor/pagination` folder. These files allow you to tailor the pagination design to match your application's style.

```bash
php pool publish:pagination
```

Once you modify the `jump.Odo.php` and `number.Odo.php` files, the changes will immediately reflect in your pagination view. This allows you to fully customize the appearance and behavior of the pagination links to align with your application's design and requirements. Feel free to update these files as needed to create a seamless and visually consistent user experience.

## Retrieving a Paginated Subset of Records
To fetch a specific subset of records from the database—such as a “page” of users—you can use the offset and limit methods on an Entity query.
```php
db()->bucket('users')
    ->offset(4)
    ->limit(10)
    ->get();
```
This query will skip the first 4 records and retrieve the next 10 records. This is useful for simple pagination or when you want to retrieve a specific slice of your dataset.

## Cursor Pagination
Cursor-based pagination is an efficient alternative to traditional offset pagination, especially for large datasets. Instead of using `OFFSET` and `LIMIT`, it uses a cursor — an encoded pointer to the last seen record — to fetch the next page. This avoids the performance degradation that comes with large offsets and ensures stable, consistent results even when data is being inserted or deleted between requests.

Doppar provides cursor pagination through the `cursorPaginate()` method, available on both Entity models and the raw query builder.

### Basic Usage
```php
User::cursorPaginate();
```

This will give you a response like this:
```json
{
    "data": [
        {
            "id": 1,
            "name": "John Doe",
            "email": "john@example.com",
            "created_at": "2024-01-01 10:00:00"
        },
        {
            "id": 2,
            "name": "Jane Smith",
            "email": "jane@example.com",
            "created_at": "2024-01-02 10:00:00"
        }
    ],
    "per_page": 15,
    "next_cursor": "eyJ2IjoyfQ==",
    "next_page_url": "http://example.com/users?cursor=eyJ2IjoyfQ==&per_page=15&direction=asc",
    "has_more": true,
    "direction": "asc"
}
```

### Paginating with Custom Options
You can pass custom options to the `cursorPaginate()` method, such as the number of records per page and the direction of pagination. Paginate 20 records per page in descending order
```php
User::cursorPaginate(20, 'id', 'desc');
```

### Fetching the Next Page
Pass the `next_cursor` value from the previous response to get the next page. In a controller, this typically looks like:
```php
public function index(Request $request)
{
    return User::query()
        ->where('status', 'active')
        ->cursorPaginate(
            perPage: 15,
            cursorColumn: 'id',
            direction: 'asc',
            cursor: $request->cursor
        );
}
```
The `next_page_url` in the response automatically carries the cursor, `per_page`, and direction parameters, so the client can simply follow the URL for the next page

> Note: The cursor column should be indexed and have reasonably unique values (such as id or created_at) to ensure correct and efficient pagination. Cursor pagination does not support jumping to arbitrary pages — it is strictly sequential forward pagination.

<a name="database-transactions"></a>

## Database Transactions
A database transaction is a sequence of database operations that are executed as a single unit. Transactions ensure data integrity by following the ACID properties (Atomicity, Consistency, Isolation, Durability). If any operation within the transaction fails, the entire transaction is rolled back, preventing partial updates that could leave the database in an inconsistent state.

Doppar provides built-in support for handling database transactions using `#[Transaction]` attribute and the `DB::transaction()` method, `DB::beginTransaction()`, `DB::commit()`, and `DB::rollBack()`.

You can manage transactions directly using the `DB` facade. This method gives you complete control over when to begin, commit, or roll back.

## Attribute-Based Transaction
The `#[Transaction]` attribute provides automatic database transaction management for doppar controller methods. It eliminates the need for manual transaction handling and ensures data consistency. You can simplify transaction management with the `#[Transaction]` attribute. This approach automatically wraps your controller methods in a transaction — no manual calls required.

Basic usage
```php
use Phaseolies\Utilities\Attributes\Transaction;

#[Transaction]
#[Route(uri: 'orders', methods: ['POST'])]
public function store(Request $request)
{
    // store method automatically wrapped in transactions
}
```

### Specify database connection
You can target a specific database connection. Specify which database connection to use:
```php
#[Transaction(connection: 'mysql')]
#[Transaction(connection: 'pgsql')]
#[Transaction(connection: 'sqlite')]
```

If not specified, uses the default connection from `config/database.php`.

### Retry Attempts
You can also configure automatic retries for transient failures or deadlocks.
```php
#[Transaction(attempts: 3)]  // Retry up to 3 times
#[Transaction(attempts: 5)]  // Retry up to 5 times
```

### Combined Configuration
You can use connention and attempts in your `#[Transaction]` attribute
```php
class OrderController extends controller
{
    #[Transaction(connection: 'pgsql', attempts: 3)]
    #[Route(uri: 'orders', methods: ['POST'])]
    public function store()
    {
        // pgsql connection + default 3 attempts
    }

    #[Transaction(connection: 'mysql', attempts: 3)]
    #[Route(uri: 'orders/{id}', methods: ['PUT'])]
    public function update()
    {
        // mysql connection + default 3 attempts
    }
}
```

This flexible configuration system makes it easy to enforce consistent transaction behavior across your entire controller
while retaining the ability to fine-tune individual operations for different use cases.

### Using `DB::transaction()` for Simplicity
The `DB::transaction()` method automatically handles committing the transaction if no exception occurs and rolls it back if an exception is thrown.
```php
use Phaseolies\Support\Facades\DB

DB::transaction(function () {
    //
});
```

### Manually Handling Transactions
In cases where more control is needed, transactions can be manually started using `DB::beginTransaction()`. The operations must then be explicitly committed or rolled back.
```php
DB::beginTransaction();
try {
    // Place your database operations here
    // For example: creating, updating, or deleting records.
    // If everything inside this block executes successfully,
    // the changes will be committed (saved) to the database.
    // Commit the transaction (permanently save changes)

    DB::commit();
} catch (\Exception $e) {
    // If an exception occurs within the try block,
    // all changes made during this transaction will be rolled back.
    DB::rollBack();
}
```

## Handling Deadlocks with Transaction Retries
Deadlocks can occur when multiple transactions compete for the same database resources. Doppar allows setting a retry limit for transactions using a second parameter in DB::transaction().
```php
DB::transaction(function () {
    // Operations that might deadlock
}, 3);
```
Will attempt up to 3 times before throwing an exception. This approach helps mitigate issues caused by deadlocks by retrying the transaction a set number of times before ultimately failing.

Using transactions properly ensures database consistency and prevents data corruption due to incomplete operations. Doppar provides flexible methods for handling transactions, allowing both automatic and manual control based on the use case.

<a name="entity-join"></a>

## Entity Join
In Doppar, Entity manual joins allow you to retrieve data from multiple tables based on a related column. The join method in Entity's Query Builder provides an easy way to combine records from different tables. This document explains how to perform various types of joins manually using Entity's Entity ORM and Query Builder.

#### Basic Join Example
A simple join operation can be performed using the join method to combine records from two tables based on a common key. Below is an example of joining users and posts tables:
```php
db()->bucket('post')
    ->select('post.*', 'users.name as user_name')
    ->join('users', 'users.id', '=', 'post.user_id')
    ->get();
```

This will return a dataset containing user data which has at least one post.

### Specifying Join Type
By default, Doppar performs an `INNER JOIN`. You can specify the type of join you want by passing the join type as an argument:
```php
db()->bucket('users')
    ->join('posts', 'users.id', '=', 'posts.user_id', 'left')
    ->get();
```

Here, a `LEFT JOIN` is used to include all users, even if they do not have associated posts.

### Performing Multiple Joins
You can join multiple tables in a single query. The example below joins the users, posts, and comments tables:
```php
db()->bucket('users')
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->join('comments', 'posts.id', '=', 'comments.post_id')
    ->get();
```
This will return data containing users, their posts, and associated comments.

### Selecting Specific Columns
To optimize queries and improve performance, you can select specific columns instead of retrieving all fields:

```php
db()->bucket('users')
    ->select('users.name', 'posts.title')
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

This query retrieves only the users.name and posts.title fields, reducing the amount of data transferred.

## DB Query
Doppar’s DB Facade, available via the `Phaseolies\Support\Facades\DB` namespace, provides a clean, expressive, and powerful interface to interact directly with your database—going beyond the ORM layer when you need low-level control or raw SQL flexibility.

Whether you're running raw queries, managing transactions, executing stored procedures, or querying database views, Doppar gives you a full toolkit under the DB facade.

#### Basic Database Utilities
| Operation              | Example                       | Description                                         |
| ---------------------- | ----------------------------- | --------------------------------------------------- |
| Get all tables         | `DB::getTables()`             | Returns an array of all table names in the database |
| Check table existence  | `DB::tableExists('user')`     | Returns `true` if the specified table exists        |
| Get table columns      | `DB::getTableColumns('user')` | Returns a list of column names from a table         |
| Get table from model   | `DB::getTable(new User())`    | Fetches the associated table name of a model        |
| Get raw PDO connection | `DB::getConnection()`         | Returns the underlying PDO object                   |

### Querying the Database with query()
The `query()` method provided by Doppar enables you to execute raw SQL statements with parameter binding to ensure safe and secure queries. This method returns a `\Phaseolies\Support\Collection` giving you full control over result handling.

The following example demonstrates how to execute a parameterized SQL query and retrieve the first matching row as an associative array:
```php
use Phaseolies\Support\Facades\DB

$user = DB::query("SELECT * FROM users WHERE name = ?", ['Aliba']);

$user->name;
```
This will return the first row where the name column matches "Kasper Snider", or null if no such row exists.

#### Get All Rows
In this example, all rows from the `users` table are retrieved and stored in the `$users` collection. If there are no matching rows, an empty array will be returned.
```php
$users = DB::query("SELECT * FROM users");
```

The `query()` method returns a Collection, allowing you to fluently chain collection methods like `map()`, `filter()`, and others. This makes it easy to transform or manipulate the result set directly after querying the database.

If you expect multiple rows, you can use the `map()` method to transform each row into a custom structure or format.
```php
<?php

use Phaseolies\Support\Facades\DB;

DB::query("SELECT * FROM users")
    ->map(function ($data) {
        return [
            'name' => $data['name'] ?? 'default',
        ];
    });
```
### Modifying the Database using execute()
Doppar provides `execute()` function and using it, you can modify database. See the basic example. Inserts a new user record. Returns the number of affected rows (1 if successful).
```php
DB::execute(
    "INSERT INTO users (name, email, password) VALUES (?, ?, ?)",
    ['John Doe', 'john@example.com', bcrypt('secret')]
);
```

You can use transaction here as well

```php
DB::transaction(function () {
    $moved = DB::execute(
        "INSERT INTO archived_posts SELECT * FROM posts WHERE created_at < ?",
        [date('Y-m-d', strtotime('-1 year'))]
    );

    $deleted = DB::execute(
        "DELETE FROM posts WHERE created_at < ?",
        [date('Y-m-d', strtotime('-1 year'))]
    );

    echo "Archived {$moved} posts and deleted {$deleted} originals";
});
```

Executes multiple statements in a single transaction. Ensures either all queries succeed or none are committed.

## Executing Stored Procedures
Doppar provides support for executing stored procedures through the `procedure()` method. This allows you to call database-stored routines directly, making it ideal for complex logic encapsulated within the database.

By default, `procedure()` returns a `\Phaseolies\Support\Collection` results.
```php
$nestedResults = DB::procedure('sp_GetAllUsers')->all();
```

This will return the all users collection data.

#### Get first row
When you only need the first row from the first result set of a stored procedure, you can use the `first()` method after calling `procedure()`. This is useful for cases where the procedure returns a list, but you only care about the top result.
```php
$firstUser = DB::procedure('sp_GetAllUsers')->first();

// with passing parameter
$firstUser = DB::procedure('sp_GetAllUsers', [1])->first();
```

#### Get last row
When you only need the last row from the first result set of a stored procedure, you can use the `last()` method after calling `procedure()`. This is useful for cases where the procedure returns a list, but you only care about the last result.
```php
$firstUser = DB::procedure('sp_GetAllUsers')->last();
```

Get the second result set (index 1) by passing 123 parameter
```php
$stats = DB::procedure('sp_GetUserWithStats', [123])->resultSet(1);
```

#### With Multiple Params
Doppar's `procedure()` method supports stored procedures that return multiple result sets. The returned result object offers convenient methods to navigate and extract specific parts of the output.
```php
$result = DB::procedure('get_multiple_results');
$result->resultSet(0); // First set
$result->resultSet(1); // Second set

// Get last row of first result set
$lastRow = $result->last();

// Get last row of second result set
$lastRowSet2 = $result->lastSet(1);
```

You can also get procedure result like this way
```php
DB::query("CALL get_multiple_results");

// ⚠️ Important Note:
// When calling stored procedures this way,
// the usual helpers like `resultSet()`, `last()`, or `lastSet()`
// are NOT available. Instead, you'll get the raw query result.
```

> The `DB::procedure()` method is available only for MySQ

#### Executing View
Doppar makes it easy to query database views using the `view()` method. A view behaves like a virtual table and can be queried just like a regular table, often used to simplify complex joins or aggregations.
```php
$stats = DB::view('vw_user_statistics');
```

#### View with WHERE Conditions
You can pass where condition in your custom view like this way. View with single WHERE condition
```php
$nyUsers = DB::view(
    'vw_user_locations', 
    ['state' => 'New York']
);
```
This is equivalent to:
```sql
SELECT * FROM vw_user_locations WHERE state = 'New York'
```

View with multiple WHERE conditions
```php
$premiumNyUsers = DB::view(
    'vw_user_locations',
    [
        'state' => 'New York',
        'account_type' => 'premium'
    ]
);
```

This is equivalent to: 
```sql
SELECT * FROM vw_user_locations  WHERE state = 'New York' AND account_type = 'premium'
``` 

### View with Parameter Binding
You can also pass params with where condition as follows. Let's see the example of using parameter binding for security
```php
$recentOrders = DB::view(
    'vw_recent_orders',
    ['status' => 'completed'],
    [':min_amount' => 100] // Additional parameters
);
```

This is equivalent to:
```sql
SELECT * FROM vw_recent_orders WHERE status = 'completed' AND amount > :min_amount
```

### Summery
| Feature           | Method                         | Purpose                                      |
| ----------------- | ------------------------------ | -------------------------------------------- |
| Raw Queries       | `query()`                      | Run SELECT statements with parameter binding |
| Data Modification | `execute()`                    | Run INSERT, UPDATE, DELETE                   |
| Transactions      | `transaction()`                | Execute multiple queries safely              |
| Stored Procedures | `procedure()`                  | Call stored procs with multiple results      |
| Views             | `view()`                       | Query DB views with filters and bindings     |
| Utilities         | `getTables()`, `tableExists()` | Explore schema                               |

Doppar's DB Facade provides all the raw SQL power you need while preserving the developer-friendly syntax you love.

## Handling Multiple Database Connection
Doppar allows you to manage multiple database connections seamlessly. This is especially useful when working with different databases for various modules (e.g., separating reporting, user data, or third-party integrations).

By default, Doppar uses the primary database connection defined in your configuration `(mysql)`. But you can easily switch between connections using `connection()` method. Below are various ways to interact with a secondary connection `(like mysql_second)`.

### Define the Secondary Connection
In your `config/database.php`, define a new connection named mysql_second:
```php
'mysql_second' => [
    'driver' => 'mysql',
    'host' => env('DB_SECOND_HOST', '127.0.0.1'),
    'port' => env('DB_SECOND_PORT', '3306'),
    'database' => env('DB_SECOND_DATABASE', 'doppar_second'),
    'username' => env('DB_SECOND_USERNAME', 'root'),
    'password' => env('DB_SECOND_PASSWORD', ''),
    'charset' => 'utf8mb4',
    'collation' => 'utf8mb4_unicode_ci',
    'prefix' => '',
    'strict' => true,
    'engine' => null,
],
```

Make sure your `.env` file contains the appropriate credentials:
```php
DB_SECOND_HOST=127.0.0.1
DB_SECOND_PORT=3306
DB_SECOND_DATABASE=doppar_second
DB_SECOND_USERNAME=root
DB_SECOND_PASSWORD=
```

### Querying with Multiple Connections
In real-world applications — especially in large-scale or modular systems — it’s common to distribute data across multiple databases. For instance, you might separate reporting data from transactional data, or isolate client data in a multi-tenant architecture.

Doppar Entity Builder makes working with multiple database connections seamless. You can easily target different connections either statically or dynamically at runtime, ensuring maximum flexibility without adding complexity.

```php
db()->connection('mysql_second')
    ->bucket('users')
    ->get();
```
This tells the doppar to run the query on the `mysql_second` connection instead of the default one. Perfect for pulling data from a secondary database on the fly.

## `DB` Facade with Specific Connection
In Doppar, if you're working with multiple databases, you don't always need to use models. Sometimes you might want to perform database operations directly using the query builder, which is both powerful and flexible.

To do this on a specific database connection, Doppar lets you chain `connection('mysql_second')` before calling any query builder method. This allows you to use the full capabilities of `DB` facades query on a different database.

Below is a basic example showing how you can use various methods on a specific connection using Doppar’s DB facade:

Run a simple query on the 'mysql_second' connection
```php
use Phaseolies\Support\Facades\DB;

DB::connection('mysql_second')->query('SELECT * FROM reports');
```

Get all table names from the 'mysql_second' database
```php
DB::connection('mysql_second')->getTables();
```

Check if a specific table exists in the secondary connection
```php
DB::connection('mysql_second')->tableExists('user');
```

Access the raw PDO connection for lower-level operations
```php
DB::connection('mysql_second')->getConnection();
```
