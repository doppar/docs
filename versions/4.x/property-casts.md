---
title: Attribute Casting
description: Doppar Attribute Casting
meta:
- name: keywords
  content: Attribute Casting, Model, Doppar
---

## Attribute Casting
### Introduction

Attribute casting allows you to automatically transform model attribute values when reading from or writing to the database — without cluttering your application code with manual type conversions.

Databases store everything as strings, integers, or raw bytes. Without casting, your PHP code is littered with `(bool)`, `(int)`, `json_decode()`, and `Carbon::parse()` calls scattered across controllers and services. Doppar's casting system centralises all of this transformation logic on the model itself — declared once, applied everywhere.

## Defining Casts

### Shorthand Attributes

Doppar provides a dedicated shorthand attribute for every built-in cast type. Each `To*` attribute maps directly to one cast type and requires no arguments — just annotate the property and you are done.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Casts\Attributes\ToBoolean;
use Phaseolies\Database\Entity\Casts\Attributes\ToDateTime;
use Phaseolies\Database\Entity\Casts\Attributes\ToArray;
use Phaseolies\Database\Entity\Casts\Attributes\ToString;
use Phaseolies\Database\Entity\Casts\Attributes\ToInteger;
use Phaseolies\Database\Entity\Casts\Attributes\ToCollection;

class Order extends Model
{
    #[ToBoolean]
    protected string $is_paid;

    #[ToDateTime]
    protected string $shipped_at;

    #[ToArray]
    protected string $meta;

    #[ToString]
    protected string $reference;

    #[ToInteger]
    protected string $item_count;

    #[ToCollection]
    protected string $line_items;
}
```

### Transform Attribute

For types that require an argument — parameterised formats, enum classes, or custom cast classes — use the general-purpose `#[Transform]` attribute. It accepts:

- A `Type` enum case: `#[Transform(Type::DateTime)]`
- A parameterised type string: `#[Transform('decimal:2')]`, `#[Transform('datetime:d/m/Y')]`
- A backed or unit enum class: `#[Transform(OrderStatus::class)]`
- Any class implementing `CastableInterface`: `#[Transform(MoneyCast::class)]`

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;
use App\Enums\OrderStatus;
use App\Enums\Priority;

class Order extends Model
{
    // Using the Type enum case
    #[Transform(Type::Boolean)]
    protected string $is_paid;

    // Parameterised decimal with 2 places
    #[Transform('decimal:2')]
    protected string $total;

    // Parameterised datetime with custom write format
    #[Transform('datetime:d/m/Y')]
    protected string $invoiced_at;

    // Backed enum class
    #[Transform(OrderStatus::class)]
    protected string $status;

    // Another enum
    #[Transform(Priority::class)]
    protected string $priority;
}
```

All `To*` shorthand attributes are subclasses of `Transform` and are resolved by the same reflection scanner. They are fully interchangeable — use whichever form reads most clearly at the call-site.

### Why protected?

Cast properties **must be declared `protected` or `private`**, never `public`. Here is why:

When a property is `public`, PHP resolves `$model->property` as a direct property access and skips `__get()` entirely. The property would be uninitialized (since Doppar never sets typed properties directly — it stores everything in the internal `$attributes` array), and PHP would throw a `TypeError: Cannot access uninitialized property`.

With `protected` or `private`, external access to `$model->property` is routed through the `__get()` magic method, which reads from `$attributes` and applies the cast. This is exactly the behaviour you want.

```php
// Correct — protected, external access goes through __get
#[ToDateTime]
protected string $published_at;

// Also fine — private works the same way
#[ToString]
private string $reference;

