---
title: Task Schedule
description: Doppar Task Schedule page
meta:
  - name: keywords
    content: Task Schedule
---

## Task Schedule
### Introduction
In Doppar, scheduled tasks allow you to automate repetitive operations such as sending emails, processing invoices, or cleaning up stale data using cron jobs. Instead of relying on external tools or manual execution, you can define and control all your scheduled logic within a dedicated Schedule class. This ensures that your application remains self-contained and consistent in how background jobs are managed.

## Why Choose Doppar Scheduler
### Real-Time Precision
Doppar runs tasks with second-level accuracy, enabling real-time automation, monitoring, and event-driven pipelines. Perfect for applications that cannot wait for the next minute.

### Smart Dual-Mode Engine
Run tasks using standard cron or switch to Doppar’s built-in daemon for continuous execution. No external tools. No systemd. No Supervisor. Just plug-and-run.

### True Background Job Execution

Launch tasks in the background with full process management:
- Per-task PID tracking
- Process metadata
- Logs for each job
- Auto-cleanup after completion

This turns Doppar into a mini process manager.

### Crash-Resilient Daemon
If a task fails, Doppar's daemon continues running without dying. Signals like SIGTERM and SIGINT are handled gracefully, so shutdowns are safe and clean.

### Full Observability
Every task writes its own logs and metadata, making your system transparent and easy to debug. The daemon logs heartbeats, providing real-time operational visibility.

Doppar Task Scheduling System — Feature Overview
| Feature                  | Doppar Scheduling System                              | Notes / Advantage                                         |
| ------------------------ | ----------------------------------------------------- | --------------------------------------------------------- |
| **Execution Precision**  | Second-based scheduling (every X seconds)       | Enables high-frequency automation and real-time jobs      |
|                          | Real-time daemon mode                           | Runs tasks continuously                                   |
|                          | Minute, hourly, daily intervals                 | Full range of scheduling options                          |
| **Daemon Capabilities**  | Built-in daemon engine                          | No need for external tools like Supervisor or systemd     |
|                          | Graceful shutdown (SIGTERM, SIGINT)             | Safe cleanup + predictable exit behavior                  |
|                          | Crash-resistant loop with auto-recovery         | Daemon never silently stops; auto stabilizes              |
|                          | Heartbeat logging                               | Great for monitoring and uptime analytics                 |
| **Concurrency & Safety** | `noOverlap()` with per-command lock files       | Prevents double execution with strong guarantees          |
|                          | PID tracking for each task                      | Know exactly what is running at any moment                |
|                          | Atomic lock writing with retries                | Eliminates race conditions in concurrent environments     |
| **Process Management**   | First-class background jobs (`inBackground()`)  | Fully managed async task execution                        |
|                          | Per-task dedicated logs                         | Cleaner debugging + individual task history               |
|                          | Process metadata (PID, timestamps, OS, command) | Deep introspection for debugging and monitoring           |
|                          | Automatic cleanup after completion              | Prevents deadlocks and stale process references           |
| **Scheduling Engine**    | Dual-mode scheduling (standard + daemon)        | Works with cron AND real-time loops                       |
|                          | High-speed due-checking loop                    | Millisecond-level responsiveness                          |
|                          | Safe fallback when daemon is not running        | Tasks still run instead of being missed                   |
| **Developer Experience** | Fluent and expressive scheduling API            | Easy to read, easy to write                               |
|                          | Rich CLI output formatting                      | Clean, informative console messages                       |
|                          | Auto-detection of high-frequency tasks          | Smarter runtime behavior with minimal setup               |
| **Observability**        | JSON-formatted process info                     | Machine-readable — ideal for dashboards or log processors |
|                          | Separate logs for daemon and tasks              | Better separation of concerns                             |
|                          | Built-in warnings and status hints              | Helps diagnose issues instantly                           |
| **Platform Support**     | Full POSIX signal support                       | Behaves like a real system service                        |
|                          | Shell-based background process launching        | Compatible with Linux/macOS environments                  |


## Scheduling Pool Commands
You can register an Pool command using the command method, passing either the command's `name` or its fully qualified class name. If your command requires additional parameters, simply supply them as we discussed [here](http://doppar.com/versions/3.x/pool-console#command-arguments). You can register you pool command in `App\Schedule\Schedule` class inside `schedule()` method.
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

## Every-Second Scheduling
The `everySecond` method schedules the command to run once every second, providing the highest possible execution frequency within Doppar's task scheduler.
```bash
$schedule->command('health:check')->everySecond();
```

