---
title: Insight
description: Doppar Insight page
meta:
  - name: keywords
    content: doppar insight
---

## Insight
### Introduction
Doppar Insight Profiler is an advanced debugging and performance monitoring tool designed to give developers deep visibility into their application's inner workings. It provides detailed insights into every request, including HTTP methods, routes, response times, memory usage, and framework versions — all within a clean, intuitive dashboard.

With Doppar Insight, developers can easily analyze performance metrics, trace database queries, inspect cache operations, monitor authentication flows, and review request-response lifecycles in real time. This makes it easier to identify performance bottlenecks, optimize resource usage, and ensure smooth application execution.

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

