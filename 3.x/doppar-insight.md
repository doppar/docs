---
title: Insight
description: Doppar Insight page
meta:
  - name: keywords
    content: doppar insight
---

## Insight
### Introduction
Doppar Insight Profiler created by [rrr63](https://github.com/rrr63) is an advanced debugging and performance monitoring tool designed to give developers deep visibility into their application's inner workings. It provides detailed insights into every request, including HTTP methods, routes, response times, memory usage, and framework versions — all within a clean, intuitive dashboard.

With Doppar Insight, developers can easily analyze performance metrics, trace database queries, inspect cache operations, monitor authentication flows, and review request-response lifecycles in real time. This makes it easier to identify performance bottlenecks, optimize resource usage, and ensure smooth application execution.

![Doppar Insight Overview](https://private-user-images.githubusercontent.com/31318802/524459747-73d556ad-34fd-433a-968f-c3a0a198bb1d.png?jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmF3LmdpdGh1YnVzZXJjb250ZW50LmNvbSIsImtleSI6ImtleTUiLCJleHAiOjE3NjU0MjgwNTMsIm5iZiI6MTc2NTQyNzc1MywicGF0aCI6Ii8zMTMxODgwMi81MjQ0NTk3NDctNzNkNTU2YWQtMzRmZC00MzNhLTk2OGYtYzNhMGExOThiYjFkLnBuZz9YLUFtei1BbGdvcml0aG09QVdTNC1ITUFDLVNIQTI1NiZYLUFtei1DcmVkZW50aWFsPUFLSUFWQ09EWUxTQTUzUFFLNFpBJTJGMjAyNTEyMTElMkZ1cy1lYXN0LTElMkZzMyUyRmF3czRfcmVxdWVzdCZYLUFtei1EYXRlPTIwMjUxMjExVDA0MzU1M1omWC1BbXotRXhwaXJlcz0zMDAmWC1BbXotU2lnbmF0dXJlPWU4OTRiYjNjZGRjMTIzOWUyNGQwZmUwNDY1NTQ1NzZhNDg0ODI0MThiM2M5Mzg1NTljYmI3MWI2M2U4NzQ3OGMmWC1BbXotU2lnbmVkSGVhZGVycz1ob3N0In0.NADYALv8e_L6Dr_Y0iab5kgMZrkUVOkVWE4L5ZmBG-w)

Whether you’re diagnosing a slow endpoint or validating backend logic, Doppar Insight Profiler offers the clarity and control you need to debug efficiently and ship with confidence.

## Installation
To get started with Doppar Insight, use the composer package manager to add the package to your doppar project's dependencies:
```bash
composer require doppar/insight
```

## Register Provider
Next, register the Doppar Insight service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `ProfilerServiceProvider` to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Insight\ProfilerServiceProvider::class,
],
```

## Publish Configuration
Now we need to publish the configuration files by running this pool command
```bash
php pool vendor:publish --provider="Doppar\Insight\ProfilerServiceProvider"
```

This command will publish the `config/insight.php` file to your application’s configuration directory. From there, you can modify the settings to suit your needs — for example, adjusting the `retention_days` value or enabling any disabled options.

