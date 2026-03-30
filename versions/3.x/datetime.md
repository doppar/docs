---
title: Date and Time Handling in Doppar
description: Doppar Date and Time Handling page
meta:
  - name: keywords
    content: date and time handling in doppar
---

## Date and Time Handling in Doppar
### Introduction
Doppar provides a robust and flexible system for working with dates and times, thanks to its integration with the Carbon library. However, understanding how timezones are managed—especially when using functions like `now()` or `Carbon::now()`—can be important when building applications that rely on accurate time data.

By default, Doppar uses UTC as the base timezone for datetime functions such as `now()` and `Carbon::now()`. 

<div class="doppar-alert doppar-alert-danger">
This might be confusing at first glance if you return timezone as reponse because any calls to these methods will return the current time in UTC.
</div>

But don't worry—Doppar is designed to internally handle timezone conversions using the timezone set in your `config/app.php` configuration file

```php
'timezone' => env('APP_TIMEZONE', 'UTC'),
```
Whenever you use dates for display, formatting, or storing them in databases, Doppar ensures they are correctly converted to or from the application's configured timezone.

## Working with Carbon
Carbon is a popular PHP library for date and time manipulation, and it's seamlessly integrated into the Doppar. Whether you're retrieving the current time or manipulating dates, Carbon provides a consistent and elegant API.

When working within Doppar, you have multiple ways to create a Carbon instance representing the current time. Here are three equivalent approaches: Let's see the example. To know more about [Carbon](https://carbon.nesbot.com/), see its oficial docs.
```php
use Carbon\Carbon;

Carbon::now();
now();
app('timezone')->now();
```
All of these methods will return the same result — a Carbon instance with the current date and time.