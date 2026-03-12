---
title: Database Migration
description: Database Migration documentation
meta:
  - name: keywords
    content: Database Migration
---

## Database Migration

### Introduction

Managing database schema changes manually is error-prone and difficult to coordinate 
across teams and environments. The migration system solves this by providing a 
structured, version-controlled way to evolve your database schema over time — keeping 
development, staging, and production environments in sync without the risk of 
inconsistency or data loss.

## Creating a Migration

Migrations are generated using the `make:migration` command from the command line. 
Each migration file is automatically timestamped and stored in the `database/migrations` 
directory, ensuring they are executed in the correct order.

To generate a new empty migration file:
```bash
php pool make:migration create
```

To create a migration with a specific name and target table:
```bash
php pool make:migration create_users_table --create=users
```

- `create_users_table` — the name of the migration file
- `--create=users` — indicates this migration will create a `users` table

## Defining a Schema

Each migration file returns an anonymous class extending `Migration` with two methods. 
The `up()` method applies the schema change, and the `down()` method reverses it. 
This allows you to both migrate forward and roll back changes safely.
```php
<?php

use Phaseolies\Support\Facades\Schema;
use Phaseolies\Database\Migration\Blueprint;
use Phaseolies\Database\Migration\Migration;

return new class extends Migration
{
    public function up(): void
    {
        Schema::create('users', function (Blueprint $table) {
            $table->id();
            $table->string('name');
            $table->string('email')->unique();
            $table->string('password', 100);
            $table->string('remember_token', 100)->nullable();
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

## Running Migrations

Once your migration files are defined, you can apply all pending migrations with a 
single command. The system tracks which migrations have already been run, so only 
new ones are executed each time.

Run all pending migrations:
```bash
php pool migrate
```

Run migrations on a specific database connection:
```bash
php pool migrate --connection=mysql
```

## Adding Columns to an Existing Table

As your application evolves, you will often need to add new columns to tables that 
already exist in the database. Rather than modifying the original migration, you 
create a new migration that targets the existing table, preserving your migration 
history.

Generate a migration targeting an existing table:
```bash
php pool make:migration add_company_name --table=users
```

Then use `Schema::table()` to modify the existing table:
```php
Schema::table('users', function (Blueprint $table) {
    $table->string('company_name', 50)->after('email');
});
```

> **Note:** The `after()` modifier is not supported by PostgreSQL. When used with 
> a PostgreSQL connection, it will be silently ignored.

> **Warning:** When adding columns to an existing table, you must use the `after()` 
> method. Adding multiple columns in a single migration without `after()` may not 
> execute correctly.

## Running a Specific Migration

There are situations where you only want to execute one particular migration file 
without running the entire pending batch. This is useful when testing a single 
schema change or applying a hotfix to a specific table.

Run a single migration file:
```bash
php pool migrate --path=/database/migrations/your_migration_file.php
```

Run a specific migration on a specific connection:
```bash
php pool migrate --connection=mysql --path=/your_migration_file.php
```

## Refreshing Migrations

The `migrate:fresh` command is a powerful tool for resetting your entire database 
schema. It drops all existing tables and re-runs every migration from the beginning, 
giving you a completely clean database state. This is particularly useful during 
development when you need to rebuild the schema from scratch.

Reset and re-run all migrations:
```bash
php pool migrate:fresh
```

Refresh a specific database connection:
```bash
php pool migrate:fresh --connection=mysql
```

## Column Types

The migration system provides a comprehensive set of column types to cover every 
common data storage need. Each type maps to the most appropriate native database 
type for the active driver, so your migrations remain portable across MySQL, 
PostgreSQL, and SQLite.

### Primary Key

A primary key uniquely identifies each row in a table. The `id()` method creates 
an auto-incrementing unsigned big integer column and sets it as the primary key. 
For use cases that require a non-integer identifier, a UUID primary key is also 
supported.

Auto-incrementing unsigned big integer primary key
```php
$table->id();
```

With a custom column name
```php
$table->id('user_id');
```

UUID primary key
```php
$table->uuid('id')->primary();
```
This stored as CHAR(36) on MySQL, UUID on PostgreSQL, TEXT on SQLite

### String Types

String columns are used for storing text data of varying lengths. Choose the 
appropriate type based on the expected size of the content — using a more 
specific type can improve storage efficiency and query performance.

Variable-length string, default maximum of 255 characters
```php
$table->string('title');
```
With a custom length
```php
$table->string('title', 100);
```
Fixed-length string, padded to the defined length
```php
$table->char('code', 10);
```

Text columns for progressively larger content
```php
$table->tinyText('excerpt');    // up to 255 characters
$table->text('body');           // up to 65,535 characters
$table->mediumText('content');  // up to 16MB
$table->longText('article');    // up to 4GB
```

### Numeric Types

Numeric columns store integer and floating point data. Signed types can hold both 
positive and negative values, while unsigned types are restricted to non-negative 
values and offer a higher positive range. For financial data, always prefer 
`decimal` over `float` or `double` to avoid rounding errors.

Signed integers, ranging from smallest to largest storage size
```php
$table->tinyInteger('score');       // 1 byte
$table->smallInteger('views');      // 2 bytes
$table->mediumInteger('clicks');    // 3 bytes
$table->integer('count');           // 4 bytes
$table->bigInteger('total');        // 8 bytes
```

Unsigned integers — non-negative values only, larger positive range
```php
$table->unsignedTinyInteger('level');
$table->unsignedSmallInteger('rank');
$table->unsignedMediumInteger('hits');
$table->unsignedInteger('votes');
$table->unsignedBigInteger('user_id');
```

Floating point and exact numeric types
```php
$table->float('rate', 8, 2); // approximate, 8 digits total, 2 decimal places
$table->double('coefficient', 15, 8); // higher precision float
$table->decimal('amount', 10, 2);   // exact numeric, recommended for currency
```

### Boolean

Boolean columns store binary true/false values and are commonly used for feature 
flags, status indicators, and permission checks. The underlying storage type 
varies by driver but behaves consistently as `true` or `false` at the application 
level.
```php
$table->boolean('is_active');
```

This Stored as TINYINT(1) on MySQL, BOOLEAN on PostgreSQL, INTEGER on SQLite

### Date and Time Types

Date and time columns cover the full range of temporal data needs, from storing 
just a date or time value to full timestamps with timezone awareness. Choose 
timezone-aware variants when your application serves users across multiple 
time zones.
```php
$table->date('published_on');           // Date only (YYYY-MM-DD)
$table->time('opens_at');               // Time only (HH:MM:SS)
$table->timeTz('opens_at_tz');          // Time only, with timezone
$table->dateTime('scheduled_at');       // Date and time, no timezone
$table->dateTimeTz('scheduled_at_tz'); // Date and time, with timezone
$table->timestamp('processed_at');      // Timestamp, no timezone
$table->timestampTz('processed_at_tz'); // Timestamp, with timezone
$table->year('graduation_year');        // Year value only
```

Shorthand timestamp columns. Adds nullable `created_at` and `updated_at` columns
```php
$table->timestamps();
```

Same as timestamps() but with timezone awareness
```php
$table->timestampsTz();
```

Adds a nullable deleted_at column for soft deletion
```php
$table->softDeletes();
```

### Binary Types

Binary columns store raw binary data such as files, images, or serialized content. 
Use the most appropriately sized type for your data to avoid unnecessary storage 
overhead. For very large files, consider storing them on the filesystem and saving 
only the path in the database.
```php
$table->binary('data');         // BLOB, up to 65,535 bytes
$table->tinyBlob('thumbnail');  // Up to 255 bytes
$table->blob('image');          // Up to 65,535 bytes
$table->mediumBlob('document'); // Up to 16MB
$table->longBlob('video');      // Up to 4GB
```

### JSON

JSON columns allow you to store structured data — such as configuration objects, 
arrays, or key-value pairs — directly in the database without requiring a separate 
table. This is particularly useful for data with a flexible or variable structure.

Standard JSON column
```php
$table->json('settings');
```

Binary JSON on PostgreSQL; falls back to JSON on other drivers
```php
$table->jsonb('metadata');
```

Array-compatible JSON column
```php
$table->jsonArray('tags');
```

### Special Types

These column types handle specific data formats that don't fit neatly into the 
standard string or numeric categories. They provide built-in validation semantics 
at the database level and improve both clarity and data integrity.

 A column restricted to a predefined set of string values
```php
$table->enum('status', ['active', 'inactive', 'pending']);
```

You can define nullable enum
```php
$table->enumNullable('role', ['admin', 'editor', 'viewer']);
```

Allows storing multiple values from a predefined set in one column (MySQL only). Falls back to TEXT on other drivers
```php
$table->set('permissions', ['read', 'write', 'delete']);
```

UUID stored as CHAR(36) on MySQL, UUID on PostgreSQL, TEXT on SQLite
```php
$table->uuid('reference_id');
```

Network address types
```php
$table->ipAddress('last_login_ip');  // Stores IPv4 and IPv6 addresses
$table->macAddress('device_mac');    // Stores MAC addresses
```

Bit flag column, useful for compact storage of multiple boolean flags
```php
$table->bit('flags', 8);
```

### Spatial / GIS Types

Spatial columns store geometric and geographic data for use with GIS (Geographic 
Information System) operations. These types enable advanced spatial queries such 
as distance calculations, boundary checks, and region overlaps. Full spatial 
support depends on your database driver and installed extensions.
```php
$table->geometry('shape');                       // Any geometry type
$table->point('location');                       // A single coordinate point
$table->lineString('route');                     // A sequence of points forming a line
$table->polygon('boundary');                     // A closed shape defined by multiple points
$table->geometryCollection('regions');           // A collection of mixed geometry objects
$table->multiPoint('stops');                     // Multiple point geometries
$table->multiLineString('paths');                // Multiple linestring geometries
$table->multiPolygon('zones');                   // Multiple polygon geometries
```

## Column Modifiers

Column modifiers refine the behavior and constraints of a column definition. They 
are chained directly onto the column method and can be combined as needed. Modifiers 
affect nullability, default values, indexing, and column positioning.

Allow the column to store NULL values
```php
$table->string('middle_name')->nullable();
```

Set a default value when none is provided
```php
$table->boolean('is_active')->default(true);
$table->string('role')->default('editor');
$table->integer('score')->default(0);
```

Enforce uniqueness across all rows for this column
```php
$table->string('email')->unique();
```

Add a plain (non-unique) index to improve query performance
```php
$table->string('slug')->index();
```

Position the column immediately after another column (MySQL only).
```php
$table->string('company')->after('email');
```
>  Silently ignored on PostgreSQL

## Checking If a Table Exists

Before modifying a table or running conditional schema logic, you may want to 
verify that a table actually exists in the database. The `hasTable()` method 
returns a boolean and works with any configured connection, making it safe to 
use in both migrations and application code.
```php
use Phaseolies\Support\Facades\Schema;

