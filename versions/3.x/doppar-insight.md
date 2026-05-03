---
title: Insight
description: Doppar Insight page
meta:
  - name: keywords
    content: doppar insight
---

## Insight

### Introduction
Doppar Insight is a request profiler for debugging, performance analysis, and traffic inspection. It gives you a clean in-browser toolbar for the current request, and it also keeps a recent request history so you can compare status codes, response times, errors, and route activity across multiple requests.

Insight is useful when you want to inspect SQL queries, cache usage, authentication state, request and response payloads, session data, logs, or performance timing without leaving the page you are working on. It is designed for day-to-day development, but it is especially helpful when you need to trace slow endpoints or understand why a route is returning `4xx` or `5xx` responses.

## Features
Insight ships with both per-request inspection and cross-request traffic visibility.

- Current request inspection for `HTTP`, `Database`, `Cache`, `Auth`, `Request`, `Response`, `Performance`, `Session`, `Logs`, and `JSON API`
- Cross-request `Overview` dashboard with traffic, latency, and error activity
- Cross-request `History` dashboard with time filters, request charts, route search, and route-level status summaries
- Stored request history with method, path, status code, duration, and exception context
- Better visibility into `4xx` and `5xx` traffic when paired with `BeforeExceptionHandler`

## Screenshots
Here is a quick visual look at the main Insight views. These screenshots are loaded from the `doppar/insight` repository, so they will stay in sync with the package after your merge.

### Overview Dashboard
The `Overview` screen combines recent request activity, latency, and error trends into one cross-request dashboard.

![Insight overview dashboard](/insight-profiler.png)

## Installation
You may install Doppar Insight via Composer.

```bash
composer require doppar/insight
```

### Register Provider
Next, register the Insight service provider so Doppar can bootstrap the profiler. Open your `config/app.php` file and add the provider to the `providers` array.

```php
'providers' => [
    // Other service providers...
    \Doppar\Insight\ProfilerServiceProvider::class,
],
```

### Publish Configuration
Now publish the Insight configuration so you can adjust retention and enable or disable the profiler when needed.

```bash
php pool vendor:publish --provider="Doppar\Insight\ProfilerServiceProvider"
```

This command publishes `config/insight.php`. From there you can control whether Insight is enabled and how long request snapshots should be kept on disk.

## Production Usage
Insight is best suited for development, staging, and short-lived debugging sessions. Because it can collect request details, exception context, session data, logs, queries, and cross-request history, it should not remain enabled for normal public production traffic.

If you need Insight on a live server, use it as a temporary internal debugging tool only:

- keep `enabled` set to `true` by default
- turn it on only for a short troubleshooting window
- allow access only from trusted internal or VPN IP addresses
- avoid exposing the toolbar or history endpoints to public users

Here is a safer production-style example for `config/insight.php`.

```php
return [
    'allow_ips' => ['127.0.0.1', '::1'],
];
```

## Better Error Tracing
After installation, add an application-level exception hook so Insight can store uncaught exceptions in its request history. This gives you a much better error tracing experience for `4xx` and `5xx` responses, especially when the request does not finish through the normal success path.

Update `app/Http/Exceptions/BeforeExceptionHandler.php` with the following code.

```php
/**
 * Handle logic to be executed before the application processes an exception
 *
 * @param Throwable $throwable
 * @return void
 */
public function handle(Throwable $throwable): void
{
    app(ErrorHistoryRecorder::class)->record($throwable);
}
```

The important line is:

```php
app(ErrorHistoryRecorder::class)->record($throwable);
```

That recorder stores the exception as part of Insight history so the `Overview` and `History` dashboards can show failed requests, exception spikes, and route-level error counts.

## Using the Toolbar
Insight injects a toolbar into HTML responses so you can inspect the request directly from the page. Click the toolbar button to open the panel and move between the available sections.

The sidebar is split into two kinds of views:

- `Overview` and `History` are cross-request views. They combine the current request with recently captured traffic.
- `HTTP`, `Database`, `Cache`, `Auth`, `Request`, `Response`, `Performance`, `Session`, `Logs`, and `JSON API` focus on the current request only.

For JSON or API responses, the toolbar is not injected into the response body, but the request can still be stored in Insight history.

## Overview Dashboard
The `Overview` view gives you a high-level activity snapshot across recent requests. Use it when you want to quickly understand whether the application is healthy before drilling into one request.

It includes request counts, status-code totals, latency trends, and error activity so you can see if failures or slowdowns are isolated or part of a wider pattern.

## History Dashboard
The `History` view is designed for comparing recent traffic over time. It is useful when you want to inspect how a route behaves across many requests instead of only checking the page that is currently open.

You can use it to:

- filter traffic by time range such as `1H`, `24H`, `7D`, `14D`, and `30D`
- review request-volume, duration, and error-activity graphs
- search routes by method or path
- filter the route table to show all traffic, only errors, or slow routes
- inspect route-level request counts, `3xx`, `4xx`, `5xx`, average duration, and maximum duration

## Data Storage
Insight stores recent request snapshots as JSON files under `storage/framework/profiler`. The `retention_days` value in `config/insight.php` controls how long those snapshots are kept before old data is removed automatically.