### Daemon Mode for Second-Based Scheduling
Running the scheduler in `daemon` mode enables second-level task execution, allowing schedules that run every second (such as `everySecond()`) to be processed continuously without waiting for the next minute tick.
```bash
php pool cron:run --daemon
```

### Managing the Cron Daemon
Doppar provides a dedicated process manager for running the scheduler as a background service. This ensures reliable second-level execution, continuous task processing, and safe lifecycle control — all without relying on external tools like Supervisor or systemd.

The Cron Daemon keeps your schedules running even after you close your terminal.

Use this command to launch the scheduler in the background:
```bash
php pool cron:daemon start
```
This:
- Starts a persistent background process
- Enables continuous per-second scheduling
- Creates and stores a PID file
- Logs activity to `storage/schedule/daemon.log`
- Validates that no other daemon is already running

Once started, the daemon will process your `everySecond()` tasks continuously and reliably.

### Stop the Daemon
Gracefully shut down the running daemon:
```bash
php pool cron:daemon stop
```
This performs:
- Clean termination using system signals
- Protection against orphan processes
- Automatic PID file cleanup
- Fallback “force kill” if graceful shutdown fails

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

Ensure scheduling continues without interruption

### Check Daemon Status
Verify whether the scheduling daemon is running:
```bash
php pool cron:daemon status
```

This displays:
- Running or stopped state
- PID (process ID)
- Start time
- Uptime
- PHP version and operating system
- Path to the log file and log size

It's the easiest way to monitor the health and lifecycle of your scheduled task runner.

### When Should You Use the Cron Daemon?
Use daemon mode when your application needs:
- Second-based schedules `(everySecond())`
- High-frequency automation
- Continuous, long-running background jobs
- A persistent scheduling engine without external cron tools

Use standard mode (php pool cron:run) when minute-based tasks are enough.

### Standard Minute-Based Scheduling
Without `daemon` mode, the scheduler operates in the traditional minute-based cycle. This is suitable for commands that run every minute or use classic cron expressions.
```bash
php pool cron:run
```

## Run at Specific Seconds
The `atSeconds` method allows a command to run at specific second marks within each minute. In this example, the command executes four times per minute—at the `0th`, `15th`, `30th`, and `45th` second.
Using `noOverlap()` ensures the next run will not start until the previous one has finished.
```php
$schedule->command('sync:data')
    ->atSeconds([0, 15, 30, 45]) // Runs 4 times per minute
    ->noOverlap();
```

## Customized Cron Expression
The cron method accepts a standard cron syntax (*/15 * * * *), which means the command will execute every 15 minutes, regardless of the hour, day, or month.
```php
$schedule->command("user:sync")->cron("*/15 * * * *");
```

The `Phaseolies\Console\Schedule\ScheduledCommand` class provides a rich set of scheduling methods such as `everyMinute()`, `dailyAt()`, `hourly()`, and more. These methods allow you to define precise execution intervals for your commands, giving you full control over task scheduling in a clean, readable way. You can explore this class to discover all available scheduling options and customize your tasks accordingly.

## Throttle Command Frequency
The `throttle()` method allows you to limit how many times a command can run over a specific time period. This is especially useful for preventing overuse of system resources.
```php
$schedule->command("data:sync")->throttle("3/1d");
```
In the example above, the command data:sync will run a maximum of 3 times per day, even if the schedule or conditions would normally trigger it more often.

The format is simple: `X/Y`, where:
- X is the number of allowed executions.
- Y is the duration: s (seconds), m (minutes), h (hours), or d (days).

You can adjust the values to match your needs, such as:
```php
// 5 attempts per minute
->throttle('5/1m')

// 10 attempt per hour
->throttle('10/1h')

// 10 attempts per day
->throttle('10/1d')

// 3 attempts per 30 seconds
->throttle('3/30s')
```

## Exclude Specific Dates
The `exclude()` method lets you skip running a command on specific dates. This is useful for holidays, maintenance windows, or any day you want to avoid running scheduled tasks.
```php
$schedule->command("ls:la")->exclude(['2025-05-29', '2025-05-30']);
```
In this example, the command will not run on May 29 or May 30, 2025, even if it's normally scheduled to run on those days.

You can also pass the dates as separate arguments instead of an array:
```php
$schedule->command("ls:la")->exclude('2025-05-29', '2025-05-30');
```
Both forms are supported and behave the same way.

