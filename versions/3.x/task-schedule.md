---
title: Task Schedule
description: Doppar Task Schedule page
meta:
  - name: keywords
    content: Task Schedule
---

## Task Schedule
### Introduction
In Doppar, scheduled tasks allow you to automate repetitive operations such as sending emails, processing invoices, or cleaning up stale data — all defined directly inside your application. Instead of relying on scattered shell scripts or manually crafted crontabs, you register every schedule in a single `App\Schedule\Schedule` class. This keeps your automation logic version-controlled, readable, and easy to maintain.

Doppar's scheduler supports two modes of execution: a standard minute-based mode compatible with any system cron, and a built-in daemon mode capable of sub-minute, per-second precision. Both modes share the same expressive API.

## Why Choose Doppar Scheduler
### Real-Time Precision
Doppar runs tasks with second-level accuracy, enabling real-time automation, monitoring, and event-driven pipelines. Perfect for applications that cannot wait for the next minute.

### Smart Dual-Mode Engine
Run tasks using standard cron or switch to Doppar's built-in daemon for continuous execution. No external tools. No systemd. No Supervisor. Just plug-and-run.

### True Background Job Execution
Launch tasks in the background with full process management:
- Per-task PID tracking
- Process metadata
- Configurable output destination per job
- Auto-cleanup after completion

This turns Doppar into a mini process manager.

### Crash-Resilient Daemon
If a task fails, Doppar's daemon continues running without dying. Signals like SIGTERM and SIGINT are handled gracefully, so shutdowns are safe and clean.

### Full Observability
Every background task records its PID, start time, command string, and output to a dedicated log by default. You can redirect that output to any file — or suppress it entirely with `withoutLog()`. The daemon writes its own heartbeat log, giving you real-time operational visibility.

Doppar Task Scheduling System — Feature Overview
| Feature                  | Doppar Scheduling System                              | Notes / Advantage                                         |
| ------------------------ | ----------------------------------------------------- | --------------------------------------------------------- |
| **Execution Precision**  | Second-based scheduling (every X seconds)             | Enables high-frequency automation and real-time jobs      |
|                          | Real-time daemon mode                                 | Runs tasks continuously                                   |
|                          | Minute, hourly, daily intervals                       | Full range of scheduling options                          |
| **Daemon Capabilities**  | Built-in daemon engine                                | No need for external tools like Supervisor or systemd     |
|                          | Graceful shutdown (SIGTERM, SIGINT)                   | Safe cleanup + predictable exit behavior                  |
|                          | Crash-resistant loop with auto-recovery               | Daemon never silently stops; auto stabilizes              |
|                          | Heartbeat logging                                     | Great for monitoring and uptime analytics                 |
| **Concurrency & Safety** | `noOverlap()` with per-command lock files             | Prevents double execution with strong guarantees          |
|                          | PID tracking for each task                            | Know exactly what is running at any moment                |
|                          | Atomic lock writing with retries                      | Eliminates race conditions in concurrent environments     |
| **Process Management**   | First-class background jobs (`inBackground()`)        | Fully managed async task execution                        |
|                          | Configurable output per job (`sendOutputTo()`)        | Route logs anywhere — or silence them with `withoutLog()` |
|                          | Process metadata (PID, timestamps, OS, command)       | Deep introspection for debugging and monitoring           |
|                          | Automatic cleanup after completion                    | Prevents deadlocks and stale process references           |
| **Scheduling Engine**    | Dual-mode scheduling (standard + daemon)              | Works with cron AND real-time loops                       |
|                          | High-speed due-checking loop                          | Millisecond-level responsiveness                          |
|                          | Safe fallback when daemon is not running              | Tasks still run instead of being missed                   |
| **Developer Experience** | Fluent and expressive scheduling API                  | Easy to read, easy to write                               |
|                          | Rich CLI output formatting                            | Clean, informative console messages                       |
|                          | Auto-detection of high-frequency tasks                | Smarter runtime behavior with minimal setup               |
| **Observability**        | JSON-formatted process info                           | Machine-readable — ideal for dashboards or log processors |
|                          | Separate logs for daemon and tasks                    | Better separation of concerns                             |
|                          | Built-in warnings and status hints                    | Helps diagnose issues instantly                           |
| **Platform Support**     | Full POSIX signal support                             | Behaves like a real system service                        |
|                          | Shell-based background process launching              | Compatible with Linux/macOS environments                  |