// Wrong — public bypasses __get, causes TypeError on access
#[ToDateTime]
public string $published_at;
```

## Available Cast Types

The `Type` enum lists every built-in cast type as a named constant.

| Shorthand Attribute | `Type` Enum Case | PHP Type on Read | Description |
|---|---|---|---|
| `#[ToString]` | `Type::String` | `string` | Any value → PHP string |
| `#[ToInteger]` | `Type::Integer` | `int` | Any value → PHP int |
| `#[ToFloat]` | `Type::Float` | `float` | Any value → PHP float |
| `#[ToBoolean]` | `Type::Boolean` | `bool` | `"1"`, `"true"`, `"yes"`, `"on"` → `true`; everything else → `false` |
| `#[ToArray]` | `Type::Array` | `array` | JSON string → PHP array |
| `#[ToJson]` | `Type::Json` | `array` | JSON string → PHP array (alias of Array) |
| `#[ToObject]` | `Type::Object` | `stdClass` | JSON string → PHP stdClass |
| `#[ToCollection]` | `Type::Collection` | `Collection` | JSON string → Doppar Collection |
| `#[ToDateTime]` | `Type::DateTime` | `Carbon` | Date string → Carbon (default format `Y-m-d H:i:s`) |
| `#[ToDate]` | `Type::Date` | `Carbon` | Date string → Carbon (time zeroed) |
| `#[ToTimestamp]` | `Type::Timestamp` | `Carbon` | Unix int → Carbon on read; Carbon → Unix int on write |

Additionally, `#[Transform]` supports parameterised and dynamic types:

| Syntax | Returns on Read | Notes |
|---|---|---|
| `#[Transform('decimal:N')]` | `float` | Rounds to N decimal places on both read and write |
| `#[Transform('datetime:FORMAT')]` | `Carbon` | FORMAT controls write only; always Carbon on read |
| `#[Transform(MyBackedEnum::class)]` | Enum case | `MyEnum::from($value)` on read; `->value` on write |
| `#[Transform(MyUnitEnum::class)]` | Enum case | Matches by `->name` on read; stores `->name` on write |
| `#[Transform(MyCast::class)]` | Mixed | Your class must implement `CastableInterface` |

## Primitive Casts

### Integer

Casts the raw database value to a native PHP `int`.

**When to use it:** Database drivers often return numeric columns as strings (e.g., `"42"` instead of `42`). Without casting, `$post->views === 42` would fail because `"42" !== 42`. The `#[ToInteger]` cast guarantees you always get a true PHP integer.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToInteger;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Post extends Model
{
    #[ToInteger]
    protected string $views;

    #[ToInteger]
    protected string $comment_count;

    #[ToInteger]
    protected string $score;

    // Equivalent using Transform
    #[Transform(Type::Integer)]
    protected string $rank;
}
```

See the read and write example:

```php
$post = Post::find(1);

$post->views;         // int(1500)   — not "1500"
$post->comment_count; // int(42)     — not "42"
$post->score;         // int(95)     — not "95"

// Type-safe comparison
$post->views === 1500; // true  (would fail without cast)

// Arithmetic works correctly
$totalEngagement = $post->views + $post->comment_count; // int

// Writing
$post->views = '999';  // stored as string, returned as int(999) on next read
$post->score = 100.9;  // float truncated to int(100) by PHP

is_int($post->views); // true
```

### Float

Casts the raw database value to a PHP `float`.

**When to use it:** Use `#[ToFloat]` for numeric values where decimal precision is acceptable and floating-point arithmetic is fine — things like coordinates, ratings, or measurements. For money or financial calculations where you need exact decimal rounding, use `#[Transform('decimal:N')]` instead.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToFloat;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Product extends Model
{
    #[ToFloat]
    protected string $weight;

    #[ToFloat]
    protected string $rating;

    #[ToFloat]
    protected string $latitude;

    // Equivalent using Transform
    #[Transform(Type::Float)]
    protected string $longitude;
}
```

> **Note:** For currency, prices, or any financial value, use `#[Transform('decimal:2')]` instead. `float` in PHP is subject to IEEE 754 rounding errors (e.g., `0.1 + 0.2 !== 0.3`), which can cause incorrect monetary calculations.

### Boolean

Casts the raw database value to a native PHP `bool`. Handles every common database representation of true and false.

**When to use it:** Boolean columns are typically stored as `TINYINT(1)` in MySQL (returning `"1"` or `"0"` as strings) or as literal `true`/`false` in PostgreSQL. Without casting, `$post->is_published == true` still works (loose comparison), but `$post->is_published === true` fails because `"1" !== true`. Casting makes type-safe comparisons and `if` conditions reliable.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToBoolean;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Post extends Model
{
    #[ToBoolean]
    protected string $is_published;

    #[ToBoolean]
    protected string $allow_comments;

    #[ToBoolean]
    protected string $is_featured;

    // Equivalent using Transform
    #[Transform(Type::Boolean)]
    protected string $requires_approval;
}
```

See the read and write example of boolean cast:
```php
$post = Post::find(1);