## Retry on Failure
The `retry()` method allows your command to automatically retry if it fails. You can define how many times it should retry and how long to wait between each attempt (in seconds).
```php
$schedule->command("email:send")->retry(3, 10)->runWithRetry();
```
In this example, if the email:send command fails, it will retry up to 3 times, waiting 10 seconds between each attempt.

## Conditional Execution with when()
The `when()` method lets you control whether a command should run based on custom logic. It accepts a closure and that runs when the condition is true.
```php
$schedule->command("backup:run")->when(fn() => true)
```
In this example, the command will only run until the when is true.

## Handle Success with onSuccess()
The `onSuccess()` method allows you to define a callback that runs after a command finishes successfully. This is useful for logging, sending notifications, or triggering follow-up actions.
```php
$schedule->command("invoice:generate")
    ->onSuccess(function ($output) {
        Log::info("Invoice generated: " . $output);
    });
```
In this example, once the `invoice:generate` command completes successfully, a log entry will be written with the output of the command.

## Handle Failures with onFailure()
The `onFailure()` method allows you to handle errors gracefully by running a callback if the command fails.
```php
$schedule->command("payment:process")
    ->onFailure(function (\Exception $e, $attempt) {
        Log::error("Failed on attempt {$attempt}: " . $e->getMessage());
    });
```
In this example, if `payment:process` fails, the error is logged along with the attempt number and error message.

## Execution Controls
Doppar's task scheduling also provides fine-grained control over how and when scheduled commands are executed. These execution control methods help manage concurrency and system performance, ensuring that commands run smoothly without interfering with each other. For instance, you can prevent overlapping executions or run tasks in the background to avoid blocking your application. These controls are essential for maintaining stability in applications with long-running or frequent tasks.

| Method                       | Description                                                                                           |
| ---------------------------- | ----------------------------------------------------------------------------------------------------- |
| `->noOverlap($minutes = 1440)` | Prevent the command from running concurrently. Optional max lock time in minutes (default: 24 hours). |
| `->inBackground()`             | Run the command **in the background** (non-blocking).                                                 |

## Preventing Task Overlaps
By default, scheduled tasks will be run even if the previous instance of the task is still running. To prevent this, you may use the `noOverlap` method:
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->noOverlap();
```
If needed, you may specify how many minutes must pass before the "noOverlap" lock expires. By default, the lock will expire after 24 hours:
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->noOverlap(20);
```

Behind the scenes, the `noOverlap` method utilizes your application's cache locks.

## Check Registerd Command
Doppar allows you to schedule recurring tasks (cron jobs) that can be executed automatically at specified intervals. To inspect all registered cron jobs and their execution configurations, use the `cron:list` command.
```bash
php pool cron:list
```
#### Sample Output
| Command | Runs In    | Without Overlapping |
|---------|------------|---------------------|
| ls:ls   | Foreground | No                  |
| la:la   | Background | Yes                 |

This command helps monitor and debug scheduled tasks within your PHP application.