## Scheduling Pool Commands
You can register a Pool command using the `command` method, passing the command's name along with any arguments or options it requires. Refer to the [Pool Console documentation](http://doppar.com/versions/3.x/pool-console#command-arguments) for details on how arguments and options are defined in a command class. All commands are registered inside the `schedule()` method of your `App\Schedule\Schedule` class.
```php
<?php

namespace App\Schedule;

use Phaseolies\Console\Schedule\InteractsWithSchedule;

class Schedule
{
    use InteractsWithSchedule;

    /**
     * Register commands to be scheduled.
     *
     * @param Schedule $schedule
     * @return void
     */
    public function schedule(Schedule $schedule): void
    {
        $schedule->command("user:sync")->everyMinute();
    }
}
```
Using the `everyMinute()` method ensures that the `user:sync` command will be executed once every minute by the scheduler.

### Passing Arguments and Options
Commands that accept arguments or options can be scheduled by appending them directly to the command string, exactly as you would type them in the terminal.
```php
$schedule->command('report:generate monthly --format=pdf')
    ->dailyAt('08:00');
```
In this example, `monthly` is a positional argument and `--format=pdf` is a named option. Doppar correctly parses and passes each token as a separate argument to the command handler.

## Every-Second Scheduling
The `everySecond()` method schedules the command to run once every second, providing the highest possible execution frequency within Doppar's task scheduler. Second-based schedules require the cron daemon to be running (see [Managing the Cron Daemon](#managing-the-cron-daemon)).
```php
$schedule->command('health:check')->everySecond();
```

### Daemon Mode for Second-Based Scheduling
Running the scheduler in `daemon` mode enables second-level task execution, allowing schedules that fire more frequently than once per minute to be processed continuously without waiting for the next cron tick.
```bash
php pool cron:run --daemon
```

### Managing the Cron Daemon
Doppar provides a dedicated process manager for running the scheduler as a background service. This ensures reliable second-level execution, continuous task processing, and safe lifecycle control — all without relying on external tools like Supervisor or systemd.

The cron daemon keeps your schedules running even after you close your terminal.

Use this command to launch the scheduler in the background:
```bash
php pool cron:daemon start
```
This:
- Starts a persistent background process
- Enables continuous per-second scheduling
- Creates and stores a PID file at `storage/schedule/cron_daemon.pid`
- Logs daemon activity to `storage/schedule/daemon.log`
- Validates that no other daemon is already running

Once started, the daemon will process your second-based tasks continuously and reliably.

### Stop the Daemon
Gracefully shut down the running daemon:
```bash
php pool cron:daemon stop
```
This performs:
- Clean termination using system signals (SIGTERM)
- Protection against orphan processes
- Automatic PID file cleanup
- Fallback force-kill (SIGKILL) if graceful shutdown fails

Use this when deploying updates or stopping scheduled tasks temporarily.

### Restart the Daemon
Restart the daemon safely without manually stopping it:
```bash
php pool cron:daemon restart
```
A restart will:
- Stop the running daemon
- Wait briefly for a clean exit
- Start a fresh daemon instance

This ensures scheduling continues without interruption across deployments.

### Check Daemon Status
Verify whether the scheduling daemon is running:
```bash
php pool cron:daemon status
```
This displays:
- Running or stopped state
- PID (process ID)
- Start time and uptime
- PHP version and operating system
- Path to the log file and current log size

It is the easiest way to monitor the health and lifecycle of your scheduled task runner.

### When Should You Use the Cron Daemon?
Use daemon mode when your application needs:
- Second-based schedules (`everySecond()`, `everyFiveSeconds()`, etc.)
- High-frequency automation
- Continuous, long-running background jobs
- A persistent scheduling engine without external cron tools

Use standard mode (`php pool cron:run`) when minute-based tasks are sufficient.

### Standard Minute-Based Scheduling
Without daemon mode, the scheduler operates in the traditional minute-based cycle. This is suitable for commands that run every minute or use classic cron expressions.
```bash
php pool cron:run
```

## Run at Specific Seconds
The `atSeconds()` method allows a command to run at specific second marks within each minute. In the example below, the command executes four times per minute — at the 0th, 15th, 30th, and 45th second. Using `noOverlap()` ensures the next run will not start until the previous one has finished.
```php
$schedule->command('sync:data')
    ->atSeconds([0, 15, 30, 45])
    ->noOverlap();
```