$post->is_published;   // bool(true)   — not "1"
$post->allow_comments; // bool(false)  — not "0"

// Type-safe comparison
$post->is_published === true;  // true  (would fail without cast)
$post->allow_comments === false; // true

// Direct use in conditions
if ($post->is_published && $post->allow_comments) {
    // render comments section
}

// Writing
$post->is_published = true;   // stored as 1
$post->is_published = false;  // stored as 0
$post->is_published = 1;      // stored as 1, returned as true

is_bool($post->is_published); // true
```

**Truthy values** (any of these → `true`):
`1`, `"1"`, `true`, `"true"`, `"yes"`, `"on"`

**Falsy values** (anything else → `false`):
`0`, `"0"`, `false`, `"false"`, `"no"`, `"off"`, `""`, `null`

### String

Casts the raw database value to a PHP `string`.

**When to use it:** Some database drivers return numeric-looking columns as integers or floats. For example, a `zip_code` column containing `"01234"` might be returned as the integer `1234`, losing the leading zero. `#[ToString]` ensures the value always arrives as a string, preserving leading zeros, formatting, and preventing type-related bugs in string functions.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToString;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class User extends Model
{
    #[ToString]
    protected string $phone;

    #[ToString]
    protected string $postal_code;

    #[ToString]
    protected string $national_id;

    // Equivalent using Transform
    #[Transform(Type::String)]
    protected string $reference_number;
}
```

## Array & JSON Casts

`#[ToArray]` and `#[ToJson]` both store and retrieve structured data as JSON. They are functionally identical — use whichever name better communicates your intent. On read, the JSON string is decoded into a PHP array. On write, the PHP array is JSON-encoded before storage.

**When to use it:** Use these when a column holds serialised structured data — settings, tags, configuration flags, metadata — that you want to work with as a native PHP array without manual `json_encode` / `json_decode` calls.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToArray;
use Phaseolies\Database\Entity\Casts\Attributes\ToJson;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Post extends Model
{
    // Use ToArray when the column represents a structured map
    #[ToArray]
    protected string $options;

    // Use ToJson when the column represents a list/tags
    #[ToJson]
    protected string $tags;

    // Equivalent using Transform
    #[Transform(Type::Array)]
    protected string $settings;
}
```

See the read and write example of array and json casts:
```php
$post = Post::find(1);

// Reading

// DB column: '{"theme":"dark","sidebar":true,"font_size":14}'
$post->options;                  // ['theme' => 'dark', 'sidebar' => true, 'font_size' => 14]
$post->options['theme'];         // "dark"
$post->options['sidebar'];       // true
$post->options['font_size'];     // 14

// DB column: '["php","doppar","orm","testing"]'
$post->tags;       // ['php', 'doppar', 'orm', 'testing']
$post->tags[0];    // "php"
count($post->tags); // 4
in_array('doppar', $post->tags); // true

// Writing

// PHP array → JSON-encoded automatically on write
$post->options = ['theme' => 'light', 'sidebar' => false, 'font_size' => 16];
// stored as: '{"theme":"light","sidebar":false,"font_size":16}'

$post->tags = ['doppar', 'casting', 'attributes'];
// stored as: '["doppar","casting","attributes"]'

// Nested structures

// DB: '{"address":{"street":"Main St","city":"Springfield","zip":"01234"},"roles":["admin","editor"]}'
$user->profile['address']['city']; // "Springfield"
$user->profile['address']['zip'];  // "01234"
$user->profile['roles'];           // ['admin', 'editor']
$user->profile['roles'][0];        // "admin"
```

> If the database column contains an invalid or empty JSON string, the cast returns an empty array `[]` rather than throwing an exception.

## Object Cast

`#[ToObject]` works identically to `#[ToArray]` but returns a `stdClass` object instead of an associative array, giving you property-style (`->`) access rather than array-style (`[]`) access.