## Background Tasks
By default, when multiple tasks are scheduled to run at the same time, they execute one after another in the order they are listed within your schedule method. This means that long-running tasks can delay the start of tasks that follow. To allow tasks to run concurrently without waiting for previous tasks to finish, you can use the `inBackground` method. This runs the tasks `asynchronously`, enabling them to execute simultaneously in the background.
```php
$schedule->command("user:sync")->cron("*/15 * * * *")->inBackground();
```
## Advanced Command Scheduling
Below is a complete example that combines multiple scheduling features to build a robust, flexible, and safe task execution setup:
```php
$schedule->command("invoice:send")
    ->timezone('Asia/Dhaka')
    ->exclude(['2025-05-29', '2025-05-30'])
    ->between('21:07', '23:30')
    ->inBackground()
    ->noOverlap()
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

##  Common Task Scheduling Methods
In addition to basic scheduling, the `Phaseolies\Console\Schedule\ScheduledCommand` class provides a rich set of frequency methods that allow you to run tasks at precise and recurring intervals. Whether you want to run a job every few minutes, hourly at a specific time, or on certain days of the month, these methods offer flexible options tailored to your application's needs. Below is a list of commonly used scheduling methods, each designed to simplify complex CRON expressions into readable and reusable code.

| Method                         | Description                                                                |
| ------------------------------ | -------------------------------------------------------------------------- |
| `everySeconds(int $seconds)`   | Run the task **every second**.                                             |
| `everyFiveSeconds()`           | Run the task **every five seconds**.                                       |
| `everyTenSeconds()`            | Run the task **every ten seconds**.                                        |
| `everyFifteenSeconds()`        | Run the task **every fifteen seconds**.                                    |
| `everyTwentySeconds()`         | Run the task **every twenty seconds**.                                     |
| `everyThirtySeconds()`         | Run the task **every thirty second**.                                      |
| `atSeconds($seconds)`          | Run the task **specific second**.                                             |
| `everyMinute()`                | Run the task **every minute**.                                             |
| `everyTwoMinutes()`            | Run the task **every 2 minutes**.                                          |
| `everyThreeMinutes()`          | Run the task **every 3 minutes**.                                          |
| `everyFourMinutes()`           | Run the task **every 4 minutes**.                                          |
| `everyFiveMinutes()`           | Run the task **every 5 minutes**.                                          |
| `everyTenMinutes()`            | Run the task **every 10 minutes**.                                         |
| `everyFifteenMinutes()`        | Run the task **every 15 minutes**.                                         |
| `everyTwentyMinutes()`         | Run the task **every 20 minutes**.                                         |
| `everyThirtyMinutes()`         | Run the task **every 30 minutes**.                                         |
| `hourly()`                     | Run the task **hourly** at **minute 0**.                                   |
| `hourlyAt(17)`                 | Run the task **hourly** at **minute 17**.                                  |
| `everyOddHour(0)`              | Run the task **every odd hour** (1, 3, 5, …) at **minute 0**.              |
| `everyTwoHours(0)`             | Run the task **every 2 hours** at **minute 0**.                            |
| `everyFourHours(0)`            | Run the task **every 4 hours** at **minute 0**.                            |
| `everySixHours(0)`             | Run the task **every 6 hours** at **minute 0**.                            |
| `daily()`                      | Run the task **daily** at **midnight (00:00)**.                            |
| `dailyAt('13:45')`             | Run the task **daily at 1:45 PM**.                                         |
| `twiceDaily(1, 13)`            | Run the task **twice daily**, at **1:00 AM** and **1:00 PM**.              |
| `twiceDailyAt(1, 13, 15)`      | Run the task **twice daily**, at **1:15 AM** and **1:15 PM**.              |
| `weekly()`                     | Run the task **weekly**, on **Sunday at midnight**.                        |
| `weeklyOn(1, '8:00')`          | Run the task **weekly**, on **Monday at 8:00 AM**.                         |
| `monthly()`                    | Run the task **monthly**, on the **1st day at midnight**.                  |
| `monthlyOn(4, '15:00')`        | Run the task **monthly**, on the **4th at 3:00 PM**.                       |
| `twiceMonthly(1, 16, '13:00')` | Run the task on the **1st and 16th** of the month at **1:00 PM**.          |
| `lastDayOfMonth('15:00')`      | Run the task on the **last day of the month at 3:00 PM**.                  |
| `quarterly()`                  | Run the task **quarterly**, on the **1st of Jan, Apr, Jul, Oct at 00:00**. |
| `yearly()`                     | Run the task **yearly**, on **January 1st at midnight**.                   |
| `cron('* * * * *')`            | Define a **custom CRON expression** for full control.                      |
| `between('22:00', '2:00')`     | Only between 10pm and 2am (spanning midnight)                              |
| `timezone('Asia/Dhaka')`       | Set the **timezone** for this task’s schedule.                             |

## Running the Scheduler
After setting up your scheduled tasks, the next step is to run them on your server. The `cron:run` command in Doppar checks your scheduled tasks against the current server time and triggers any tasks that are due.

### Standard Minute-Based Scheduling (using system cron)
This is the classic approach. If your tasks run every minute or use standard cron expressions, add the following entry to your server’s crontab:

```bash
# Standard Minute-Based Scheduling
* * * * * cd /path-to-your-project && php pool cron:run >> /dev/null 2>&1
```
This runs the scheduler once per minute.

### Daemon Mode for Second-Based Scheduling (also triggered via cron)

```bash
# Daemon Mode for Second-Based Scheduling
* * * * * cd /path-to-your-project && php pool cron:run --daemon >> /dev/null 2>&1
```

### Cron Daemon — No System Cron Required
Doppar includes its own process manager, meaning you don’t need to add anything to the system crontab.
Just start the daemon using:
```bash
php pool cron:daemon start
```

Available daemon management commands
```bash
php pool cron:daemon start      # Start background scheduler
php pool cron:daemon stop       # Stop scheduler
php pool cron:daemon restart    # Restart scheduler
php pool cron:daemon status     # Check scheduler status
```

This approach replaces the need for OS-level cron entirely and gives you a reliable, self-contained scheduling engine similar to Supervisor or systemd — but built directly into Doppar.
