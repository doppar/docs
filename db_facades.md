## DB Facade: Getting Started
### Introduction
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
Doppar’s query() method allows execution of raw SQL queries with safe bindings.
#### Get One Row
```php
<?php

use Phaseolies\Support\Facades\DB

DB::query("SELECT * FROM user WHERE name = ?", ['Kasper Snider'])->fetch();
```
Executes a query and returns the first matching row as an associative array.

#### Get All Matching Rows
```php
DB::query("SELECT * FROM user WHERE name = ?", ['Kasper Snider'])->fetchAll();
```

### Modifying the Database using execute()
Doppar provides `execute()` function and using it, you can modify database. See the basic example. Inserts a new user record. Returns the number of affected rows (1 if successful).
```php
$inserted = DB::execute(
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

### Executing Stored Procedures
Doppar supports stored procedure execution with executeProcedure().
```php
// Get nested results (original behavior)
$nestedResults = DB::executeProcedure('sp_GetAllUsers')->all();
```

#### Get flattened first result set
```php
$users = DB::executeProcedure('sp_GetAllUsers')->flatten();
```
Calls a procedure and flattens the first result set into a simple array.

#### Get first row only
```php
$firstUser = DB::executeProcedure('sp_GetUserById', [1])->first();
```

Get the second result set (index 1) by passing 123 parameter
```php
$stats = DB::executeProcedure('sp_GetUserWithStats', [123])->resultSet(1);
```

#### With Multiple Params
```php
$totalUsers = 0;
$results = DB::executeProcedure(
    'sp_GetPaginatedUsers',
    [1, 10, 'active', 'created_at', 'DESC'],
    [&$totalUsers]
)->all();
// $results[0] contains user data
// $totalUsers contains total count
```

#### Executing View
Doppar provides executeView method to run your view. See the basic example
```php
$stats = DB::executeView('vw_user_statistics');
```

#### View with WHERE Conditions
You can pass where condition in your custom view like
```php
// View with single WHERE condition
$nyUsers = DB::executeView(
    'vw_user_locations', 
    ['state' => 'New York']
);

// Equivalent to: SELECT * FROM vw_user_locations WHERE state = 'New York'

// View with multiple WHERE conditions
$premiumNyUsers = DB::executeView(
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
$recentOrders = DB::executeView(
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
| Stored Procedures | `executeProcedure()`           | Call stored procs with multiple results      |
| Views             | `executeView()`                | Query DB views with filters and bindings     |
| Utilities         | `getTables()`, `tableExists()` | Explore schema                               |

Doppar's DB Facade provides all the raw SQL power you need while preserving the developer-friendly syntax you love.