**When to use it:** Use `#[ToObject]` when the JSON structure closely mirrors an object with named properties and you prefer `->` notation for readability. The practical difference between `#[ToArray]` and `#[ToObject]` is purely syntactic.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToObject;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Post extends Model
{
    #[ToObject]
    protected string $config;

    // Equivalent using Transform
    #[Transform(Type::Object)]
    protected string $settings;
}
```

> If the database column contains an invalid JSON string, the cast returns an empty `stdClass` object rather than throwing an exception.

## Collection Cast

`#[ToCollection]` deserialises a JSON string into a Doppar `Collection` instance on read, and serialises it back to JSON on write. This gives you Doppar's full Collection API — `map`, `filter`, `first`, `pluck`, `each`, `count`, and more — directly on the model property.

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToCollection;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Product extends Model
{
    #[ToCollection]
    protected string $images;

    #[ToCollection]
    protected string $tags;

    // Equivalent using Transform
    #[Transform(Type::Collection)]
    protected string $reviews;
}
```

## Date & Time Casts

Doppar integrates with [Carbon](https://carbon.nesbot.com/) for all date and time casting. Every date/time cast returns a `Carbon` instance on read, giving you the full Carbon API — formatting, timezone conversion, diffing, human-readable output, comparison, arithmetic, and more.

### datetime

`#[ToDateTime]` parses any date-time string from the database into a `Carbon` instance. On write, a `Carbon` or `DateTimeInterface` value is formatted as `Y-m-d H:i:s` before storage. Plain strings are passed through unchanged.

**When to use it:** Use for any column that stores a full date and time, such as `published_at`, `updated_at`, `shipped_at`, or `scheduled_for`.

```php
use Carbon\Carbon;
use Phaseolies\Database\Entity\Casts\Attributes\ToDateTime;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Post extends Model
{
    #[ToDateTime]
    protected string $published_at;

    #[ToDateTime]
    protected string $updated_at;

    // Equivalent using Transform
    #[Transform(Type::DateTime)]
    protected string $scheduled_at;
}
```

See the read and write example of datetime cast:
```php
$post = Post::find(1);

// Reading

// DB column: '2025-01-15 09:30:00'
$post->published_at;                              // Carbon instance
$post->published_at->format('d M Y');             // "15 Jan 2025"
$post->published_at->format('l, F j Y');          // "Wednesday, January 15 2025"
$post->published_at->format('H:i');               // "09:30"
$post->published_at->toDateString();              // "2025-01-15"
$post->published_at->toTimeString();              // "09:30:00"
$post->published_at->diffForHumans();             // "3 months ago"
$post->published_at->isToday();                   // false
$post->published_at->isPast();                    // true
$post->published_at->isFuture();                  // false
$post->published_at->addDays(7)->format('Y-m-d'); // "2025-01-22"
$post->published_at->startOfDay()->toDateTimeString(); // "2025-01-15 00:00:00"

// Writing

// Carbon → formatted as 'Y-m-d H:i:s' on write
$post->published_at = Carbon::now();
// stored as: '2025-01-15 09:30:00'

$post->published_at = Carbon::parse('2025-06-01 12:00:00');
// stored as: '2025-06-01 12:00:00'

// String → passed through unchanged
$post->published_at = '2025-08-20 00:00:00';
// stored as: '2025-08-20 00:00:00'
```

### date

`#[ToDate]` works like `#[ToDateTime]` but always strips the time component. The resulting Carbon instance represents midnight on the given date.

**When to use it:** Use for columns that represent a calendar date with no meaningful time, such as `birthday`, `hire_date`, `expiry_date`, or `order_date`. Storing and reading time information for these columns is irrelevant and can cause confusion.

```php
use Carbon\Carbon;
use Phaseolies\Database\Entity\Casts\Attributes\ToDate;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class User extends Model
{
    #[ToDate]
    protected string $birthday;

    #[ToDate]
    protected string $subscription_expires_on;

    // Equivalent using Transform
    #[Transform(Type::Date)]
    protected string $joined_on;
}
```

