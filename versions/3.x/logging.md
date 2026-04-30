---
title: Log
description: Doppar Log page
meta:
  - name: keywords
    content: logging, log, monolog, slack, daily log, helper, facade, doppar
---

## Logging

### Introduction

Logging is one of the first tools you reach for when you need to
understand what happened inside a request, background command, or
integration flow. Doppar's logging layer is built on top of
**Monolog**, giving you a familiar structured logger with support for
multiple channels, severity levels, and context data.

You can write logs through the `Log` facade or the global helper
functions. Both styles resolve the same underlying logger service.

## Configuration

Logging is configured in `config/log.php`. This file defines the default
channel, the available channels, and the file or remote destination each
channel should use.

Set the default channel in your `.env` file:

```bash
LOG_CHANNEL=daily
LOG_LEVEL=debug
SLACK_WEBHOOK_URL=https://hooks.slack.com/services/your/webhook/url
```

The default Doppar configuration supports these channels:

| Channel  | Purpose |
|----------|---------|
| `single` | Write all logs to one file at `storage/logs/doppar.log` |
| `daily`  | Rotate logs daily while keeping the same base log path |
| `stack`  | Send the same log entry to multiple configured channels |
| `slack`  | Forward logs to a Slack webhook |

## Writing Log Messages

Doppar exposes two equally valid logging styles:
the `Log` facade and the global helper functions. Use whichever style
fits the surrounding code better.

### Using the `Log` Facade

Use the facade when you want an explicit, class-based logging API in
controllers, services, jobs, or commands.

```php
use Phaseolies\Support\Facades\Log;

Log::info('User signed in.');
Log::warning('Payment retry scheduled.');
Log::error('Webhook processing failed.');
```

The logger supports the standard eight logging levels:
`emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info`,
and `debug`.

```php
use Phaseolies\Support\Facades\Log;

Log::debug('Debugging payload received.');
Log::notice('Profile was updated.');
Log::critical('Primary database is unavailable.');
Log::emergency('System is no longer responding.');
```

### Using the Global Log Helpers

Use the helper functions when you want concise logging without importing
the facade. These helpers use the current default channel unless you log
through `Log::channel(...)`.

```php
info('User signed in.');
warning('Payment retry scheduled.');
error('Webhook processing failed.');
debug('Debugging payload received.');
critical('Primary database is unavailable.');
```

## Logging Context Data

Sometimes the message alone is not enough. Doppar lets you pass a second
argument as structured context so request details, identifiers, or other
diagnostic metadata travel with the log entry.

### Context with the `Log` Facade

Use the second argument to attach structured data to the log entry.

```php
use Phaseolies\Support\Facades\Log;

Log::info('Application is terminating.', [
    'request' => $request->all(),
    'response_status' => $response?->getStatusCode(),
    'exception' => $exception?->getMessage(),
]);
```

### Context with Helper Functions

The global helpers accept the same second argument, so helper and facade
logging stay consistent.

```php
info('User login failed.', [
    'email' => $request->input('email'),
    'ip' => $request->ip(),
]);
```

When context data is provided, Doppar includes it in the formatted log
output for the supported handlers.

## Writing to a Specific Channel

Sometimes a message should go somewhere other than the default channel.
Use `channel()` on the `Log` facade when you want to target a specific
configured destination.

### Channel-Based Logging

Each channel must exist in `config/log.php`.

```php
use Phaseolies\Support\Facades\Log;

Log::channel('stack')->info('Application booted.');
Log::channel('daily')->warning('Background import is slow.');
Log::channel('single')->notice('User profile updated.');
Log::channel('slack')->alert('Third-party API is down.');
```

This is useful when your application needs different delivery behavior
for local diagnostics, rotating files, and team alerts.

## Choosing a Channel

Different channels solve different operational problems. A single file
is simple, daily rotation is safer for long-running production systems,
stack is useful when you want the same log entry in multiple places, and
Slack is best reserved for high-signal operational alerts.

### Example Channel Strategy

One practical production setup is to keep `daily` as the default
application log while sending only urgent failures to `slack`.

```bash
LOG_CHANNEL=daily
LOG_LEVEL=debug
```

Then use explicit Slack logging only where the event deserves immediate
team visibility.

```php
use Phaseolies\Support\Facades\Log;

Log::channel('slack')->critical('Checkout service is unavailable.', [
    'service' => 'checkout',
    'region' => 'ap-south-1',
]);
```