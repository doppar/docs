## Database Migration
### Introduction
Managing database changes by hand is risky and error-prone. Doppar's migration system takes the guesswork out of the equation by automating and organizing your schema changes — so you can focus more on building features and less on database headaches.

Doppar provides a seamless and structured way to manage your database migrations, making it easy to evolve your database schema over time. With Doppar’s migration system, you can track changes, collaborate with your team, and keep your development, staging, and production environments in sync.

### Creating a Migration
To generate a new migration file in Doppar, use the `make:migration` command provided by the Pool Console. This command will scaffold a new migration class in the `database/migrations` directory.
```bash
php pool make:migration create_users_table --create=users
```
This commands
- `create_users_table`: The name of the migration file
- `--create=users`: Indicates that this migration is intended to create a users table

Once created, you can define the table structure inside the generated file using Doppar’s schema builder.

### Define Basic Schema
Here, `users` is the table name. This command will generate a new migration file by returning an anonymous class instance. Now assume we are going to update this for define our database schema.

```php
<?php

use Phaseolies\Support\Facades\Schema;
use Phaseolies\Database\Migration\Blueprint;
use Phaseolies\Database\Migration\Migration;

return new class extends Migration
{
    /**
     * Run the migrations
     *
     * @return void
     */
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

    /**
     * Reverse the migrations
     *
     * @return void
     */
    public function down(): void
    {
        Schema::dropIfExists('users');
    }
};
```

### Run Migration
To migrate your all file or newly created migration files, run this migrate command
```php
php pool migrate
```

### Available Fields
Let's see all the available columns types and options
#### id()
Creates an auto-incrementing primary key (unsigned big integer).
```php
Schema::create('users', function (Blueprint $table) {
    $table->id();
});
```

#### string()
Defines a column with the `VARCHAR` type, ideal for storing short strings like names, titles, or labels. By default, the maximum length is 255 characters.
```php
Schema::create('users', function (Blueprint $table) {
    $table->string('title');
    // with custom length
    $table->string('title', 100);
});
```

#### char()
Defines a column with the `CHAR` type, ideal for storing short strings like names, titles, or labels. By default, the maximum length is 255 characters.
```php
Schema::create('users', function (Blueprint $table) {
    $table->char('name');
});
```

#### tinyText()
Defines a column with the `TINYTEXT` type, ideal for storing small blocks of text such as summaries, excerpts, or content.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->tinyText('excerpt');
});
```

#### mediumText()
Defines a column with the `MEDIUMTEXT` type, ideal for storing longer blocks of text such as summaries, excerpts, or content that exceeds the 255-character limit of a string.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->mediumText('body');
});
```

#### text()
Defines a column with the `TEXT` type, ideal for storing longer blocks of text such as summaries, excerpts, or content that exceeds the 255-character limit of a string.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->text('body');
});
```

#### longText()
Defines a column with the `LONGTEXT` type, perfect for storing very large amounts of text, such as full articles, blog post content, or rich HTML.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->longText('description');
});
```

#### json()
Defines a column with the `JSON` type, ideal for storing structured data like arrays, objects, or key-value pairs. Useful when the data format is dynamic or flexible.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->json('attributes');
});
```

#### integer()
Defines a column with the `INT` type, ideal for storing whole numbers such as counts, IDs, or any data that doesn’t require decimal precision.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->integer('post_views');
});
```

#### boolean()
Defines a column with the `BOOLEAN` type, which stores binary values: true (1) or false (0). Typically used for flags or status indicators, such as whether a post is active or published.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->boolean('status');
});
```

#### timestamps()
Defines two columns: `created_at` and `updated_at`. These are automatically managed by Eloquent to track when a record is created and last updated.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->timestamps();
});
```

#### unique()
Enforces a unique constraint on a column, ensuring that no two rows can have the same value for that column. It's often used on columns like email addresses, slugs, or usernames to maintain data integrity.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->string('slug')->unique();
});
```

#### nullable()
Allows a column to accept `null` values. By default, columns in Doppar are required, but using `nullable()` makes the column optional, meaning it can store `NULL` values.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->string('excerpt')->nullable();
});
```