See the read and write example of date cast:
```php
$user = User::find(1);

// Reading

// DB column: '1990-03-25'
$user->birthday;                            // Carbon instance (midnight)
$user->birthday->format('d F Y');           // "25 March 1990"
$user->birthday->format('Y');               // "1990"
$user->birthday->age;                       // 35  (Carbon property)
$user->birthday->diffInYears(Carbon::now()); // 35

// DB column: '2025-12-31'
$user->subscription_expires_on->isPast();    // false (future date)
$user->subscription_expires_on->isFuture();  // true
$user->subscription_expires_on->diffInDays(); // days until expiry

// Writing

$user->birthday = Carbon::parse('1995-07-20');
// stored as: '1995-07-20 00:00:00'

$user->subscription_expires_on = Carbon::now()->addYear();
// stored as: '2026-04-13 00:00:00'
```

### timestamp

`#[ToTimestamp]` is for Unix timestamp columns — columns that store seconds since the Unix epoch as an integer.

- **On read** — the raw integer (or string representation of an integer) is wrapped in a `Carbon` instance, giving you the full date/time API.
- **On write** — a `DateTimeInterface` value is converted to its Unix integer via `->getTimestamp()` before storage. Plain integers are passed through unchanged.

**When to use it:** Use when your database schema explicitly stores Unix timestamps as integers (e.g., `last_login_at INT`, `expires_at INT`). This is common in legacy schemas, high-performance tables, or when integrating with external systems that exchange timestamps as integers.

```php
use Carbon\Carbon;
use Phaseolies\Database\Entity\Casts\Attributes\ToTimestamp;
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use Phaseolies\Database\Entity\Casts\Type;

class Session extends Model
{
    #[ToTimestamp]
    protected string $last_activity_at;

    #[ToTimestamp]
    protected string $expires_at;

    // Equivalent using Transform
    #[Transform(Type::Timestamp)]
    protected string $created_ts;
}
```

### Parameterised datetime Format

Use `#[Transform('datetime:FORMAT')]` when your database stores dates in a non-standard format (e.g., `d/m/Y`) and you need to write back in that same format.

**Important:** The format string only controls the **write direction** — how a `DateTimeInterface` value is formatted before storage. On **read**, you always get a full `Carbon` instance regardless of the format. This means `->toDateString()`, `->format(...)`, `->diffForHumans()`, and all other Carbon methods are always available.

```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;

class Invoice extends Model
{
    // DB stores '15/01/2025' — write must use same format
    #[Transform('datetime:d/m/Y')]
    protected string $issued_on;

    // DB stores '2025-01-15' — standard ISO format
    #[Transform('datetime:Y-m-d')]
    protected string $due_on;

    // DB stores '15 Jan 2025'
    #[Transform('datetime:d M Y')]
    protected string $reminded_at;
}
```

## Decimal Cast

`#[Transform('decimal:N')]` rounds a numeric value to exactly `N` decimal places on both read and write, using PHP's `round()` function internally.

**When to use it:** Use for any monetary, financial, or scientific value where exact decimal precision is required and floating-point arithmetic cannot be trusted. Use `decimal:2` for prices and currency amounts, `decimal:4` for tax rates and percentage values, `decimal:6` for scientific measurements.

```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;

class Product extends Model
{
    #[Transform('decimal:2')]
    protected string $price;

    #[Transform('decimal:2')]
    protected string $compare_price;

    #[Transform('decimal:4')]
    protected string $tax_rate;

    #[Transform('decimal:4')]
    protected string $discount_rate;
}
```

## Enum Cast

Doppar automatically maps database scalar values to PHP 8 enum cases using `#[Transform(MyEnum::class)]`. Both backed enums (`string` or `int`) and pure unit enums are supported.

### Backed String Enum

A backed string enum associates each case with a string value. On read, Doppar calls `MyEnum::from($rawValue)`. On write, it stores `$enum->value`.

```php
namespace App\Enums;

enum OrderStatus: string
{
    case Pending    = 'pending';
    case Processing = 'processing';
    case Shipped    = 'shipped';
    case Delivered  = 'delivered';
    case Cancelled  = 'cancelled';
}
```

The model that is using enum cast:
```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use App\Enums\OrderStatus;

class Order extends Model
{
    #[Transform(OrderStatus::class)]
    protected string $status;
}
```

