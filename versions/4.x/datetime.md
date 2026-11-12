---
title: Date and Time Handling in Doppar
description: Doppar Date and Time Handling page
meta:
  - name: keywords
    content: date and time handling in doppar
---

## Datetime

Doppar provides a robust and flexible system for working with dates and times through its Carbon integration. Timezone behavior is explicit: the application timezone is configured through `config/app.php`, while individual Carbon instances may use their own timezone.

Doppar uses `UTC` by default. If your application returns a timezone or formatted timestamp, remember that the configured application timezone is applied when the application boots.

Set the application timezone in `config/app.php`:

```php
'timezone' => env('APP_TIMEZONE', 'UTC'),
```

Doppar applies this timezone to PHP's default timezone and to the framework timezone service. Database date and time values should still be stored and queried consistently with your database driver's timezone configuration.

## Working with Carbon

Carbon is a popular PHP library for date and time manipulation, and it is integrated into Doppar. To learn more about Carbon, see the [official Carbon documentation](https://carbon.nesbot.com/).

The `now()` helper accepts an optional timezone. With no argument, it uses the application/PHP default timezone:

```php
now();
```

Pass a timezone when a value must be created in a specific timezone:

```php
now('Asia/Dhaka');
now(new DateTimeZone('UTC'));
```

You can also use Carbon directly. Uses PHP's configured default timezone.
```php
use Carbon\Carbon;

Carbon::now();
```

Explicitly creates a UTC instance.
```php
Carbon::now('UTC');
```

Use the framework timezone service when you want the configured application timezone explicitly:

Uses `config/app.php`'s timezone.
```php
app('timezone')->now(); 
```

`Carbon::now()` and `now()` without an argument normally represent the same instant and timezone because Doppar configures PHP's default timezone during application startup. An explicit timezone argument overrides that default.

## Date-time precision

Doppar preserves microseconds when a `DateTimeInterface` value is formatted for date-time query operations or pivot updates. Values without microseconds retain the familiar `Y-m-d H:i:s` format; values with microseconds are written as `Y-m-d H:i:s.u`.

```php
$date = DateTimeImmutable::createFromFormat(
    'Y-m-d H:i:s.u',
    '2024-03-20 15:45:00.123456'
);
```

Date-time query operations preserve the `.123456` precision.
```php
$builder->whereDateTimeBetween('created_at', $date, $date);
```

The default `datetime` cast keeps the existing `Y-m-d H:i:s` output for second-precision values and appends `.u` when the value contains non-zero microseconds. Custom `datetime:<format>` casts use the supplied format exactly when serializing a `DateTimeInterface` for storage. On read, Doppar parses the stored value into a Carbon instance.

When passing date-only strings to `whereDateTimeBetween()`, Doppar expands the first value to `00:00:00` and the second value to `23:59:59`:

```php
$builder->whereDateTimeBetween('created_at', '2024-03-20', '2024-03-21');
```