#### default()
Sets a default value for a column if no value is provided during the creation of a record. This is useful for ensuring a column has a predetermined value when it’s not explicitly set.
```php
Schema::create('posts', function (Blueprint $table) {
    $table->boolean('status')->default(true);
});
```

#### Number Types
Doppar migration includes various numeric column types to handle different ranges and precisions of data. Here's a quick overview:
```php
// Signed Integers:
$table->tinyInteger('tiny_int_column'); // 1 byte, range: -128 to 127
$table->smallInteger('small_int_column'); // 2 bytes, range: -32,768 to 32,767
$table->mediumInteger('medium_int_column'); // 3 bytes, range: -8,388,608 to 8,388,607
$table->integer('int_column'); // 4 bytes, range: -2,147,483,648 to 2,147,483,647
$table->bigInteger('big_int_column'); // 8 bytes, range: -9.2 quintillion to 9.2 quintillion

// Unsigned Integers (no negative values):
$table->unsignedInteger('unsigned_int_column'); // 0 to 255
$table->unsignedTinyInteger('unsigned_tiny_int_column'); // 0 to 65,535
$table->unsignedSmallInteger('unsigned_small_int_column'); // 0 to 16,777,215
$table->unsignedMediumInteger('unsigned_medium_int_column'); // 0 to 4,294,967,295

// Floating Point Types:
$table->float('float_column', 8, 2); // Approximate numeric, total 8 digits, 2 after decimal
$table->double('double_column', 15, 8); // Higher precision float, total 15 digits, 8 after decimal
$table->decimal('decimal_column', 10, 2); // Exact numeric, total 10 digits, 2 after decimal (ideal for currency)
```

#### Date/Time Types
Doppar migration includes a variety of date and time-related columns to cover different temporal data needs:

```php
$table->date('date_column'); // Stores only the date (format: YYYY-MM-DD)
$table->dateTime('datetime_column'); //  Stores date and time (format: YYYY-MM-DD HH:MM:SS)
$table->dateTimeTz('datetime_tz_column'); // Like dateTime, but includes time zone support
$table->time('time_column'); // Stores only time (format: HH:MM:SS)

// Time Zone Aware Columns:
$table->timeTz('time_tz_column'); //  Like time, but with time zone awareness
$table->timestamp('timestamp_column'); // Stores date and time, often used for tracking created/updated times
$table->timestampTz('timestamp_tz_column'); // Time-stamped with time zone support

$table->year('year_column');
$table->softDeletes(); // Adds a deleted_at timestamp column
```

#### Binary Types
Doppar migration defines several binary and BLOB (Binary Large Object) column types to handle various sizes of binary data:
```php
// Standard Binary:
$table->binary('binary_column'); // Creates a BLOB column suitable for storing small binary data (up to 65,535 bytes). 

// Extended BLOB Types (MySQL-specific):
$table->tinyBlob('tiny_blob_column'); // Stores up to 255 bytes.
$table->blob('blob_column'); // Stores up to 65,535 bytes.
$table->mediumBlob('medium_blob_column'); // Stores up to 16 MB.
$table->longBlob('long_blob_column'); // Stores up to 4 GB
```

#### Special Types
Doppar migration utilizes several specialized column types to handle unique data requirements:
```php
// Defines a column with a set of predefined string  values. Commonly used for status indicators or categorical data.
$table->enum('enum_column', ['active', 'pending', 'cancelled']); 

// Allows storage of multiple values from a predefined list in a single column.
$table->set('set_column', ['red', 'green', 'blue']);

$table->uuid('uuid_column'); // Creates a column to store Universally Unique Identifiers (UUIDs).

$table->ipAddress('ip_address_column'); // Stores IPv4 and IPv6 addresses.
$table->macAddress('mac_address_column'); // Stores MAC addresses. Typically stored as strings in the format 00:00:00:00:00:00
$table->json('json_column'); // Stores JSON-formatted data. Supported in MySQL 5.7+
```