Now the read and write operation of enum cast:
```php
$order = Order::find(1);

// Reading

// DB column: 'shipped'
$order->status;              // OrderStatus::Shipped  (enum case)
$order->status->value;       // "shipped"
$order->status->name;        // "Shipped"

// DB column: 'pending'
$order->status === OrderStatus::Pending;  // true

// Use in match
$label = match ($order->status) {
    OrderStatus::Pending    => 'Awaiting payment',
    OrderStatus::Processing => 'Being prepared',
    OrderStatus::Shipped    => 'On its way',
    OrderStatus::Delivered  => 'Arrived',
    OrderStatus::Cancelled  => 'Cancelled',
};

// Writing

$order->status = OrderStatus::Delivered;
// stored as: 'delivered'

$order->status = OrderStatus::Cancelled;
// stored as: 'cancelled'
```

### Backed Integer Enum

A backed integer enum works identically but maps to integer values.

```php
namespace App\Enums;

enum Priority: int
{
    case Low    = 1;
    case Medium = 2;
    case High   = 3;
    case Urgent = 4;
}
```

The model that is using integer enum cast:
```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use App\Enums\Priority;

class Task extends Model
{
    #[Transform(Priority::class)]
    protected string $priority;
}
```

### Pure Unit Enum

A pure (unit) enum has no backing scalar value. Doppar matches the stored string against case names on read and stores the case name as a string on write.

```php
namespace App\Enums;

enum UserRole
{
    case Admin;
    case Editor;
    case Viewer;
    case Guest;
}
```

The model that is using unit enum cast:
```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use App\Enums\UserRole;

class User extends Model
{
    #[Transform(UserRole::class)]
    protected string $role;
}
```

See the example of read and write operation with unit enum cast:
```php
$user = User::find(1);

// Reading

// DB column: 'Admin'
$user->role;        // UserRole::Admin  (enum case)
$user->role->name;  // "Admin"

// DB column: 'Viewer'
$user->role === UserRole::Viewer;  // true

// Use in conditions
if ($user->role === UserRole::Admin) {
    // show admin controls
}

// Writing

$user->role = UserRole::Editor;
// stored as: 'Editor'

$user->role = UserRole::Guest;
// stored as: 'Guest'
```

> If the stored value does not match any case name (or backed value), Doppar throws a `ValueError`. This surfaces data integrity issues early rather than silently returning null or the wrong case.

## Custom Cast Classes

For transformations that go beyond the built-in types, implement your own cast class using `CastableInterface`. The interface has exactly two methods: `get()` for converting from database to PHP, and `set()` for converting from PHP to database.

### Writing a Custom Cast

```php
namespace Phaseolies\Database\Entity\Casts\Contracts;

interface CastableInterface
{
    /**
     * Convert the raw database value to its PHP representation.
     */
    public function get(mixed $value): mixed;

    /**
     * Convert the PHP value to its database-ready format.
     */
    public function set(mixed $value): mixed;
}
```

Key rules for custom casts:

1. Both `get()` and `set()` receive `null` when the column is null — always handle this case.
2. The `set()` method should return a scalar value (string, int, float) or null — not a PHP object.
3. Register your cast by passing the class name to `#[Transform(YourCast::class)]`.
4. If your cast class takes constructor arguments, Doppar instantiates it via `new YourCast()` — so constructor parameters must have defaults if you share the class across properties.

### Example: Money Cast

Stores monetary values as integer cents in the database (`1999`) and returns a human-readable formatted string in PHP (`"USD 19.99"`).

```php
namespace App\Casts;

use Phaseolies\Database\Entity\Casts\Contracts\CastableInterface;

class MoneyCast implements CastableInterface
{
    public function __construct(
        private string $currency = 'USD'
    ) {}

    /**
     * DB: 1999  →  PHP: "USD 19.99"
     */
    public function get(mixed $value): mixed
    {
        if ($value === null) return null;

        return sprintf('%s %.2f', $this->currency, $value / 100);
    }

    /**
     * PHP: 19.99  →  DB: 1999
     */
    public function set(mixed $value): mixed
    {
        if ($value === null) return null;

        return (int) round((float) $value * 100);
    }
}
```

Usage example of `MoneyCast`
```php
use Phaseolies\Database\Entity\Casts\Attributes\Transform;
use App\Casts\MoneyCast;

class Product extends Model
{
    #[Transform(MoneyCast::class)]
    protected string $price;

    #[Transform(MoneyCast::class)]
    protected string $compare_price;

    #[Transform(MoneyCast::class)]
    protected string $discount;
}
```