## Customized Cron Expression
The `cron()` method accepts any standard cron expression, giving you full scheduling flexibility beyond the built-in helper methods. The expression `*/15 * * * *` runs the command every 15 minutes regardless of hour, day, or month.
```php
$schedule->command("user:sync")->cron("*/15 * * * *");
```
The `Phaseolies\Console\Schedule\ScheduledCommand` class also provides a rich set of named scheduling methods such as `everyMinute()`, `dailyAt()`, `hourly()`, and many more. See the [Common Scheduling Methods](#common-task-scheduling-methods) reference table at the end of this page.

## Throttle Command Frequency
The `throttle()` method limits how many times a command can run over a specific time window, regardless of how frequently the schedule or conditions would normally trigger it. This is useful for protecting system resources and rate-limiting expensive operations.
```php
$schedule->command("data:sync")->throttle("3/1d");
```
In the example above, `data:sync` will run a maximum of 3 times per day.

The format is `X/Y`, where:
- `X` is the number of allowed executions.
- `Y` is the time window: `s` (seconds), `m` (minutes), `h` (hours), or `d` (days).

Examples:
```php
// 5 executions per minute
->throttle('5/1m')

// 10 executions per hour
->throttle('10/1h')

// 10 executions per day
->throttle('10/1d')

// 3 executions per 30 seconds
->throttle('3/30s')
```

## Exclude Specific Dates
The `exclude()` method prevents a command from running on specific calendar dates. This is useful for holidays, planned maintenance windows, or any day you want to pause scheduled execution.
```php
$schedule->command("ls:la")->exclude(['2025-05-29', '2025-05-30']);
```
The command will not run on May 29 or May 30, 2025, even if it is normally scheduled to run on those days.

You can also pass dates as separate arguments instead of an array — both forms are equivalent:
```php
$schedule->command("ls:la")->exclude('2025-05-29', '2025-05-30');
```

## Retry on Failure
The `retry()` method configures automatic retries when a command fails. You define the maximum number of attempts and the delay in seconds between each attempt. Call `runWithRetry()` at the end of the chain to activate this behaviour.
```php
$schedule->command("email:send")->retry(3, 10)->runWithRetry();
```
In this example, if `email:send` fails, it will retry up to 3 more times with a 10-second wait between each attempt.

## Conditional Execution with when()
The `when()` method controls whether a command should run based on a runtime condition. It accepts a closure that must return `true` for the command to proceed.
```php
$schedule->command("backup:run")->when(fn() => true);
```
If the closure returns `false`, the command is skipped for that execution cycle without triggering any error or failure callback.

## Handle Success with onSuccess()
The `onSuccess()` method registers a callback that runs after a command completes successfully. Use it to send notifications, write audit logs, or trigger follow-up actions.
```php
$schedule->command("invoice:generate")
    ->onSuccess(function ($output) {
        Log::info("Invoice generated: " . $output);
    });
```
Once `invoice:generate` completes successfully, a log entry is written containing the command's output.

## Handle Failures with onFailure()
The `onFailure()` method registers a callback that runs when a command fails, giving you a structured way to handle errors without letting them go unnoticed.
```php
$schedule->command("payment:process")
    ->onFailure(function (\Exception $e, $attempt) {
        Log::error("Failed on attempt {$attempt}: " . $e->getMessage());
    });
```
If `payment:process` fails, the error and the attempt number are both logged.

## Execution Controls
Doppar provides fine-grained control over how background commands execute. These methods manage concurrency, process isolation, and output routing — ensuring that commands run safely without interfering with each other or polluting your log directory.

| Method                         | Description                                                                                                   |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------- |
| `->noOverlap($minutes = 1440)` | Prevent the command from running concurrently. Optional max lock time in minutes (default: 24 hours).         |
| `->inBackground()`             | Run the command in the background (non-blocking).                                                             |
| `->sendOutputTo($path)`        | Redirect background process stdout/stderr to the given file path instead of the default per-job log file.     |
| `->withoutLog()`               | Discard all background process output — no log file is created.                                               |

## Preventing Task Overlaps
By default, a new instance of a scheduled task can start even if the previous instance is still running. Use `noOverlap()` to prevent this. Doppar creates a file-based lock before the task starts and releases it automatically when the task finishes, regardless of whether it succeeded or failed.
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->noOverlap();
```
If needed, you may specify how many minutes must pass before the lock expires automatically. This acts as a safety net if a process crashes before it can release the lock. The default expiry is 24 hours (1440 minutes):
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->noOverlap(20);
```
Doppar also performs a live PID check when evaluating an existing lock. If the process that created the lock is no longer running, the lock is treated as stale, cleaned up, and a new run is allowed — even before the expiry time.

## Background Tasks
By default, when multiple tasks are scheduled to run at the same time, they execute sequentially in the order they are defined inside `schedule()`. This means a long-running task can delay all tasks that follow it. Use `inBackground()` to make a command run asynchronously, allowing subsequent tasks to start without waiting for it to complete.
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->inBackground();
```
When a command runs in the background, Doppar:
- Spawns a separate OS process for the command
- Captures its PID and records process metadata
- Chains a `cron:finish` callback to handle lock release and exit-code reporting once the process ends

Combining `inBackground()` with `noOverlap()` is the recommended approach for high-frequency or long-running tasks:
```php
$schedule->command('data:import pending --limit=500')
    ->everyFiveSeconds()
    ->inBackground()
    ->noOverlap();
```
Here, `noOverlap()` ensures that if an import run takes longer than 5 seconds, the next scheduled tick will detect the running process and skip, rather than launching a second concurrent import.

## Controlling Background Task Output
When `inBackground()` is used, Doppar writes each background job's stdout and stderr to a dedicated log file at `storage/schedule/cron_{hash}.log` by default. This is useful for debugging individual jobs, but it creates one file per unique command. You have two options to override this behaviour.

### Redirect Output to a Custom File
Use `sendOutputTo()` to write the background job's output to a specific file of your choice instead of the default per-job log file. This is useful when you want to consolidate output from multiple jobs into one place.
```php
$schedule->command('report:generate monthly')
    ->dailyAt('06:00')
    ->inBackground()
    ->sendOutputTo(storage_path('logs/reports.log'));
```
You can also point it at the main application log:
```php
$schedule->command('data:sync')
    ->everyFiveMinutes()
    ->inBackground()
    ->sendOutputTo(storage_path('logs/doppar.log'));
```

### Suppress All Output
Use `withoutLog()` to discard all background process output entirely. No log file will be created or written for that job. This is the cleanest option for high-frequency tasks where per-run output is not needed.
```php
$schedule->command('cache:warm')
    ->everyMinute()
    ->inBackground()
    ->withoutLog();
```
`withoutLog()` is a convenience shorthand for `->sendOutputTo('/dev/null')`.

> **Note:** Output suppression only affects the background process stdout/stderr. The daemon's own activity log at `storage/schedule/daemon.log` and Doppar's application log at `storage/logs/doppar.log` are unaffected.

## Check Registered Commands
Doppar allows you to inspect all registered scheduled commands and their configuration at a glance using the `cron:list` command.
```bash
php pool cron:list
```
#### Sample Output
| Command | Runs In    | Without Overlapping |
|---------|------------|---------------------|
| ls:ls   | Foreground | No                  |
| la:la   | Background | Yes                 |

This command is useful for verifying that your scheduled commands are registered correctly and confirming their concurrency and execution mode settings.

## Advanced Command Scheduling
The example below combines multiple scheduling features to build a robust, production-ready task setup:
```php
$schedule->command("invoice:send")
    ->timezone('Asia/Dhaka')
    ->exclude(['2025-05-29', '2025-05-30'])
    ->between('21:07', '23:30')
    ->inBackground()
    ->noOverlap()
    ->withoutLog()
    ->throttle('3/1d')
    ->when(fn() => $this->invoice->isPending())
    ->retry(3, 5)
    ->onSuccess(function ($output) {
        Log::info("Command succeeded: " . $output);
    })
    ->onFailure(function (\Exception $e, $attempt) {
        Log::error("Failed on attempt {$attempt}: " . $e->getMessage());
    })
    ->runWithRetry();
```
This schedule:
- Runs only between 21:07 and 23:30 in the Asia/Dhaka timezone
- Skips May 29 and 30, 2025
- Executes in the background without blocking other tasks
- Prevents concurrent overlapping runs
- Suppresses per-job log file creation
- Limits to 3 executions per day
- Only runs when there are pending invoices
- Retries up to 3 times on failure with a 5-second delay between each attempt
- Logs success and failure via callbacks

## Common Task Scheduling Methods
The `Phaseolies\Console\Schedule\ScheduledCommand` class provides a comprehensive set of frequency methods covering everything from per-second precision to yearly intervals. These methods replace complex cron expressions with readable, self-documenting code.

| Method                         | Description                                                                |
| ------------------------------ | -------------------------------------------------------------------------- |
| `everySecond()`                | Run the task **every second**.                                             |
| `everySeconds(int $seconds)`   | Run the task every **N seconds** (1–59).                                   |
| `everyFiveSeconds()`           | Run the task **every 5 seconds**.                                          |
| `everyTenSeconds()`            | Run the task **every 10 seconds**.                                         |
| `everyFifteenSeconds()`        | Run the task **every 15 seconds**.                                         |
| `everyTwentySeconds()`         | Run the task **every 20 seconds**.                                         |
| `everyThirtySeconds()`         | Run the task **every 30 seconds**.                                         |
| `atSeconds($seconds)`          | Run the task at **specific second(s)** within each minute.                 |
| `everyMinute()`                | Run the task **every minute**.                                             |
| `everyTwoMinutes()`            | Run the task **every 2 minutes**.                                          |
| `everyThreeMinutes()`          | Run the task **every 3 minutes**.                                          |
| `everyFourMinutes()`           | Run the task **every 4 minutes**.                                          |
| `everyFiveMinutes()`           | Run the task **every 5 minutes**.                                          |
| `everyTenMinutes()`            | Run the task **every 10 minutes**.                                         |
| `everyFifteenMinutes()`        | Run the task **every 15 minutes**.                                         |
| `everyTwentyMinutes()`         | Run the task **every 20 minutes**.                                         |
| `everyThirtyMinutes()`         | Run the task **every 30 minutes**.                                         |
| `hourly()`                     | Run the task **hourly** at minute 0.                                       |
| `hourlyAt(17)`                 | Run the task **hourly** at minute 17.                                      |
| `everyOddHour(0)`              | Run the task **every odd hour** (1, 3, 5, …) at minute 0.                  |
| `everyTwoHours(0)`             | Run the task **every 2 hours** at minute 0.                                |
| `everyFourHours(0)`            | Run the task **every 4 hours** at minute 0.                                |
| `everySixHours(0)`             | Run the task **every 6 hours** at minute 0.                                |
| `daily()`                      | Run the task **daily** at midnight (00:00).                                |
| `dailyAt('13:45')`             | Run the task **daily at 1:45 PM**.                                         |
| `twiceDaily(1, 13)`            | Run the task **twice daily** at 1:00 AM and 1:00 PM.                      |
| `twiceDailyAt(1, 13, 15)`      | Run the task **twice daily** at 1:15 AM and 1:15 PM.                      |
| `weekly()`                     | Run the task **weekly** on Sunday at midnight.                             |
| `weeklyOn(1, '8:00')`          | Run the task **weekly** on Monday at 8:00 AM.                              |
| `monthly()`                    | Run the task **monthly** on the 1st at midnight.                           |
| `monthlyOn(4, '15:00')`        | Run the task **monthly** on the 4th at 3:00 PM.                            |
| `twiceMonthly(1, 16, '13:00')` | Run the task on the **1st and 16th** of the month at 1:00 PM.              |
| `lastDayOfMonth('15:00')`      | Run the task on the **last day of the month** at 3:00 PM.                  |
| `quarterly()`                  | Run the task **quarterly** on Jan 1, Apr 1, Jul 1, Oct 1 at 00:00.        |
| `yearly()`                     | Run the task **yearly** on January 1st at midnight.                        |
| `cron('* * * * *')`            | Define a **custom cron expression** for full scheduling control.           |
| `between('22:00', '2:00')`     | Only run **between** the two given times (supports overnight windows).     |
| `timezone('Asia/Dhaka')`       | Set the **timezone** for this task's schedule evaluation.                  |

## Running the Scheduler
After defining your scheduled tasks, you need to ensure the scheduler runs on your server. Doppar provides two approaches depending on whether you want minute-level or second-level precision.

### Standard Minute-Based Scheduling (using system cron)
Add the following entry to your server's crontab. This runs the Doppar scheduler once per minute, triggering any tasks whose schedule matches the current time.
```bash
* * * * * cd /path-to-your-project && php pool cron:run >> /dev/null 2>&1
```

### Daemon Mode for Second-Based Scheduling (triggered via cron)
If you prefer to have the system cron manage the daemon, add this entry instead. The `--daemon` flag is ignored if the daemon is already running, so this is safe to leave in the crontab.
```bash
* * * * * cd /path-to-your-project && php pool cron:run --daemon >> /dev/null 2>&1
```

### Cron Daemon — No System Cron Required
Doppar includes its own process manager, meaning you do not need to add anything to the system crontab at all. Simply start the daemon once and it will keep running independently:
```bash
php pool cron:daemon start
```

Available daemon management commands:
```bash
php pool cron:daemon start      # Start the background scheduler
php pool cron:daemon stop       # Stop the scheduler gracefully
php pool cron:daemon restart    # Restart without manual stop/start
php pool cron:daemon status     # Show PID, uptime, and log info
```

This approach replaces OS-level cron entirely and gives you a reliable, self-contained scheduling engine built directly into Doppar.