#### Spatial Types (GIS)
Doppar migration utilizes several specialized column types to handle geospatial data, enabling advanced geographical queries and operations:
```php
$table->geometry('geometry_column'); // Stores any type of geometry data.
$table->point('point_column'); // Represents a single location in coordinate space (latitude and longitude).
$table->lineString('line_string_column'); // Stores a sequence of points forming a continuous line.
$table->polygon('polygon_column'); // Defines a shape consisting of multiple points forming a closed loop.
$table->geometryCollection('geometry_collection_column'); // Stores a collection of geometry objects.
$table->multiPoint('multi_point_column'); // Stores multiple point geometries.
$table->multiLineString('multi_line_string_column'); // Stores multiple linestring geometries.
$table->multiPolygon('multi_polygon_column'); // Stores multiple polygon geometries.
```

#### Determining Whether a Table Exists
Before modifying or interacting with a database table, you may want to check if it exists. Doppar provides the `hasTable()` method via the Schema facade to perform this check.
```php
use Phaseolies\Support\Facades\Schema;

public function up()
{
    if (Schema::hasTable('users')) {
        // do something
    }
}
```

####  Dropping a Table
Tables can be dropped quite easily using the drop() method.

```php
use Phaseolies\Support\Facades\DB;

DB::table('users')->drop()
```

#### Truncate a Table
Tables can be Truncated easily using the `truncate()` method.
```php
use Phaseolies\Support\Facades\DB;

DB::table('users')->truncate()
DB::table('users')->truncate(true) // passing true mean force reset auto increment
```

#### Enable Disable Foreign Key Constraints
You can easily disable and enable foreign key constraints using Schema facades by calling `disableForeignKeyConstraints()` method to disable and `enableForeignKeyConstraints()` method to enable like
```php
use Phaseolies\Support\Facades\Schema;

Schema::disableForeignKeyConstraints();
Schema::enableForeignKeyConstraints();
```

#### Working With Foreign Keys
For creating foreign key constraints on your database tables. Let’s add a foreign key to an example table:
```php

use Phaseolies\Support\Facades\Schema;
use App\Models\User;

public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->foreignIdFor(User::class)->nullable();

        // or you can use like that
        $table->unsignedBigInteger('user_id');
        $table->foreign('user_id')->references('id')->on('users');
    })
}
```
In Doppar, when defining foreign key constraints within your migrations, you can specify actions to be taken when the referenced record is deleted. This ensures referential integrity and allows for automatic management of related records.

#### Setting Up CASCADE Actions
There are three primary ways to define `CASCADE` actions in Doppar migrations:
```php
public function up()
{
     // true for onDeleteCascade and true for onUpdateCascade
     $table->foreignIdFor(User::class, true, true);

     // Or
     $table->unsignedBigInteger('user_id');
     $table->foreign('user_id')
         ->references('id')
         ->on('users')
         ->onDelete('CASCADE');

     // Using the cascadeOnDelete Shortcut:
     $table->unsignedBigInteger('user_id');
     $table->foreign('user_id')
         ->references('id')
         ->on('users')
         ->cascadeOnDelete();
}
```
You can also use this method `cascadeOnDelete()`, `restrictOnDelete()`, `nullOnDelete()`, `cascadeOnUpdate()`, `restrictOnUpdate()`, `nullOnUpdate()`.

### Refreshing Migrations
The `php pool migrate:fresh` command is a powerful tool in Doppar that allows you to reset and re-run all your migrations. This is particularly useful during development when you need to rebuild your database schema without manually rolling back and reapplying each migration.

When you run:
```php
php pool migrate:fresh
```
Doppar performs the following actions
- Rolls back all existing migrations by executing the down() methods.
- Re-runs all migrations by executing the up() methods

This process effectively rebuilds your entire database schema.






