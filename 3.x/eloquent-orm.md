---
title: Eloquent ORM
description: Doppar Eloquent ORM page
meta:
  - name: keywords
    content: Eloquent ORM
---

  - [Getting Started](#getting-started)
  - [Query Using ORM](#query-using-orm)
  - [Query Binding](#query-binding)
  - [Insertion and Update](#insertion-and-update)
  - [Aggregation](#aggregation)
  - [Querying JSON Columns](#querying-json-columns)
  - [Querying Date Columns](#querying-date-columns)
  - [Transform Eloquent Collection](#transform-eloquent-collection)
  - [Pagination](#pagination)
  - [Database Transactions](#database-transactions)
  - [Eloquent Join](#eloquent-join)
  - [DB Query](#db-query)
  - [Handling Large Dataset](#handling-large-dataset)
  - [Handling Multiple Database Connection](#handling-multiple-database-connection)

## Eloquent
### Introduction
Doppar features its own powerful data interaction tool, Eloquent, an object-relational mapper (ORM), which simplifies and enhances the way you work with your database. With ORM, every database table is linked to a dedicated "Data Model" that serves as your gateway to managing table data seamlessly. Beyond just fetching records, Doppar Data Mapper empowers you to effortlessly insert, update, and delete records, making database interactions intuitive and efficient. Whether you're building complex queries or handling simple data operations, ORM ensures a smooth and enjoyable experience, tailored to streamline your development workflow.

## Query Using ORM
### Retrieving Models
Once you have created a model and its associated database table, you are ready to start retrieving data from your database. You can think of each Eloquent model as a powerful query builder allowing you to fluently query the database table associated with the model. The model's all method will retrieve all of the records from the model's associated database table:
```php
<?php

use App\Models\Post;

foreach (Post::all() as $post) {
    echo $post->title;
}
```
You can also fetch all data using direct `get()` method like `Post::get()`.

### Expending Queries
The all method in Eloquent fetches every record from a model’s corresponding table. However, because Eloquent models double as query builders, you can refine queries using conditions and retrieve only the matching results by calling the get method.
```php
Post::where('status', true)
    ->orderBy('title')
    ->limit(10)
    ->get();
```

You can write query with multiple where condition by chaining multiple where with the queries
```php
Post::where('title', 'Sincere Littel')
    ->where('status', 1)
    ->first();
```

## Query Debugging
You can inspect queries before execution by chaining debugging methods before `get()`:
```php
User::limit(10)->debug();
```

You will see the response like this
```json
{
    "sql": "SELECT * FROM users LIMIT 10",
    "bindings": [],
    "select": [
    "*"
    ],
    "where": [],
    "order": [],
    "group": [],
    "limit": 10,
    "offset": null,
    "joins": [],
    "eager_load": []
}
```

Available Debug Methods:
- `->debug()` – Logs query, bindings, and execution time.
- `->explain()` – Runs SQL EXPLAIN for query analysis.
- `->dumpSql()` – Outputs the raw SQL string.
- `->ddSql()` – Dumps and halts with the raw SQL.

> Note: These methods are non-intrusive and designed for development use only.

### Measure Memory Usage
You can measure memory consumption of a query result by chaining `->withMemoryUsage()` after data retrieval:
```php
User::limit(10)->get()->withMemoryUsage();

// Memory usage: 2 MB, Peak: 2 MB
```
What It Does:
- Outputs the memory used to fetch and hold the result.
- Helps profile large datasets or optimize memory-sensitive operations.

> Note: These methods are non-intrusive and designed for development use only.

### Collections
As observed, Eloquent methods such as `all` and `get` are designed to fetch multiple records from the database. Rather than returning a standard PHP array, these methods yield an instance of `Phaseolies\Support\Collection`, providing a rich set of tools to work with the retrieved models efficiently.

If you want to convert collection to array, you can use `toArray()` method.
```php
Post::where('status', true)
    ->orderBy('title')
    ->limit(10)
    ->get()
    ->toArray();
```

### Dynamic `where` Conditions
Doppar supports dynamic query building using expressive method calls, allowing you to easily filter results using field names directly in the method. like
```php
User::whereName('Nure')->get();
```
This is equivalent to:
```php
User::where('name', 'Nure')->get();
```
The method name must follow `camelCase` or `StudlyCase` based on the column name. For example:
```php
User::whereEmail('user@example.com')
    ->whereCountryCode('880')
    ->get();
```
Only works with existing columns in your model's table.

### Fetch the First Records
To fetch the first records you can use `first()` function as like below
```php
Post::first();
```

Actually this above query returns the Post model object so you can directly fetch its attributes like
```php
Post::first()?->title ?? 'defult';
```

### GroupBy and OrderBy
You can also use `groupBy()` and `orderBy()` to handle your records like
```php
User::orderBy('id', 'desc')->groupBy('name')->get();
```

### random()
The `random()` method in Doppar's Eloquent ORM extension allows you to retrieve a random subset of records from the database. It's particularly useful when you want to display randomized content, such as featured products, suggested users, or shuffled items in a feed.

Internally, `random()` applies an `ORDER BY RAND()` clause to the query and limits the result set using the provided number.

Here's an example:
```php
User::random(10)->get();
```
This will return 10 random users from the users table.

### toSql()
The toSql() method in Eloquent is a useful tool when you want to inspect the raw SQL query that would be executed for a given Eloquent query. Instead of retrieving the results from the database, toSql() returns the SQL statement as a string, allowing you to debug or log the query before it's run. This is especially helpful during development when you need to understand how your query builder chain is being translated into SQL.

Here's an example:
```php
User::where('status', 'active')->toSql();
```
This will output:
```sql
select * from `users` where `status` = ?
```
Using `toSql()` helps you optimize and troubleshoot queries by giving insight into what Doppar is sending to the database under the hood.

### find()
The `find()` method in Eloquent is used to retrieve a single record from the database by its primary key. It offers a quick and efficient way to look up a specific model instance without having to write a full where clause. If a matching record is found, an instance of the model is returned; otherwise, null is returned.

Here’s a basic example:
```php
User::find(1);
```
In this case, Eloquent returns a collection of User instances corresponding to the given IDs. find() is ideal when you know the exact primary key(s) you're looking for and want a concise way to retrieve the corresponding record(s).

This is the sort version of this query
```php
User::where('id', 1)->first();
```

You can also pass multiple primary key to fetch data like this way
```php
User::find([1, 2, 3]);
```

### count()
To see how many records your table has, you can use count() function as like
```php
User::count();
User::orderBy('id', 'desc')->groupBy('name')->count();
User::where('active', true)->count();
```

### Newest and Oldest Records
Doppar makes it convenient to retrieve the most recent or earliest records from a database using the newest() and oldest() query methods. These methods sort the results based on a specified column, with id being the default field if none is provided.

To fetch the latest records (in descending order):
```php
User::newest()->get();
```
This retrieves the most recently added users based on the id column by default. If you want to sort by a different column, you can pass it as an argument:
```php
User::newest('name')->get();
```
Similarly, to retrieve the oldest records (in ascending order):
```php
User::oldest()->get();
```
Again, you can specify a custom column to determine what "oldest" means in your context:
```php
User::oldest('id')->get();
```
These methods offer a clean and readable way to sort query results by time or any relevant field, enhancing code clarity while maintaining flexibility.

### Selecting Specific Columns
In many cases, you may not need to retrieve every column from a table—especially when working with large datasets. Eloquent's `select()` method allows you to specify exactly which columns you want to fetch, helping optimize performance and reduce memory usage.

Here are a few examples:
```php
// Selecting specific columns using an array
User::select(['name', 'email'])->get();

// Selecting specific columns using multiple arguments
User::select('name', 'email')->get();
```

Both of these will return only the name and email fields for each user, excluding all other columns.

You can also use select() in more advanced queries, such as when grouping results:
```php
Post::query()
    ->select('title', 'COUNT(*) as count')
    ->groupBy('category_id')
    ->get();
```
Using select() helps tailor your queries to fetch only the data you actually need, improving both efficiency and clarity in your code.


### Excludes specific columns
Excludes specific columns from the `SELECT` query. This method is useful when you want to retrieve all columns except certain ones, such as timestamp or metadata fields.
```php
Post::omit('created_at', 'updated_at', 'excerpt')->get();
```
In the example above, all columns will be selected except `created_at` and `updated_at`.
You can also pass an array instead of variadic arguments:
```php
Post::omit(['deleted_at', 'archived_at'])->get();
```
If the query initially selects all columns using *, `omit()` will first resolve the actual column names via the model's table schema and then exclude the specified ones.

## selectRaw()
The selectRaw() method in Doppar's query builder provides a powerful way to include raw SQL expressions in your queries. It's especially useful when you need to perform calculations, use SQL functions, or include complex expressions that go beyond Eloquent's default capabilities.

### Basic Usage

You can use selectRaw() to write a custom SQL expression directly in the SELECT clause. For example, to count the total number of orders:

```php
use App\Models\Order;

$total = Order::query()
    ->selectRaw('COUNT(*) as order_count')
    ->first();

echo $total->order_count;
```
This will execute a query similar to:
```sql
SELECT COUNT(*) as order_count FROM orders;
```

### Calculated Columns
You can also perform calculations using existing fields in the database. For instance, to calculate the total value of each order:
```php
$orders = Order::query()
    ->selectRaw('price * quantity as total_value')
    ->get();

foreach ($orders as $order) {
    echo $order->total_value;
}
```
This calculates a total_value field on the fly for each record using the price and quantity columns.

### Chaining Multiple selectRaw() Calls
You can chain multiple selectRaw() calls to build a SELECT clause that includes several raw expressions. Each call adds to the final SQL:
```php
Order::query()
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
Order::query()
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

### whereRaw()
The `whereRaw()` method allows you to write a raw SQL WHERE clause directly into an Eloquent query. This is useful when your query requires SQL features that aren't easily expressed using Eloquent's fluent methods — such as complex conditions, custom SQL functions, or case-insensitive matching.

You pass a raw SQL string as the first argument, and an optional array of bindings as the second. This helps maintain parameter binding and protects against SQL injection.

Here’s an example:
```php
User::query()
    ->whereRaw('LOWER(name) LIKE LOWER(?)', ['%jewel%'])
    ->get();
```
This query returns all users whose name matches “jewel” in a case-insensitive way, by applying the `LOWER()` SQL function to both the column and the search term.

### orderByRaw()
The `orderByRaw()` method allows you to write a raw SQL ORDER BY clause directly into an Eloquent query. This is useful when you need advanced sorting logic that can't be easily represented using Eloquent’s fluent orderBy() method — such as ordering by multiple columns with different sort directions or using SQL functions.

You pass a raw SQL string as the first argument, which defines the custom sorting logic. Because the input is raw SQL, it gives you full control over the order clause — but it’s important to ensure the syntax is correct and secure.

Here’s an example:
```php
Employee::query()
    ->orderByRaw('salary DESC, name ASC')
    ->get();
```

Use orderByRaw() when your ordering requirements exceed the capabilities of standard Eloquent methods.

### groupByRaw()
The `groupByRaw()` method allows you to define a raw SQL GROUP BY clause in an Eloquent query. It’s especially useful when you need to group results by SQL functions or multiple columns that can't be easily handled using Doppar’s fluent groupBy() method.

You pass a raw SQL string as the argument, which gives you full control over the grouping logic.

Here’s an example:
```php
User::query()
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
To check whether a specific row exists in your database, you can use the exists() function. This method returns a boolean value (true or false) based on whether the specified condition matches any records. Here's an example:
```php
User::where('id', 1)->exists();

// Returns `true` if a matching row exists, otherwise `false`.
```

### whereIn()
The whereIn() method filters records where a column's value matches any value in the given array.
```php
User::whereIn('id', [1, 2, 4])->get();
```
This retrieves all users with id values of 1, 2, or 4.

### orWhereIn()
The orWhereIn() method filters records where a column's value optionally matches any value in the given array.
```php
User::orWhereIn('id', [1, 2, 4])->get();
```

### whereBetween()
The whereBetween() method in Doppar lets you filter records where a column’s value falls within a specified range. It’s particularly useful for working with numeric values or date/time columns, such as filtering records created within a certain time frame.

#### Example: Filter Users by Date Range
```php
User::query()
    ->whereBetween('created_at', ['2025-02-29', '2025-04-29'])
    ->get();
```
This query will retrieve all users whose created_at timestamp falls between February 29, 2025 and April 29, 2025, inclusive.

You can use whereBetween() with any comparable column — such as prices, IDs, scores, or dates — to make your queries more expressive and efficient.
```php
User::whereBetween('id', [1, 10])->get();
```
This query retrieves all user records where the id is between 1 and 10, inclusive. That means it will return users with id values of 1, 2, 3, ..., 10.

### whereNotBetween()
The whereNotBetween() method allows you to filter records where a given column's value do not falls within a specified range. This is commonly used for date or numerical ranges.
```php
User::query()
    ->whereNotBetween('created_at', ['2025-02-29', '2025-04-29'])
    ->get();
```

### orWhereBetween()
Doppar allows you to construct expressive, flexible queries using methods like orWhereBetween(). This method is particularly useful when you want to retrieve records that match either a specific condition or fall within a certain range.

Example: Combine exact match with a range
```php
Post::query()
    ->where('status', 'published')
    ->orWhereBetween('views', [100, 500])
    ->get();
```
This will return all posts that are either published or have between 100 and 500 views, inclusive.

### orWhereNotBetween()
Similarly, orWhereNotBetween() is used when you want to retrieve records that match a condition or fall outside a given range. This adds more control to your filtering logic.
```php
Post::query()
    ->where('status', 'published')
    ->orWhereNotBetween('views', [100, 500])
    ->get();
```
This query retrieves posts that are either published or have a view count less than 100 or greater than 500.

### whereNull()
In Doppar, the whereNull() method is used to filter records where a specific column has no value—i.e., it contains NULL. This is useful when you're identifying incomplete or pending data.

Example: Find Posts Without a Created Date
```php
Post::query()
    ->whereNull('created_at')
    ->get();
```
This query retrieves all posts where the created_at field is NULL, which might indicate drafts or records that haven't been finalized yet.

### whereNotNull()
Conversely, whereNotNull() filters records where a given column does contain a value. It's ideal for fetching entries that are already completed or published.

Example: Fetch Published Posts
```php
Post::query()
    ->whereNotNull('published_at')
    ->get();
```
This retrieves all posts that have a value in the published_at field, meaning they’ve been published.

### orWhereNull()
The orWhereNull() method in Eloquent is used to add an OR condition to the query, checking if a column is NULL. In your example
```php
Post::query()
    ->where('status', 'draft')
    ->orWhereNull('reviewed_at')
    ->get();
```

### orWhereNotNull()
The orWhereNotNull() method in Eloquent is used to add an OR condition to the query, checking if a column is not NULL. In your example:
```php
Post::query()
    ->where('status', 'draft')
    ->orWhereNotNull('reviewed_at')
    ->get();
```

### pluck()
The pluck() method in Doppar is a convenient way to extract the values of a single or double column from your query results. Instead of returning full model instances, pluck() pulls out just the values from the specified field—perfect for generating simple lists.

Example: Get All Post Titles
```php
Post::query()->pluck('title');
```
This will return a collection of all titles from the posts table, such as:
```php
[
    "How to Use Doppar",
    "Getting Started with PHP",
    "Understanding Eloquent Queries"
]
```

### DB::sql()
The `DB::sql()` method is used to insert raw SQL expressions into an Eloquent query. It’s useful when you need to perform calculations, use SQL functions, or alias complex expressions directly within a select(), orderBy(), or other query builder methods.

This is especially handy when Eloquent’s query builder doesn’t cover a specific SQL feature or function.

Here’s an example:
```php
use Phaseolies\Support\Facades\DB;

Employee::query()
    ->select(
        'name',
        'salary',
        DB::sql("ROUND(salary * 7.5 / 100, 2) as tax_amount")
    )
    ->get();
```
In this case, `DB::sql()` adds a computed column named `tax_amount` that represents 7.5% of each employee’s salary, rounded to two decimal places.

You can use DB::sql() to extract, filter, or unquote JSON fields stored in your database. Here’s an example that extracts a nested value from a JSON column:
```php
User::query()
    ->select(
        'name',
        DB::sql("JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.preferences.notifications.email')) as email_notifications_enabled")
    )
    ->get();
```
In this case, the `attributes` column is a JSON object, and the query extracts the value at the path `preferences.notifications.email`, returning it as `email_notifications_enabled`.

### toDictionary()
The `toDictionary()` method executes the query and returns the results as a simple key-value dictionary. This is useful when you need a flat, associative representation of a dataset — for example, populating dropdowns, lookups, or mapping IDs to labels.

You specify which column to use as the keys and which column to use as the values.

Here’s an example:
```php
User::toDictionary('id', 'name');
```
This returns a collection like:
```php
[
    '1' => 'Alice',
    '2' => 'Bob',
    '3' => 'Charlie',
]
```
Internally, it selects only the specified key and value columns, iterates through the results, and builds a dictionary-style collection. This can be more memory-efficient than returning full model instances when you only need specific columns.

### toDiff()
The `toDiff()` method calculates the difference between two numeric columns in your query and returns the result with a custom alias. This is especially useful when you want to compute values like profit, margin, balance, or other derived metrics directly within the database query.

You provide two column names, and an optional alias for the resulting column.

Here’s an example:
```php
$profitMargins = Product::toDiff('price', 'cost', 'profit');
```
This query returns a collection of results where each item includes a single `column: profit`, which is the result of `price - cost`. Internally, the method generates a raw SQL expression like `price - cost as profit` and passes it to the query’s select() clause.

If no alias is provided, the result defaults to difference:
```php
Product::toDiff('revenue', 'expenses');
```
Returns:
```php
[
    ['difference' => 1200],
    ['difference' => 875],
    // ...
]
```
### toTree()
The `toTree()` method transforms a flat list of records into a nested tree structure based on parent-child relationships. This is especially useful for hierarchical data like categories, menus, comments, or organizational units.

You provide the name of the primary key, the parent key column, and an optional children key to nest the relationships.

Here’s an example:
```php
$categoryTree = Category::toTree('id', 'parent_id', 'children');
```
This will return a collection of top-level categories, each containing a `children` relation recursively filled with their respective subcategories.

Each item is recursively nested using the specified keys. The default key used to store children is children, but you can customize this as needed.

Structure:

If your flat data looks like this:
```php
[
    ['id' => 1, 'name' => 'Electronics', 'parent_id' => null],
    ['id' => 2, 'name' => 'Laptops', 'parent_id' => 1],
    ['id' => 3, 'name' => 'Phones', 'parent_id' => 1],
    ['id' => 4, 'name' => 'Gaming', 'parent_id' => 2],
]
```

`toTree()` will convert it into:
```json
[
  {
    "id": 1,
    "name": "Electronics",
    "children": [
      {
        "id": 2,
        "name": "Laptops",
        "children": [
          {
            "id": 4,
            "name": "Gaming",
            "children": []
          }
        ]
      },
      {
        "id": 3,
        "name": "Phones",
        "children": []
      }
    ]
  }
]
```
It automatically guards against circular references, throwing an exception if one is detected.

### toRatio()
The `toRatio()` method calculates the ratio between two numeric columns and returns the result with a custom alias. This is particularly useful when you're tracking progress, rates, or percentages — such as completion rate, conversion rate, or usage metrics.

You provide the numerator and denominator column names, along with an optional alias and decimal precision for rounding.

Here’s an example:
```php
$completionRatios = Project::query()
    ->toRatio(
        'completed_tasks',
        'total_tasks',
        'completion_rate',
        2
    );
```

This generates a query that returns a `completion_rate` column by dividing `completed_tasks` by `total_tasks`, rounded to 2 decimal places.
Behind the scenes:

The query uses the SQL expression:
```sql
ROUND(completed_tasks / NULLIF(total_tasks, 0), 2) as completion_rate
```
The `NULLIF()` ensures you don’t divide by zero — if `total_tasks` is 0, the result will be NULL instead of throwing a database error.

Default Usage:

If no alias or precision is provided, it defaults to:
```php
->toRatio('wins', 'matches');
```
Which results in:
```php
[
  { "ratio": 0.75 },
  { "ratio": 1.00 },
  { "ratio": null }
]
```
Use this method when you want clean, readable ratios without handling raw SQL manually.

## Querying JSON Columns
Doppar's query builder includes powerful methods to search and extract values from JSON columns, using native MySQL JSON functions under the hood. This is useful when storing structured or dynamic data (e.g. settings, metadata, or preferences) in a single column.

Let’s assume a user record has the following field `attributes` as JSON like this:
```php
{
  "theme": "dark",
  "language": "en",
  "notifications": {
    "sms": false,
    "email": true
  }
}
```
### whereJson()
The `whereJson()` method allows you to query a specific value at a JSON path to check if a JSON path contains a specific value, including nested or array-based structures.

Here's how you might use it:
```php
User::whereJson('attributes', '$.theme', 'dark')
    ->whereJson('attributes', '$.notifications.email', true)
    ->get();
```

This method is ideal for exact value matching inside a JSON structure, including support for booleans.

When you need full control over the SQL expression — such as when working with JSON functions —` whereRaw()` lets you write raw SQL where clauses directly into your query.

Here’s how you can query a specific value inside a JSON column using JSON_EXTRACT:
```php
User::query()
    ->whereRaw("JSON_EXTRACT(attributes, '$.theme') = ?", ['dark'])
    ->get();
    
// ⚠️ Important Notes:
// - This uses raw SQL with JSON_EXTRACT, which is MySQL/MariaDB specific
```
For safer, chainable, and reusable alternatives, consider whereJson() if your use case fits.

### selectRaw or DB::sql() for JSON Extraction
If you want to extract a value from a JSON column in your select statement, use `DB::sql()` or `selectRaw()`:
```php
User::query()
    ->select(DB::sql("JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.theme')) as theme_preference"))
    ->get();
```
Or, with `selectRaw()`
```php
User::query()
    ->selectRaw("JSON_UNQUOTE(JSON_EXTRACT(attributes, '$.theme')) as theme_preference")
    ->get();
```
This will give you a new column `theme_preference` in your result set, showing each user's selected theme.

## Querying Date Columns
### whereDate()
The `whereDate()` method filters records by matching only the date part (`ignoring time`) of a column against a given value. It supports custom comparison operators.
```php
// Where date equals a specific date
User::query()
    ->whereDate('created_at', '2023-01-01')
    ->get();

// Where date is greater than a specific date
User::query()
    ->whereDate('created_at', '>', '2023-01-01')
    ->get();
```

### whereMonth()
The `whereMonth()` method filters records by matching only the month part of a date or datetime column against a given value. The month can be given as a number (1 for January, 12 for December) and supports custom comparison operators.
```php
// Where month is January (month 1)
Order::query()
    ->whereMonth('order_date', 1)
    ->get();

// Where month is greater than March
Order::query()
    ->whereMonth('order_date', '>', 3)
    ->get();
```

### whereYear()
The `whereYear()` method filters records by matching only the year part of a date or datetime column against a given value. Supports custom comparison operators.
```php
// Where year is 2023
Post::query()
    ->whereYear('published_at', 2023)
    ->get();

// Where year is greater than 2020
Post::query()
    ->whereYear('published_at', '>', 2020)
    ->get();
```

### whereDay()
The `whereDay()` method filters records by matching only the day of the month (1–31) from a date or datetime column. Supports custom comparison operators.
```php
// Where day is the 15th
Event::query()
    ->whereDay('event_date', 15)
    ->get();

// Where day is less than 10
Event::query()
    ->whereDay('event_date', '<', 10)
    ->get();
```
### whereTime()
The `whereTime()` method filters records by matching only the time part (`HH:MM:SS`) of a datetime or time column. Supports custom comparison operators.

```php
// Where time is after 14:00:00
Appointment::query()
    ->whereTime('start_time', '>', '14:00:00')
    ->get();

// Where time equals 09:30:00
Appointment::query()
    ->whereTime('start_time', '09:30:00')
    ->get();
```

### whereToday()
The `whereToday()` method filters records where the date part of a column matches today’s date.
```php
// Records created today
Order::query()
    ->whereToday('created_at')
    ->get();
```

### whereYesterday()
The `whereYesterday()` method filters records where the date part of a column matches yesterday’s date.
```php
// Records from last year
Statistic::query()
    ->whereYesterday('recorded_at')
    ->get();
```

### whereThisMonth()
The `whereThisMonth()` method filters records where the month part of a column matches the current month.
```php
// Records from this month
Sale::query()
    ->whereThisMonth('sale_date')
    ->get();
```

### whereLastMonth()
The `whereLastMonth()` method filters records where the month part of a column matches the previous month.
```php
// Records from last month
Invoice::query()
    ->whereLastMonth('invoice_date')
    ->get();
```

### whereThisYear()
The `whereThisYear()` method filters records where the year part of a column matches the current year.
```php
// Records from this year
Report::query()
    ->whereThisYear('report_date')
    ->get();
```

### whereLastYear()
The `whereLastYear()` method filters records where the year part of a column matches the previous year.
```php
// Records from last year
Statistic::query()
    ->whereLastYear('recorded_at')
    ->get();
```

### whereDateBetween()
The `whereDateBetween()` method filters records where a date or datetime column falls between two given values (inclusive). By default, only the date part is compared. You can enable time comparison with the `$includeTime` parameter.
```php
// Between two dates (date only comparison)
Result::query()
    ->whereDateBetween('test_date', '2023-01-01', '2023-01-31')
    ->get();
```

### whereDateTimeBetween()
The `whereDateTimeBetween()` method filters records where a datetime column falls between two given datetime values (inclusive of both boundaries). This method includes time comparison by default and is ideal for precise timestamp ranges.

Usage example
```php
// Basic datetime range (inclusive of both boundaries)
User::query()
    ->whereDateTimeBetween('created_at', '2025-01-01 00:00:00', '2025-10-31 13:59:59')
    ->get();

// Using DateTime objects
$start = new DateTime('2025-01-01 00:00:00');
$end = new DateTime('2025-10-31 13:59:59');
User::query()
    ->whereDateTimeBetween('created_at', $start, $end)
    ->get();

// Date strings with automatic time handling
// Start becomes '2025-01-01 00:00:00', End becomes '2025-01-31 23:59:59'
User::query()
    ->whereDateTimeBetween('created_at', '2025-01-01', '2025-01-31')
    ->get();

// Specific time window within a day
User::query()
    ->whereDateTimeBetween('created_at', '2025-01-15 09:00:00', '2025-01-15 17:00:00')
    ->get();
```

### iLike()
The `iLike()` method performs a case-insensitive pattern match on a given column using SQL’s LIKE operator, making it easy to filter results regardless of case differences.

This is particularly useful when you want to search for substrings in text columns without worrying about letter casing.

Example usage:
```php
User::query()
    ->iLike('name', '%john%')
    ->orderBy('name', 'desc')
    ->get();
```
This query returns all users whose name contains "john", "John", "JOHN", etc., ordered by name descending.

How it works:

The method generates a raw SQL condition like this:
```sql
LOWER(name) LIKE LOWER('%john%')
```
By lowering both the column and the pattern, it ensures the match is case-insensitive.

### transformBy()
The `transformBy()` method allows you to add a custom transformation or calculated expression to your query results. You pass a callback that receives a fresh query builder instance, and you return either a raw SQL expression string or a builder that builds the expression. The result is added as a new aliased column in your query.

Usage examples

Concatenate `first_name` and `last_name` as `full_name`
```php
User::query()
    ->transformBy(function ($query) {
        return "CONCAT(first_name, ' ', last_name)";
    }, 'full_name')
    ->get();
```
This appends a `full_name` column combining `first_name` and `last_name` with a space.

Select uppercase name from users table
```php
User::query()
    ->transformBy(function ($builder) {
        return "UPPER(name)";
    }, 'upper_name')
    ->get();
```
This adds an `upper_name` column that returns the uppercase version of the name field.

Calculate net compensation with salary and bonus
```php
Employee::query()
    ->transformBy(function ($builder) {
        return "(salary * 0.8) + (bonus * 1.2)";
    }, 'net_compensation')
    ->get();
```
This appends a `net_compensation` column calculating a weighted sum of salary and bonus for each employee.


Compute platform-weighted average engagement
```php
Post::query()
    ->where('category_id', 3)
    ->where('created_at', '>', now()->subYear())
    ->transformBy(function ($builder) {
        return "AVG(0.6 * facebook_share + 0.3 * linkedin_share + 0.1 * twitter_share)";
    }, 'platform_weighted_avg')
    ->first();
```
This query filters posts from category 3 created within the past year, and appends a `platform_weighted_avg` column representing the weighted average engagement score across multiple social platforms.

### wherePattern()
The `wherePattern()` method allows you to filter query results using SQL pattern matching techniques such as `LIKE`, `REGEXP`, or `SIMILAR` TO. It wraps a raw where clause for flexible, expressive string pattern filters.

Usage Examples

Match email format using REGEXP
```php
User::query()
    ->wherePattern('email', '^[a-z]+@demo\.com$', 'REGEXP')
    ->get();
```
This filters users whose email matches the pattern `name@demo.com`, where name consists of lowercase letters only.

Perform a case-insensitive match using LIKE
```php
User::query()
    ->wherePattern('name', '%Jewel%', 'LIKE')
    ->whereRaw('LOWER(name) LIKE LOWER(?)', ['%Jewel%'])
    ->get();
```
This finds users whose name contains `“Jewel”`, regardless of case, by combining `wherePattern()` and a manual `LOWER()` comparison for extra control.

### firstLastInWindow()
The `firstLastInWindow()` method allows you to compute window functions such as `FIRST_VALUE` and `LAST_VALUE` over a partitioned and ordered dataset. This is especially useful for analytics-style queries where you want to attach aggregated or reference values from within a group to each row.

Usage Example

Example: Get each employee's department highest salary
```php
Employee::query()
    ->select('name', 'salary', 'department')
    ->firstLastInWindow(
        'salary',              // Column to get value from
        'salary DESC',         // Order salaries descending
        'department',          // Partition by department
        true,                  // true i.e., max salary in DESC order
        'department_max_salary' // Alias
    )
    ->orderBy('salary', 'desc')
    ->get();
```

This will return all employees, along with the maximum salary within their department as a new column called `department_max_salary`. If first was set to `false`, it would return the `minimum (or last)` based on the ordering.

This is a powerful tool for windowed analytics and reporting, giving you rich insights from grouped data without breaking the row-level context.

### movingDifference()
Calculates the difference between a current value and the previous row's value in an ordered series using SQL LAG() function. This is useful for identifying changes over time or detecting trends.

Track daily energy consumption difference
```php
EnergyLog::query()
    ->select('date', 'kilowatt_hours')
    ->movingDifference('kilowatt_hours', 'date', 'daily_usage_change')
    ->orderBy('date')
    ->get();
```

Daily Sales Difference
```php
Sale::query()
    ->select('date', 'amount')
    ->movingDifference('amount', 'date', 'daily_change')
    ->orderBy('date')
    ->get();
```

Output:

| date       | amount | daily\_change |
| ---------- | ------ | ------------- |
| 2025-06-01 | 1000   | 1000          |
| 2025-06-02 | 1200   | 200           |
| 2025-06-03 | 800    | -400          |

### movingAverage()
Calculates a moving average over a specified window of rows using SQL's AVG() window function.
Commonly used to smooth out short-term fluctuations and highlight longer-term trends.

What It Does

For each row in the result set, movingAverage() computes the average of the specified column using the current row and a defined number of preceding rows, based on the order column. The number of rows is determined by the $windowSize parameter.

SQL example generated:
```sql
AVG(column) OVER (
    ORDER BY order_column
    ROWS BETWEEN N PRECEDING AND CURRENT ROW
) AS alias
```
Where N = windowSize - 1.

Use Cases
- Smoothing sales data over weeks or months
- Analyzing stock prices with trailing window averages
- Tracking temperature or energy use in environmental logs
- Monitoring trends in any sequential, numeric dataset

Calculate weekly moving average of daily sales
```php
Sale::query()
    ->select('date', 'amount')
    ->movingAverage('amount', 7, 'date', 'weekly_avg')
    ->orderBy('date')
    ->get();
```
Adds a `weekly_avg` column showing a rolling average of the last 7 days' sales (including the current day).

30-Day Moving Average of Stock Prices
```php
StockPrice::query()
    ->select('date', 'closing_price')
    ->movingAverage('closing_price', 30, 'date', 'monthly_avg')
    ->orderBy('date')
    ->get();
```
Adds `monthly_avg` showing the average closing price over the last 30 days per row.

## Query Filter Using match()
The match() method in Doppar offers a clean and expressive way to apply dynamic, reusable filters to your queries. It supports both simple key-value pairs and more complex conditions using callbacks, making it ideal for building powerful, customizable queries without cluttering your controller or model logic.

Very basic filtering
```php
Post::match([
    'id' => 1,
    'user_id' => 1
])->get();
```
Applies standard where clauses for each key-value pair. Allows complex conditions inside a closure for more flexible logic.

Filter using requested data
```php
Post::match($request->only(['title', 'user_id']))->paginate();
```
Automatically filters based on available request fields—great for search forms or filters.

Combining simple and complex filters
```php
Post::match([
    'user_id' => [1, 2, 3],
    function ($query) {
        $query->where('views', '>', 100)
              ->where('status', 1);
    }
])
->orderBy('created_at', 'desc')
->get();
```

### match() with a Callback
In some cases, you may want to pass a callback directly into the `match()` method to filter results based on a related condition. This is especially useful for applying inline, scoped constraints without defining a separate scope method.

See a very basic example
```php
Post::match(function ($query) {
    $query->where('views', '>', 100)
          ->whereBetween('created_at', ['2023-01-01', '2023-12-31']);
})->get();
```

### Relationship Existence with a Callback

The following query fetches users who have at least one post, orders them by name in descending order, and limits the results to the first 10 records:
```php
User::match(fn($query) => $query->present("posts"))
    ->orderBy("name", "desc")
    ->limit(10)
    ->get();
```

The `match()` method enhances query readability and reduces repetitive condition-building, especially in service layers or controllers.

## Conditional Query Execution 
In Doppar, the `if()` method provides a powerful and elegant way to conditionally add query constraints based on a given condition. This allows you to build more flexible and dynamic queries, improving code readability and reducing unnecessary complexity.

Doppar ORM's `if()` method allows you to conditionally add query constraints based on a given condition. If the condition evaluates to true, the corresponding query modification is applied; otherwise, it is skipped.

Basic usage of `if()`
```php
Post::query()
    ->if($request->has('date'), function($query) use ($request) {
        $query->where('date', $request->has('date'));
    })
    ->get();
```
In this example, the `if()` method checks if the `date` is present in the request. If it is, the query will filter the posts based on the provided `date`. If not, the `where()` condition is skipped.

### Conditional Query Modifications with a Default Case
You can also provide a default case to be applied when the condition is false or not provided:
```php
Post::query()
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
Post::query()
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
Post::query()
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

## Query Binding
Doppar allows you to define reusable query in your model. You simply define methods on your model, and Doppar will automatically detect and attach them during query execution. You just need to use `__` syntax before your query as a prefix.

This makes your code cleaner, shorter, and more expressive, improving the developer experienc.
##### Example
```php
User::admin()->get();
```
This automatically applies both `admin()` query filters. Your model should have the following method like this way.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;
use Phaseolies\Database\Eloquent\Builder;

class User extends Model
{
     /**
     * Admin users
     *
     * @param Builder $query
     * @return Builder
     */
    public function __admin(Builder $query): Builder
    {
        return $query->where('role', 'admin');
    }
}
```
Doppar inspects query builder method calls. If the method name matches a defined method on the model, Doppar executes it as part of the query. Method must return `Phaseolies\Database\Eloquent\Builder` instance. You can chain as many dynamic binding as you want.

### Query Binding with Parameters
Doppar lets you define reusable queries and pass parameters to them, making your code more flexible. By simply adding arguments to your methods, Doppar automatically integrates them into the query logic, keeping your code clean and expressive.
```php
User::active('Nure')->get();
```
In this example, the `active()` method accepts a parameter, and Doppar automatically attaches it to the query logic.

Model Definition with Parameters
```php
public function __active($query, $name)
{
    return $query->where('status', 1)->whereName($name);
}
```
The first parameter in these methods is always the query builder object.

## Insertion and Update
### Inserts

Of course, when using Doppar, working with data isn't limited to just retrieving records — inserting new entries is just as important. Fortunately, Doppar makes this process straightforward. To add a new record to the database, you simply create a new instance of the model, assign values to its attributes, and then call the save() method:
```php
use App\Models\User;

$user = new User();
$user->name = 'Mahedi Hasan';
$user->email = 'mahedi@doppar.com';
$user->save();
```

This inserts a new row into the users table with the specified name and email. The save() method takes care of generating and executing the appropriate SQL insert query behind the scenes, making data insertion clean and intuitive.

Alternatively, you may use the create method to "save" a new model using a single PHP statement. The inserted model instance will be returned to you by the create method:
```php
use App\Models\User;

User::create([
    'name' => 'Doppar'
]);
```

### `firstOrCreate()`
The `firstOrCreate()` method is used to retrieve the first record that matches the given attributes. If no matching record exists, it will create a new record using the provided attributes and additional values.

Unlike `updateOrCreate()`, this method does not update an existing record — it will simply return it unchanged if found.
```php
User::firstOrCreate(
    ['email' => 'howdy@doppar.com'], // attributes to match
    ['name' => 'Doppar']             // values to use when creating
);
```

This is particularly useful when you need to ensure a record exists without accidentally updating existing data.

### Forking Models
In addition to inserting and retrieving records, sometimes you’ll want to create a new record by copying an existing one. The `fork()` method makes this easy by producing a new instance of the model without the `primary key`. You can then modify its attributes and persist it to the database as a brand-new record.

Basic replication
```php
$newPost = $post->fork();
$newPost->title = 'Copy of ' . $post->title;
$newPost->save();
```
This creates a copy of the existing post, removes the `primary key`, and allows you to make changes before saving as a new record.

Replicate with specific attributes excluded
```php
$newPost = $post->fork(['views_count', 'published_at']);
```
Here, the new instance will be a copy of the original, but the `views_count` and `published_at` attributes will be excluded.

The `fork()` method is especially useful when you need to quickly duplicate an existing model without re-entering all of the data.

### Updates
The `save()` method in Doppar isn't just for inserting new records—it can also be used to update existing ones. To update a record, first retrieve the model instance, modify the attributes you want to change, and then call `save()`. Doppar will handle generating the appropriate UPDATE statement. Additionally, the updated_at timestamp will be refreshed automatically, so you don't need to set it manually:
```php
use App\Models\User;

$user = User::find(1);
$user->name = 'Nure';
$user->save();
```

## Dirty Attributes Before Saving
Before calling `save()`, you can inspect which attributes have changed (are "dirty") using built-in methods. Doppar provides `getDirtyAttributes` method that returns an array of the attributes that have been modified:
```php
$user = User::find($id);
$user->name = $request->name;
$user->email = $request->email;

$dirty = $user->getDirtyAttributes();
dd($dirty); // e.g., ['name' => 'Nure']

// Use this method to check if a specific attribute is dirty:
if ($user->isDirtyAttr('name')) {
    // The name has changed
}

// Optional: Check if anything changed before saving
if (!empty($user->getDirtyAttributes())) {
    // Only changed fields will be updated
    // Though Doppar by default only update dirty records
    $user->save();
}
```
This ensures unnecessary database updates are avoided and improves performance. You can also update like this way, this way also usage dirty attrubutes to update data.
```php
User::find($id)
    ->update([
        'name' => $request->name
    ]);
```

### Conditional Updates
Updates can also be performed against models that match a given query.
```php

User::query()
    ->where('country', 'bd')
    ->update([
        'code' =>  '+880'
    ]);
```

### Updating witn `tap()`
The `tap()` helper function allows you to perform actions on a value (typically a model instance) while always returning that value. This is especially useful when you want to execute side effects (like updates, logging, or mutations) without breaking method chaining or needing temporary variables.
```php
$tag = tap(
    Tag::find(1),
    function ($tag) {
        $tag->update([
            'name' => 'Aliba'
        ]);
    }
);

return $tag; // Returns the updated model instance
```

This pattern ensures that you can modify a value and still work with the original object without needing an extra line for returning it.

### `updateOrCreate()`
Occasionally, you may need to update an existing model. The `updateOrCreate()` method, the `updateOrCreate()` method persists the model, so there's no need to manually call the save method.

The `updateOrCreate()` method is used to either update an existing record or create a new one if no matching record is found. It simplifies handling scenarios where you need to ensure a record exists with specific attributes while updating other fields.
```php
User::updateOrCreate(
    ['email' => 'howdy@doppar.com'], // attributes to match
    ['name' => 'Doppar'] // values to update/create
);
```

### `updateOrIgnore()`
Sometimes you may want to update an existing model only if it already exists, and simply do nothing if no matching record is found. The `updateOrIgnore()` method provides this behavior.

Unlike `updateOrCreate()`, the `updateOrIgnore()` method does not create a new record when no match is found — it will simply return `null`. If a record is found, it will update the given fields and persist the model to the database.
```php
User::updateOrIgnore(
    ['email' => 'howdy@doppar.com'], // attributes to match
    ['name' => 'Doppar'] // values to update/ignore
);
```

### Updating Data with fill()
In Doppar, you can streamline the process of updating multiple attributes by using the `fill()` method. This method allows you to pass an array of key-value pairs representing the fields you want to update. After filling the model with new data, you simply call save() to persist the changes:
```php
$user = User::find(1);
$user->fill([
    'name' => 'John',
    'email' => 'updated@doppar.com',
]);
$user->save();
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

$affectedRows = User::query()->upsert(
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

$affectedRows = User::query()->upsert(
    $records,
    ["email", "phone"],  // Combination must be unique
    ["name"],            // Only update the name
    false
);
```
Result:
- Matches on both email and phone.
- If both match, updates name; otherwise inserts.

## Bulk Insert Using saveMany()
When you need to insert multiple records at once, Doppar provides the saveMany() method, which simplifies batch inserts. Instead of saving each record individually, you can pass an array of data, and Doppar will insert them all in a single operation.

Here’s an example of how you can use saveMany() to insert multiple users:
```php
$password = bcrypt('abc');

User::saveMany([
    ['name' => 'John', 'email' => 'john@a.com', 'password' => $password],
    ['name' => 'Jane', 'email' => 'jane@b.com', 'password' => $password],
    ['name' => 'Bob', 'email' => 'bob@c.com', 'password' => $password]
]);
```
This inserts all three users into the users table in one batch, making the process more efficient than inserting them one by one.

### Chunk Size for Large Datasets
If you are working with a large dataset, inserting all records at once might lead to performance issues or memory limitations. In such cases, you can specify a chunk size as the second parameter to saveMany() to batch the inserts into smaller chunks. For example:
```php
$password = bcrypt('abc');

User::saveMany([
    ['name' => 'John', 'email' => 'john@abc.com', 'password' => $password],
    ['name' => 'Jane', 'email' => 'jane@abc.com', 'password' => $password],
    ['name' => 'Bob', 'email' => 'bob@abc.com', 'password' => $password]
], 1000);
```
This will insert the records in chunks of 1000, helping prevent memory overflow and improving performance when dealing with a large volume of data.

Using `saveMany()` is a great way to handle bulk inserts in Doppar, ensuring your application remains efficient even with large datasets.

## Mass Assignment

You may use the create method to "save" a new model using a single PHP statement. The inserted model instance will be returned to you by the method:
```php
use App\Models\User;

$user = User::create([
    'name' => 'Nure',
]);
```

However, before using the **create** method, you will need to specify a `$creatable` property on your model class. These properties are required because all Eloquent models are protected against mass assignment vulnerabilities by default.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $creatable = ['name'];
}
```

So, to get started, you should define which model attributes you want to make mass assignable. You may do this using the **$creatable** property on the model. For example, let's make the name attribute of our User model mass assignable:

### Deleting Models
To delete a model, you may call the delete method on the model instance:
```php
User::find(1)->delete();
```

#### Deleting an Existing Model by its Primary Key

In Doppar, if you already know the primary key(s) of the records you want to delete, there's no need to retrieve the models first. Instead, you can call the purge method directly. This method efficiently deletes records by their primary key and supports various input formats, including single IDs, multiple IDs, and arrays
```php
User::purge(1);
User::purge(1, 2, 3);
User::purge([1, 2, 3]);
```

### Deleting Models Using Queries
Of course, Doppar also allows you to perform bulk deletions by building a query that targets specific records. For instance, you can delete all users that are marked as inactive in a single operation. Just like with mass updates, these mass deletions are executed directly in the database.
```php
User::query()
    ->where('active', false)
    ->delete();
```

## Aggregation
The Doppar ORM provides a rich set of aggregation functions that allow you to perform statistical and summary operations directly through your model queries. These methods help extract meaningful insights from your data without writing raw SQL, making your code clean, expressive, and efficient.

### Total Sum of a Column
To calculate the sum of all values in a specific column (e.g., total views across all posts):
```php
Post::query()->sum('views');
```

### Average Value of a Column
To compute the average value of a column (e.g., average number of views):
```php
Post::query()->avg('views');
```

### Maximum Value in a Column
To get the highest value in a column (e.g., the most expensive product):
```php
Product::query()->max('price');
```

### Minimum Value in a Column
To get the lowest value in a column (e.g., cheapest product):
```php
Product::query()->min('price');
```

### Standard Deviation Calculation
To compute the standard deviation of values in a column (useful for measuring variability):
```php
Product::query()->stdDev('price');
```

### Variance Calculation
To calculate the variance of values (a squared measure of data spread):
```php
Product::query()->variance('price');
```

### Multiple Aggregations in One Query
To retrieve count, average, min, and max in a single grouped query:
```php
Product::query()
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
Post::query()->distinct('user_id');
```

### Calculating Conditional Sum
To sum values under a specific condition (e.g., sales where title is "maiores"):
```php
Product::query()
    ->where('title', 'maiores')
    ->sum('price');
```

### Total Sales by Category
To aggregate sales per category by multiplying price and quantity:
```php
Product::query()
    ->select(['category_id', 'SUM(price * quantity) as total_sales'])
    ->groupBy('category_id')
    ->get();
```

## Column Modification: increment() & decrement()
#### Incrementing a Column
To increase the value of a numeric column (e.g., views):
```php
$post = Post::find(1);
$post->increment('views'); // Increments by 1 by default
$post->increment('views', 10); // Increments by 10
```

You can also update additional fields during the increment:
```php
$post->increment('views', 1, [
    'updated_at' => date('Y-m-d H:i:s'),
    'modified_by' => Auth::id()
]);
```

### Decrementing a Column
To decrease the value of a numeric column:
```php
$post = Post::find(1);
$post->decrement('views'); // Decrements by 1 by default
$post->decrement('views', 10); // Decrements by 10
```

You can also attach extra updates when decrementing:
```php
$post->decrement('views', 1, [
    'updated_at' => date('Y-m-d H:i:s'),
    'modified_by' => Auth::id()
]);
```

Doppar ORM empowers you to run comprehensive statistical queries, manage data updates, and build analytics-driven features with minimal effort. Whether you're building dashboards, reports, or tracking metrics, these aggregation tools are essential for maintaining efficient and elegant code.

## Transform Eloquent Collection
The Doppar ORM offers expressive methods like `map()`, `filter()`, and `each()` to transform, filter, and iterate over Eloquent collections with ease. These methods provide fine control over the shape and content of your data after fetching it from the database.

### map() – Transforming Collection Items
Use the `map()` method to transform each item in a collection. This is especially useful when you want to reformat or limit the fields returned in your response.

Example: Return Only the name of Each User
```php
User::all()
    ->map(function ($item) {
        return [
            'name' => $item->name
        ];
    });
```

> Use case: Customize API responses or prepare data for front-end consumption by trimming down unnecessary fields.

### `map()` with Property Shortcut
When you only need to access a property or call a method without arguments on each item, you can use the property shortcut syntax. This makes your code more concise:
```php
$users = User::all();

$names = $users->map->name;

// Equivalent to $users->map(fn($user) => $user->name)
```

The property shortcut works great with relationships as well:
```php
$post = Post::find(1);

$tagNames = $post->tags->map->name;

// Collection: ['doppar', 'php', 'eloquent']
```

Here, `$post->tags` returns a collection of related Tag models, and `map->name` transforms it into a collection of just the name values

### filter() – Conditional Filtering
After transforming a collection, you can use filter() to return only items that match a given condition.

Example: return posts where `status = 1`, with only title and status
```php
Post::all()
    ->map(function ($item) {
        return [
            'title' => $item->title,
            'status' => $item->status
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
User::all()
    ->each(function ($user) {
        $user->sendEmail();
    });
```

### `each()` with Property Shortcut
The `each()` method iterates over every item you already know. But in doppar, for cases where you want to call the same method on every item, you can use the method shortcut:
```php
$users = User::all();

$users->each->sendEmail();
```
This is equivalent to the closure example above, but much more concise.

These methods make Doppar’s Eloquent collection manipulation concise, readable, and highly adaptable for API responses, data formatting, and conditional logic. See more from [collections](collections.html).

## Pagination
Pagination is an essential feature for working with large datasets, enabling you to fetch and display data in manageable chunks. This improves both performance and user experience, especially in applications that deal with lists like users, posts, products, or logs.

Doppar makes pagination seamless through the built-in `paginate()` method, which handles all the behind-the-scenes logic for splitting your results into pages.

### Basic Usage
To paginate a dataset, simply call the paginate() method like as follows:
```php
User::paginate(1);
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
@foreach ($data['data'] as $user)
    <tr>
        <td>{{ $user->id }}</td>
        <td>{{ $user->name }}</td>
        <td>{{ $user->username }}</td>
        <td>{{ $user->email }}</td>
    </tr>
@endforeach

<!-- "Previous" and "Next" buttons, along with page numbers. -->
{!! paginator($data)->links() !!}

<!-- "Previous" and "Next" buttons, along with page jump options. -->
{!! paginator($data)->linkWithJumps() !!} // 
```

## Customize Default Pagination
Doppar provides a Bootstrap 5 pagination view by default. However, you can also customize this view to suit your needs. To customize the pagination view, Doppar offers the publish:pagination pool command. Running this command will create two files, `jump.blade.php` and `number.blade.php`, inside the `resources/views/vendor/pagination` folder. These files allow you to tailor the pagination design to match your application's style.

```bash
php pool publish:pagination
```

Once you modify the `jump.blade.php` and `number.blade.php` files, the changes will immediately reflect in your pagination view. This allows you to fully customize the appearance and behavior of the pagination links to align with your application's design and requirements. Feel free to update these files as needed to create a seamless and visually consistent user experience.

## Retrieving a Paginated Subset of Records
To fetch a specific subset of records from the database—such as a “page” of users—you can use the offset and limit methods on an Eloquent query.
```php
User::query()
    ->offset(4)  // Skip the first 4 records
    ->limit(10)  // Retrieve the next 10 records
    ->get();
```
This is useful for simple pagination or when you want to retrieve a specific slice of your dataset.

<a name="database-transactions"></a>

## Database Transactions
A database transaction is a sequence of database operations that are executed as a single unit. Transactions ensure data integrity by following the ACID properties (Atomicity, Consistency, Isolation, Durability). If any operation within the transaction fails, the entire transaction is rolled back, preventing partial updates that could leave the database in an inconsistent state.

Doppar provides built-in support for handling database transactions using the `DB::transaction()` method, `DB::beginTransaction()`, `DB::commit()`, and `DB::rollBack()`.

### Using `DB::transaction()` for Simplicity
The DB::transaction() method automatically handles committing the transaction if no exception occurs and rolls it back if an exception is thrown.
```php
use Phaseolies\Support\Facades\DB

DB::transaction(function () {
    $user = User::create($request->all());
    $post = Post::create([
        'user_id' => $user->id, 
        'title' => 'War is started'
    ]);
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
}, 3); // Will attempt up to 3 times before throwing an exception
```

This approach helps mitigate issues caused by deadlocks by retrying the transaction a set number of times before ultimately failing.

Using transactions properly ensures database consistency and prevents data corruption due to incomplete operations. Doppar provides flexible methods for handling transactions, allowing both automatic and manual control based on the use case.

<a name="eloquent-join"></a>

## Eloquent Join
In Doppar, Eloquent manual joins allow you to retrieve data from multiple tables based on a related column. The join method in Eloquent's Query Builder provides an easy way to combine records from different tables. This document explains how to perform various types of joins manually using Eloquent's Eloquent ORM and Query Builder.

#### Basic Join Example
A simple join operation can be performed using the join method to combine records from two tables based on a common key. Below is an example of joining users and posts tables:
```php
$users = User::query()
    ->select('posts.*', 'users.name as user_name')
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->get();
```

This will return a dataset containing user data which has at least one post.

### Specifying Join Type
By default, Doppar performs an `INNER JOIN`. You can specify the type of join you want by passing the join type as an argument:
```php
User::query()
    ->join('posts', 'users.id', '=', 'posts.user_id', 'left')
    ->get();
```

Here, a `LEFT JOIN` is used to include all users, even if they do not have associated posts.

### Applying Conditions in Joins
You can apply additional conditions to filter the joined data. The example below joins the posts table and fetches only the users who have published posts:
```php
User::query()
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->where('posts.published', true)
    ->orderBy('users.name', 'ASC')
    ->get();
```

### Performing Multiple Joins
You can join multiple tables in a single query. The example below joins the users, posts, and comments tables:
```php
User::query()
    ->join('posts', 'users.id', '=', 'posts.user_id')
    ->join('comments', 'posts.id', '=', 'comments.post_id')
    ->get();
```
This will return data containing users, their posts, and associated comments.

### Selecting Specific Columns
To optimize queries and improve performance, you can select specific columns instead of retrieving all fields:

```php
User::query()
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
You can pass where condition in your custom view like
```php
// View with single WHERE condition
$nyUsers = DB::view(
    'vw_user_locations', 
    ['state' => 'New York']
);

// Equivalent to: SELECT * FROM vw_user_locations WHERE state = 'New York'

// View with multiple WHERE conditions
$premiumNyUsers = DB::view(
    'vw_user_locations',
    [
        'state' => 'New York',
        'account_type' => 'premium'
    ]
);

// Equivalent to: 
// SELECT * FROM vw_user_locations 
// WHERE state = 'New York' AND account_type = 'premium'
```

### View with Parameter Binding
You can also pass params with where condition as follows
```php
// 4. Using parameter binding for security
$recentOrders = DB::view(
    'vw_recent_orders',
    ['status' => 'completed'],
    [':min_amount' => 100] // Additional parameters
);

// Equivalent to:
// SELECT * FROM vw_recent_orders 
// WHERE status = 'completed' AND amount > :min_amount
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

## Handling Large Dataset
Working with large datasets can be challenging in terms of performance and memory efficiency. The Doppar framework provides you process and stream large amounts of data without exhausting system resources. These tools are especially useful for batch operations, background jobs, reporting systems, and data migrations.

This module introduces a variety of data handling strategies tailored to different use cases, including:

- Chunk-based processing to break large datasets into manageable segments.
- Cursor-based streaming for low-memory iteration over millions of records.
- Generators to provide lazy, memory-efficient traversal of datasets.
- Fiber-based concurrency for advanced parallel data processing and streaming.

All these approaches are designed with memory safety in mind, incorporating garbage collection and careful resource cleanup to avoid memory leaks or excessive overhead during execution.

Whether you're building scalable ETL pipelines or simply need to process records in a loop without crashing your application, Doppar’s dataset handling capabilities provide a robust and flexible foundation.

### chunk
When dealing with large datasets, loading all records into memory at once can cause significant performance degradation or even out-of-memory errors. The `chunk()` method addresses this by breaking the dataset into smaller chunks and processing each chunk independently. This allows you to iterate through the entire dataset efficiently, without overwhelming system resources.

The `chunk()` method accepts three parameters:
- `$chunkSize` – the number of records to retrieve per batch.
- `$processor` – a callback function that receives each chunk for processing.
- `$total (optional)` – the total number of records expected, useful for tracking progress.

Each chunk is fetched using pagination via `LIMIT` and `OFFSET`. After processing a chunk, the memory is freed, and garbage collection is invoked to minimize leaks.

Below is an example where we process active users in chunks of 100 records:
```php
use App\Models\User;
use Phaseolies\Support\Collection;

User::query()
    ->where('status', true)
    ->chunk(100, function (Collection $users) {
        foreach ($users as $user) {
            // Perform operations on each user
        }
    });
```
This approach keeps memory usage low and is ideal for operations such as batch updates, exporting data, or applying transformations to large tables.

### Tracking Progress with chunk()
In addition to memory-efficient processing, the `chunk()` method in Doppar also supports progress tracking. This is especially useful when you want to provide feedback during long-running operations, such as logging progress, displaying status bars, or estimating time to completion.

By passing a third parameter, `$total`, to the `chunk()` method, your processor callback will receive not only the current chunk of data but also:
```php
User::query()
    ->where('status', true)
    ->chunk(
        chunkSize: 100,
        processor: function (Collection $users, $processed, $total) {
            foreach ($users as $user) {
                // Perform operations on each user
                echo "Processed $processed of $total\n";
            }
        },
        total: 500
    );
```
This pattern is ideal when you're running background tasks or CLI scripts and want real-time feedback or logging.

### fchunk() – Concurrent Chunk
When speed is critical and your system can handle parallel operations, Doppar provides the `fchunk()` method—a fiber-based parallel chunk processor. This allows multiple chunks to be processed concurrently, taking advantage of lightweight PHP Fibers to speed up the total processing time.

Unlike traditional `chunk()` processing, which is sequential, `fchunk()` runs multiple chunks in parallel, each in its own fiber. This can significantly reduce the time needed to process large datasets, especially when the per-record logic is I/O-bound or computationally light.

Parameters
- **chunkSize:** Number of records per chunk.
- **processor:** Callback to process each chunk of records.
- **concurrency:** Number of fibers (chunks) to process in parallel.

> ⚠️ Note: Fiber-based concurrency is cooperative, not true multithreading. It still runs on a single PHP thread, but enables interleaving tasks efficiently.

Example with concurrently processing active users
```php
User::query()
    ->where('status', true)
    ->fchunk(
        chunkSize: 100,
        processor: function (Collection $users) {
            foreach ($users as $user) {
                // Handle user logic
            }
        },
        concurrency: 4
    );
```
In this example:
- Users are fetched in chunks of 100.
- Up to 4 chunks are processed in parallel
- Each chunk is passed to the callback where individual records are handled.

This technique is ideal for high-throughput batch jobs where database or API operations can be overlapped efficiently. It's especially helpful in CLI scripts or daemons where time-to-completion matters.

### cursor() – Stream Records
The `cursor()` method is designed for ultra-efficient, low-memory iteration over large datasets. Instead of loading all records or chunks into memory, `cursor()` uses a forward-only PDO cursor to stream records one at a time directly from the database.

This method is ideal when dealing with millions of records, performing exports, or executing operations where memory usage must stay constant.

Parameters
- **processor:** A callback function that is invoked for each record.
- **total (optional):** Total number of records expected, useful for progress tracking.

### How It Works
Internally, `cursor()` prepares the query, binds parameters, and fetches each row as an associative array. Each row is immediately converted into a model and passed to the callback function. After each record is processed, the memory is cleaned up and garbage collection is triggered.
```php
User::query()
    ->where('status', true)
    ->cursor(function ($user) {
        // Handle user logic
    });
```
The operation is memory-stable regardless of dataset size.

- *Best for:* Iterating huge datasets with low memory usage.

> ⚠️ Avoid using cursor() if your processing logic requires sorting, seeking, or buffering large sets of rows.

You can also handle progress in cursor like this way
```php
User::query()
    ->where('status', true)
    ->cursor(function ($user, $processed, $total) {
        //  Handle user logic
        echo "Processed $processed of $total\n";
    }, 500);
```

### fcursor() – Hybrid Cursor
The `fcursor()` method blends the memory efficiency of a traditional cursor with the performance benefits of Fibers. This hybrid approach lets you stream records from the database in batches using a cursor, but buffer and process them efficiently using Fiber-based suspension.

Unlike `cursor()` which processes one row at a time, `fcursor()` buffers a group of records (using a configurable bufferSize) and yields them together to the processor. This reduces the overhead of per-row context switching and provides a balance between memory usage and performance.

Parameters
- **processor:** A callback function invoked for each record in the buffered batch.
- **bufferSize:** The number of records to accumulate before passing them to the processor (default is 1000).

#### How It Works
- A PDO cursor fetches records from the database.
- Records are buffered into an array.
- Once the buffer reaches the defined bufferSize, the Fiber suspends and yields control to the processor.
- Processing resumes after the callback is done, and continues until all records are exhausted.

Example with user list streaming
```php
User::query()
    ->where('status', true)
    ->fcursor(function ($user) {
        //
    });
```
This method is particularly useful when:

- You want better performance than single-row cursors.
- Your logic can handle medium-sized memory buffers.
- You’re building CLI tools or background workers that must process large volumes reliably.

> ⚡ Efficient like cursor(), but faster for many real-world workloads due to fiber-based batching.

### stream() – Lazy Generator
The `stream()` method offers a clean, generator-based approach to iterating over large datasets in a memory-efficient way. It loads records in small chunks `(like chunk())`, but yields one record at a time, allowing you to use it with foreach just like a native PHP array — without holding everything in memory.

Unlike `cursor()`, `stream()` handles records in small batches (e.g., 100 at a time) and is suitable when you prefer chunked I/O with on-the-fly transformation or mapping.

Parameters
- **chunkSize:** The number of records to fetch per internal chunk.
- **transform (optional):** A callback function to transform each model before yielding.

Example with Lazy Streaming
```php
$users = User::query()
    ->where('status', true)
    ->stream(100); // Fetch 100 at a time lazily

foreach ($users as $user) {
    //
}
```

In this example Each user is yielded individually from the generator. Memory usage stays consistent regardless of dataset size.

## Transforming Models During Stream
You can also transform records before they are yielded using the optional `transform` parameter.
```php
foreach (
    User::query()
        ->where('status', true)
        ->stream(1000, fn($user) => strtoupper($user->name)) // Convert name to uppercase
    as $userName
) {
    dump($userName); // Already transformed string
}
```
In this example
- Here, each user is mapped to an uppercase string before being yielded.
- The result is a generator of transformed values (not models).
- Useful for lightweight export pipelines, CSV dumps, or messaging queues.

> 💡 Tip: Use stream() when you want a foreach-friendly syntax with low memory use, especially in long-running CLI tasks or APIs.

### fstream() – Streaming with Backpressure Control
The `fstream()` method is an advanced Fiber-powered generator for streaming large datasets efficiently. It combines the lazy iteration style of `stream()` with the performance and concurrency control of Fibers.

Unlike `stream()`, `fstream()` is designed for high-throughput environments where you may want to buffer records in controlled amounts and suspend/resume execution more precisely.

Parameters
- **chunkSize:** Number of records to fetch per database request.
- **transform (optional):** Callback to transform each model before yielding.
- **bufferSize:** Maximum number of items to buffer before yielding (defaults to 1000).

Example: Streaming with Transformation
```php
foreach (
    User::query()
        ->where('status', true)
        ->fstream(1000, fn($user) => strtoupper($user->name))
    as $userName
) {
    dump($userName); // Already transformed
}
```

In this example
- Records are fetched in chunks of 1000.
- Transformed immediately into uppercase strings.
- Buffer is filled (up to default 1000) before being flushed into the foreach loop.
- Memory remains stable, and Fiber context switching keeps performance high.

####  How It Works
- Internally, Fibers pull in small batches of records.
- Each record is passed through the transform (if given).
- Fibers suspend after each yield, keeping processing responsive.
- A buffer size (default 1000) prevents overloads while maximizing I/O efficiency.

### batch() – Grouped Batch Processing 
The `batch()` method is tailored for scenarios where you want to process large numbers of records in grouped batches instead of one at a time. It provides an efficient way to collect data in memory-limited chunks and flush it to a processor in meaningful units (e.g., every 1000 records).

This is particularly useful when writing to external systems, doing bulk inserts, or when you want to limit the number of operations per database or API call.

Parameters
- **chunkSize:** Number of records to fetch from the database per query.
- **batchProcessor:** A callback that receives each batch as a Collection.
- **batchSize:** Number of records per batch to accumulate before processing (default: 1000).

Example: Process Users in Batches of 1000
```php
User::query()
    ->where('status', true)
    ->batch(
        chunkSize: 500,
        batchProcessor: function ($batch) {
            foreach ($batch as $user) {
                //
            }

            echo "Processed batch of {$batch->count()} users\n";
        },
        batchSize: 1000
    );
```

In this example
- This example fetches records in chunks of 500 from the database.
- Records are accumulated into a batch of 1000 before calling the processor.
- The batchProcessor is only triggered once the batch size is met (or at the end of the stream).

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
In real-world applications, especially large-scale or modular systems, it's common to store data across multiple databases. For example, reporting data might live in a separate database from transactional data, or different clients (in a multi-tenant architecture) may each have their own database.

Doppar makes it easy to query from different database connections using Eloquent models. Whether you want a model to always use a specific connection, or you want to dynamically switch connections at runtime, Doppar provides clean and flexible options.

Assume you're using a model like `Report`. Below are three ways to run queries on different database connections:

```php
// Default way
Report::query()
    ->orderBy('id', 'desc')
    ->get();

// With specific connection
Report::query('mysql_second')
    ->orderBy('id', 'desc')
    ->get();

// Using connection() method
Report::connection('mysql_second')
    ->orderBy('id', 'desc')
    ->get();
```
This tells the model to run the query on the `mysql_second` connection instead of the default one. Perfect for pulling data from a secondary database on the fly.

## Database Connection in Models
In Doppar, you can assign a specific database connection to a model by setting the `$connection` property within the model class. Once this is defined, all operations performed on the model will automatically use the specified connection, so you don’t need to explicitly set it each time.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

class Report extends Model
{
    protected $connection = 'mysql_second';
}
```
With the connection explicitly set in the model, all database operations on the `Report` model will use the `mysql_second` connection by default.

```php
// Uses the 'mysql_second' connection

Report::query()->orderBy('id', 'desc')->get();

Report::query()
    ->where('id', 1)
    ->update([
        'status' => 'reviewed',
    ]);

// All operations below continue using 'mysql_second'
$report = Report::find(1);
$report->title = 'Updated Report';
$report->save();

Report::create([
    'title' => 'New Report',
    'status' => 'pending',
]);
```
There is no need to manually set the connection using `connection()` or passing connection using `query('mysql_second')` method. Doppar will automatically route all queries to the correct database connection based on the model’s `$connection` property.

## `DB` Facade with Specific Connection
In Doppar, if you're working with multiple databases, you don't always need to use models. Sometimes you might want to perform database operations directly using the query builder, which is both powerful and flexible.

To do this on a specific database connection, Doppar lets you chain `connection('mysql_second')` before calling any query builder method. This allows you to use the full capabilities of `DB` facades query on a different database.

Below is a basic example showing how you can use various methods on a specific connection using Doppar’s DB facade:
```php
use Phaseolies\Support\Facades\DB;

// Run a simple query on the 'mysql_second' connection
DB::connection('mysql_second')->query('SELECT * FROM reports');

// Get all table names from the 'mysql_second' database
DB::connection('mysql_second')->getTables();

// Check if a specific table exists in the secondary connection
DB::connection('mysql_second')->tableExists('user');

// Access the raw PDO connection for lower-level operations
DB::connection('mysql_second')->getConnection();
```

## Running Migrations on Specific Connection
Doppar supports multi-database migrations out-of-the-box through its custom Schema and Blueprint classes. These allow you to define and execute schema changes (e.g., creating tables) on any configured database connection—not just the default one.

This is especially useful for:
- Modular applications where different modules use isolated databases
- Multi-tenant systems
- Reporting or archive databases

#### Creating a Table on `mysql_second`
Doppar’s `Schema::connection()` method returns a new instance of Schema configured to use the given database connection. Internally, this class uses the `DB::connection()` method to route all schema operations (create, drop, check, modify) to the appropriate database.
```php
<?php

use Phaseolies\Support\Facades\Schema;
use Phaseolies\Database\Migration\Blueprint;
use Phaseolies\Database\Migration\Migration;

return new class extends Migration
{
    /**
     * Run the migrations.
     */
    public function up(): void
    {
        Schema::connection('mysql_second')
            ->create('reports', function (Blueprint $table) {
                $table->id();
                $table->timestamps();
            });
    }

    /**
     * Reverse the migrations.
     */
    public function down(): void
    {
        Schema::connection('mysql_second')->dropIfExists('reports');
    }
};
```
All schema operations become connection-aware simply by chaining `connection('connection_name')`. For example:
```php
use Phaseolies\Support\Facades\Schema;

if (Schema::connection('mysql_second')->hasTable('user')) {
    // Perform some logic if the 'user' table exists in mysql_second
}

// Drop a table on a different connection
Schema::connection('archive_db')->dropIfExists('logs');

// Create a table with foreign key checks disabled
Schema::connection('tenant_db')->disableForeignKeyConstraints();

Schema::connection('tenant_db')
    ->create('orders', function (Blueprint $table) {
        $table->id();
        $table->timestamps();
    });

Schema::connection('tenant_db')->enableForeignKeyConstraints();
```

The `Schema::connection()` method makes it simple to work with multiple databases in Doppar. Whether you're creating tables, checking schema state, or managing constraints, you can confidently direct all schema commands to the right database with clean and expressive syntax.