if (Schema::hasTable('users')) {
    // The table exists — safe to proceed
}
```

## Dropping and Truncating Tables

Sometimes you need to completely remove a table or clear all its data during 
development or as part of a migration rollback. The following methods allow you 
to drop or truncate tables directly using an expressive, chainable syntax.

### Dropping a Table

Dropping a table removes it entirely from the database along with all its data 
and structure. This operation is irreversible, so use it with caution in 
production environments.
```php
use Phaseolies\Support\Facades\DB;

DB::table('users')->drop();
```

### Truncating a Table

Truncating a table removes all rows but keeps the table structure intact. This 
is faster than individually deleting all rows and is commonly used to reset 
data during testing or local development.

Remove all rows but preserve the table structure
```php
DB::table('users')->truncate();
```

Remove all rows and reset the auto-increment counter back to 1
```php
DB::table('users')->truncate(true);
```

## Foreign Key Constraints

Foreign keys enforce referential integrity between tables by ensuring that a 
value in one table must correspond to an existing value in another. The migration 
system provides a clean, expressive API for defining and managing foreign key 
constraints that works consistently across all supported database drivers.

### Disabling and Enabling Constraints

When performing bulk operations such as seeding, truncating, or importing data, 
you may need to temporarily disable foreign key constraint checks to avoid 
integrity errors. Always re-enable constraints immediately after your operation 
completes to maintain data integrity.
```php
use Phaseolies\Support\Facades\Schema;