Now see the example of read and write:
```php
$product = Product::find(1);

// DB: 1999
$product->price;         // "USD 19.99"
$product->compare_price; // "USD 24.99"
$product->discount;      // "USD 5.00"

// Writing
$product->price = 29.99;
// stored as: 2999

$product->price = 0.99;
// stored as: 99
```

## Cast Behaviour on Read & Write

Understanding exactly when casts are applied prevents unexpected behaviour.

### On Read

The cast handler's `get()` method is called every time you access the attribute via `$model->attribute`, `$model['attribute']`, or during serialisation. The raw value is always preserved in `$attributes` — the cast result is never stored back. This means the cast runs fresh on every access.

```php
$order = Order::find(1);

// What's in $attributes:
// ['is_paid' => '1', 'meta' => '{"key":"val"}', 'total' => '99.99']

$order->is_paid; // bool(true)    ← BooleanCast::get('1')
$order->meta;    // ['key'=>'val'] ← ArrayCast::get('{"key":"val"}')
$order->total;   // float(99.99)  ← DecimalCast::get('99.99')

// Re-reading always re-casts from the same raw value
$order->is_paid; // bool(true)  (re-cast, not cached)
```

### On Write

The cast handler's `set()` method is called before the value is stored in `$attributes`. For **complex types** (array, json, object, collection, datetime, date, timestamp, decimal, enum, and custom cast classes), the set handler serialises the value to its database format. For **primitive types** (int, float, bool, string), no set-cast is applied — PDO handles the conversion natively.

```php
$order = new Order();

// Complex types — set-cast applied
$order->meta       = ['key' => 'value'];   // stored as '{"key":"value"}'
$order->status     = OrderStatus::Paid;    // stored as 'paid'
$order->total      = 99.999;              // stored as 100.0  (decimal:2)
$order->shipped_at = Carbon::now();       // stored as '2025-01-15 09:30:00'

// Primitive types — PDO handles, no set-cast
$order->is_paid = true;   // stored as true (PDO → 1 in MySQL)
$order->views   = 100;    // stored as 100

// Read back — get-cast applied
$order->meta['key'];    // "value"
$order->status->value;  // "paid"
$order->total;          // float(100.0)
$order->is_paid;        // bool(true)
```

### During Database Hydration

When Doppar loads a model from a query result, values from the database row are written into `$attributes` via `setAttribute()`. This triggers the set-cast. All built-in set handlers are designed to pass through raw database strings unchanged (no double-encoding). The get-cast is then applied on first read.

```php
// DB row: ['meta' => '{"key":"value"}', 'is_paid' => '1']

// Hydration:
// ArrayCast::set('{"key":"value"}') → '{"key":"value"}' (no change)
// Boolean: no set-cast (primitive — PDO handles it)

// First read:
$post->meta;    // ['key' => 'value']  ← ArrayCast::get()
$post->is_paid; // true                ← BooleanCast::get()
```

## Null Handling

Every built-in cast handler returns `null` immediately when the incoming value is `null`, without attempting any conversion. You do not need to guard against null values when accessing cast properties — a null column will always return null in PHP, regardless of the cast type.

```php
$post = Post::find(1);

// DB: published_at IS NULL
$post->published_at; // null  — not a Carbon instance, not an exception

// DB: meta IS NULL
$post->meta; // null  — not [], not an exception

// DB: status IS NULL
$post->status; // null  — not an enum case, not an exception

// Safe to check before using
if ($post->published_at !== null) {
    echo $post->published_at->format('d M Y');
}
```

Custom cast classes should follow the same convention — check for `null` at the top of both `get()` and `set()` and return `null` early.

## Serialization

Casts are automatically applied when converting a model to an array or JSON. `toArray()` and `toJson()` always return properly typed, cast values — never raw database strings.

```php
$order = Order::find(1);

$order->toArray();

[
  'id'         => 1,
  'is_paid'    => true,                    // not "1"
  'total'      => 99.99,                   // not "99.990000"
  'meta'       => ['theme' => 'dark'],     // not '{"theme":"dark"}'
  'status'     => 'paid',                  // enum->value automatically
  'shipped_at' => Carbon instance,
  'item_count' => 3,                       // not "3"
]

$order->toJson();

{
    "id":1,
    "is_paid":true,
    "total":99.99,
    "meta":{
        "theme":"dark"
    },
    "status":"paid",
    "item_count":3
}
```