Schema::disableForeignKeyConstraints();
// ... perform bulk operation
Schema::enableForeignKeyConstraints();
```

### Defining Foreign Keys

There are two ways to define a foreign key: a shorthand method that infers the 
column name and referenced table from a model class, or an explicit definition 
that gives you full control over every aspect of the constraint.

Shorthand — infers column name and referenced table from the model
```php
use App\Models\User;

Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->foreignIdFor(User::class)->nullable();
});
```

Explicit definition — full control over column and reference
```php
use App\Models\User;

Schema::create('posts', function (Blueprint $table) {
    $table->id();
    $table->unsignedBigInteger('user_id');
    $table->foreign('user_id')->references('id')->on('users');
});
```
### Cascade Actions

Cascade actions define what happens to related rows when a referenced record is 
updated or deleted. Setting the appropriate cascade action is important for 
maintaining data consistency and avoiding orphaned records in your database.

Via foreignIdFor shorthand (first bool = onDeleteCascade, second = onUpdateCascade)
```php
$table->foreignIdFor(User::class, true, true);
```

Inline cascade definition
```php
$table->foreign('user_id')
    ->references('id')
    ->on('users')
    ->onDelete('CASCADE');
```

Named shortcut methods for common cascade behaviors
```php
$table->foreign('user_id')->references('id')->on('users')->cascadeOnDelete();
$table->foreign('user_id')->references('id')->on('users')->restrictOnDelete();
$table->foreign('user_id')->references('id')->on('users')->nullOnDelete();
$table->foreign('user_id')->references('id')->on('users')->cascadeOnUpdate();
$table->foreign('user_id')->references('id')->on('users')->restrictOnUpdate();
$table->foreign('user_id')->references('id')->on('users')->nullOnUpdate();
```

## Running Migrations on a Specific Connection

For applications that use multiple databases — such as multi-tenant systems, 
modular architectures, or reporting databases — you can target any configured 
connection when running migrations. The `Schema::connection()` method returns 
a schema builder instance configured for the specified connection, and all 
operations chain naturally from it.
```php
return new class extends Migration
{
    public function up(): void
    {
        Schema::connection('mysql_second')->create('reports', function (Blueprint $table) {
            $table->id();
            $table->string('title');
            $table->timestamps();
        });
    }

    public function down(): void
    {
        Schema::connection('mysql_second')->dropIfExists('reports');
    }
};
```

All schema operations support connection chaining, giving you full control over 
which database each operation targets:

Check whether a table exists on a specific connection
```php
use Phaseolies\Support\Facades\Schema;

if (Schema::connection('mysql_second')->hasTable('users')) {
    //
}
```

Drop a table on a specific connection
```php
Schema::connection('archive_db')->dropIfExists('logs');
```

Manage foreign key constraints on a specific connection
```php
Schema::connection('tenant_db')->disableForeignKeyConstraints();

Schema::connection('tenant_db')->create('orders', function (Blueprint $table) {
    $table->id();
    $table->timestamps();
});

Schema::connection('tenant_db')->enableForeignKeyConstraints();
```