**Hidden attributes are respected:**

Properties in `$unexposable` are excluded from both `toArray()` and `toJson()`, regardless of whether they have a cast:

```php
use Phaseolies\Database\Entity\Casts\Attributes\ToBoolean;
use Phaseolies\Database\Entity\Casts\Attributes\ToArray;
use Phaseolies\Database\Entity\Casts\Attributes\ToString;

class User extends Model
{
    protected $unexposable = ['password', 'remember_token', 'national_id'];

    #[ToBoolean]
    protected string $is_active;

    #[ToArray]
    protected string $settings;

    #[ToString]
    protected string $national_id;  // excluded from output
}

User::find(1)->toArray();
// password, remember_token, national_id are absent
// is_active → bool, settings → array
```

**Relationships are included with casts applied:**

When a model has loaded relationships, `toArray()` recursively applies casts through the full object graph:

```php
$user = User::find(1);
$user->load('orders');

$user->toArray();

{
  "id": 1,
  "is_active": true,
  "orders": [
    {
      "id": 10,
      "is_paid": true,         // cast on nested model
      "total": 99.99,          // cast on nested model
      "status": "delivered",   // cast on nested model
      ...
    }
  ]
}
```

## Inspecting Casts at Runtime

Doppar provides several methods on every model to inspect the cast configuration at runtime — useful for debugging, building generic serialisers, or writing test assertions.

`hasCast` true if the attribute has a #[Transform] / To* annotation
```php
$order = new Order();

$order->hasCast('is_paid');   // true
$order->hasCast('status');    // true
$order->hasCast('title');     // false  (no cast declared)
$order->hasCast('id');        // false
```

`getCastType` Returns the resolved cast type string, or null if no cast is declared.
```php
$order->getCastType('is_paid');   // "boolean"
$order->getCastType('total');     // "decimal:2"
$order->getCastType('status');    // "App\Enums\OrderStatus"
$order->getCastType('meta');      // "array"
$order->getCastType('shipped_at'); // "datetime"
$order->getCastType('title');     // null
```

`getCasts` Returns all cast definitions as an associative array.
```php
$order->getCasts();

[
  'is_paid'    => 'boolean',
  'total'      => 'decimal:2',
  'status'     => 'App\Enums\OrderStatus',
  'meta'       => 'array',
  'shipped_at' => 'datetime',
  'item_count' => 'integer',
]
```

`getCastAttributes` Returns all model attributes with casts already applied.
```php
$order->getCastAttributes();

[
  'id'         => 1,
  'is_paid'    => true,
  'total'      => 99.99,
  'status'     => OrderStatus::Delivered,
  'meta'       => ['theme' => 'dark'],
  'shipped_at' => Carbon instance,
  'item_count' => 3,
]
```

## Cache Management

Cast metadata is scanned from property attributes via reflection the first time a cast is accessed on a given class. The result is stored in a static per-class cache and shared across all instances for the lifetime of the process.

```
First access:
  $order->is_paid
    → ensureCastsCached()
    → cache miss: ReflectionClass(Order) scans all properties
    → finds #[ToBoolean] on $is_paid, #[Transform('decimal:2')] on $total, etc.
    → stores ['is_paid' => 'boolean', 'total' => 'decimal:2', ...] in static cache
    → BooleanCast::get('1') → true

Subsequent accesses (same or different instance):
  $order->is_paid
    → ensureCastsCached()
    → cache hit: no reflection
    → BooleanCast::get('1') → true
```

In test environments or when classes are dynamically reloaded, you can flush the cache manually:

Flush cache for one specific model class
```php
Order::resetCastCache(Order::class);
```

Flush cache for all model classes at once
```php
Order::resetCastCache();
```

The `CastManager` also maintains a separate resolved-handler cache (one handler instance per cast type string). You can flush it independently in tests:

Flush the resolved handler instance cache
```php
use Phaseolies\Database\Entity\Casts\CastManager;

CastManager::flush();
```
