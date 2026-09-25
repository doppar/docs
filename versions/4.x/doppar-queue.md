---
title: Queue
description: Doppar queue manager
meta:
  - name: keywords
    content: queue manager
---

 - [Features](#features)
 - [Installation](#installation)
 - [Upgrading to v4.1.0](#upgrading-to-v410)
 - [Quick Start](#quick-start)
 - [Queue Connections and Drivers](#queue-connections-and-drivers) (v4.1.0)
 - [Job Priority](#job-priority) (v4.1.0)
 - [Unique Jobs](#unique-jobs) (v4.1.0)
 - [Bulk Dispatching](#bulk-dispatching) (v4.1.0)
 - [Leases and Crash Recovery](#leases-and-crash-recovery) (v4.1.0)
 - [Inspecting Queues](#inspecting-queues) (v4.1.0)
 - [Custom Drivers](#custom-drivers) (v4.1.0)
 - [Testing](#testing) (v4.1.0)
 - [Job Chaining](#job-chaining)
 - [Job Lifecycle](#job-lifecycle)
 - [Queue Commands](#queue-commands)
 - [Production Setup](#production-setup)

## Queue

### Introduction
A queue is a background job processing system that allows tasks to run asynchronously instead of blocking your main application flow. It improves performance, distributes workload efficiently, and ensures time-consuming tasks are handled reliably in the background. Queues are essential for scaling modern applications and maintaining smooth user experiences.

> **Doppar Queue v4.1.0** turns the queue into a driver-based system. Jobs can now be stored in **Redis** as well as the database, and the queue gains job priorities, unique jobs, retry backoff, automatic recovery of jobs from crashed workers, and more. Everything that is new in v4.1.0 is marked **Available from v4.1.0** throughout this page. Existing code keeps working — see [`Upgrade to v4.1.0`](#upgrade-to-v410)

Doppar queue system offers robust features including multiple queue support for organizing jobs by priority or category, automatic retry logic with configurable attempts and delays, and comprehensive failed job tracking for easier debugging. It supports delayed execution for scheduling jobs in the future, graceful shutdown handling to safely stop workers, and built-in memory management that automatically restarts workers when limits are exceeded.

## Features
The Doppar Framework queue system is designed to handle background tasks efficiently with reliability and scalability in mind. Its feature set ensures smooth job processing, better performance, and full control over how tasks are executed. See the features of doppar queue.

| Feature                               | Description                                                                                   |
| ------------------------------------- | --------------------------------------------------------------------------------------------- |
| **Multiple Queue Support**            | Organize jobs by priority and type.                                                           |
| **Automatic Retry Logic**             | Configurable retry attempts with delays.                                                      |
| **Failed Job Tracking**               | Store and analyze failed jobs for debugging.                                                  |
| **Delayed Execution**                 | Schedule jobs for future execution.                                                           |
| **Graceful Shutdown**                 | Handle SIGTERM and SIGINT signals safely.                                                     |
| **Memory Management**                 | Automatic worker restart when memory limits are exceeded.                                     |
| **Job Serialization**                 | Safely serialize complex job data for storage in queues.                                      |
| **Fluent API**                        | Fluent syntax for job dispatching and chaining.                                               |
| **Custom Failure Callbacks**          | Handle job failures gracefully at the job or chain level.                                     |
| **Job Chaining**                      | Execute multiple jobs sequentially, where each job runs only after the previous one succeeds. |
| **Chain-Level Callbacks**             | Define `then()` and `catch()` handlers for entire job chains.                                 |
| **Per-Job Execution Timeouts**        | Define maximum execution time per job using the `#[Queueable(timeout:)]` attribute.           |
| **Static and Instance Dispatching**   | Dispatch jobs via static methods or directly from job instances.                              |
| **Synchronous Job Execution**         | Optionally execute jobs immediately using `dispatchSync()`.                                   |
| **Worker Options**                    | Configure queue workers with `--queue`, `--sleep`, `--memory`, `--timeout`, and `--limit`.    |
| **Queue Drivers** *(v4.1.0)*          | Store jobs in the database, in Redis, or in memory, and add your own driver.                  |
| **Multiple Connections** *(v4.1.0)*   | Use several backends side by side and choose one per job, per worker, or per command.         |
| **Job Priority** *(v4.1.0)*           | Run urgent jobs first within a queue, with `withPriority()` or `#[Queueable(priority:)]`.     |
| **Priority Queue Lists** *(v4.1.0)*   | A worker can drain several queues in strict order: `--queue=high,default,low`.                |
| **Unique Jobs** *(v4.1.0)*            | Refuse a duplicate of a job that is already waiting or running.                               |
| **Retry Backoff** *(v4.1.0)*          | Wait longer after each failed attempt, e.g. `[10, 60, 300]` seconds.                          |
| **Crash Recovery** *(v4.1.0)*         | A job held by a worker that died is handed to another worker when its lease expires.          |
| **Automatic Lease Renewal** *(v4.1.0)*| Jobs with a timeout keep their lease while they run, so a long job is never run twice at once.|
| **Bulk Dispatching** *(v4.1.0)*       | Push many jobs at once with `Queue::pushMany()`.                                              |
| **Queue Statistics** *(v4.1.0)*       | Read ready, delayed, and running counts per queue with `Queue::stats()`.                      |

## Installation
You may install Doppar Queue via the composer require command:
```bash
composer require doppar/queue
```

### Register Launcher
Next, register the Queue launcher so that Doppar can initialize it properly. Open your `runtime/config/app.php` file and add the `QueueLauncher` to the launchers array:
```php
'launchers' => [
    // Other launchers
    \Doppar\Queue\QueueLauncher::class,
],
```

This step ensures that Doppar knows about Queue and can load its functionality when the application boots.

### Publish Configuration
Now we need to publish the configuration files by running this pool command.
```php
php pool vendor:publish --launcher="Doppar\Queue\QueueLauncher"
```

This publishes the queue migrations and, from **v4.1.0**, the `runtime/config/queue.php` configuration file (see [Queue Connections and Drivers](#queue-connections-and-drivers)). The configuration file is optional: without it the queue uses the `database` connection, exactly as before.

Now run migrate command to migrate queue related tables
```php
php pool migrate
```

This creates two tables:
- `queue_jobs` - Stores pending and processing jobs
- `failed_jobs` - Stores failed jobs for debugging

> These tables are only used by the `database` connection. If you use the [Redis driver](#redis-driver), jobs and failed jobs live in Redis and the tables stay unused.

## Upgrading to v4.1.0
> **Available from v4.1.0**

If you already use Doppar Queue, update the package first:
```bash
composer update doppar/queue
```

The database driver now reads three new columns of the `queue_jobs` table: `priority`, `lease_expires_at`, and `unique_key`. New installs get them from the queue migration when you run `php pool migrate`. If you created `queue_jobs` on an earlier version, that migration has already run and will not run again, so **add the columns with a migration of your own before starting workers on v4.1.0**:
```php
<?php

use Phaseolies\Support\Facades\Schema;
use Phaseolies\Database\Migration\Migration;
use Phaseolies\Database\Migration\Blueprint;

return new class extends Migration
{
    public function up(): void
    {
        // One Schema::table() call per column: SQLite only runs the first
        // statement of a batch.
        Schema::table('queue_jobs', function (Blueprint $table) {
            $table->smallInteger('priority')->default(0);
        });

        Schema::table('queue_jobs', function (Blueprint $table) {
            $table->unsignedInteger('lease_expires_at')->nullable();
        });

        Schema::table('queue_jobs', function (Blueprint $table) {
            $table->string('unique_key', 191)->nullable();
        });

        // A UNIQUE constraint cannot be added through ALTER TABLE on every
        // database, but a unique index can.
        db()->execute('CREATE UNIQUE INDEX queue_jobs_unique_key_unique ON queue_jobs (unique_key)');
    }

    public function down(): void
    {
        //
    }
};
```

Your existing jobs are not touched. The `database` connection is the only one that uses this table; the Redis driver needs no migration.

Everything you had keeps working: jobs, `#[Queueable]`, chains, `queue:run`, and jobs that only implement `JobInterface`. A few behaviors did change, and you should check them if you call the queue APIs directly:

| Change | What to do |
| ------ | ---------- |
| `Queue::pop()` returns a `Doppar\Queue\Support\ReservedJob` instead of the `QueueJob` model. It still has `$queue`, `$payload`, and `$attempts`, but not the model methods (`reserve()`, `release()`, `deleteJob()`). | Use `Queue::delete($job)`, `Queue::release($job, $delay)`, and `Queue::markAsFailed($job, $e)`. |
| `Queue::pop()`, `delete()`, `release()`, and `markAsFailed()` no longer swallow errors. A database or Redis outage now throws instead of returning `null` or `false`. | Workers already handle this and log it. Catch the exception in your own code if you call these methods. |
| `Queue::push()` and `dispatch()` return `null` when a [unique job](#unique-jobs) is refused as a duplicate. | Only matters if you use unique jobs. |
| `queue:monitor` shows **Ready**, **Delayed**, and **Processing** columns instead of **Pending** and **Processing**. | Update anything that parses its output. |
| Static properties of a job class are no longer written into the payload. Before, their value at dispatch time was saved and written back over the live value when the worker unserialized the job. | Nothing. This fixes a bug. |
| A job whose worker crashed and whose attempts are already used up is now moved to the failed jobs instead of running again. | Nothing. See [Leases and Crash Recovery](#leases-and-crash-recovery). |

## Quick Start
Doppar makes it easy to create a new queue job using the pool command. For example, to generate a job for sending a welcome email, run:
```bash
php pool make:job SendWelcomeEmailJob
```

This will create a ready-to-use job class that you can customize and dispatch to your queue.

### Dispatch the Job
Once your job class is ready, you can dispatch it to the queue like this:
```php
(new SendWelcomeEmailJob($user))->dispatch();
```

This sends the job to the queue for asynchronous processing, allowing your application to continue running without waiting for the task to complete.

Now update your newly create jobs as like this. By default you will get `#[Queueable]` as commented, uncomment it to use this job class as queueable.

```php
<?php

namespace App\Jobs;

use Doppar\Queue\Job;
use Doppar\Queue\Dispatchable;
use Doppar\Queue\Attributes\Queueable;
use App\Models\User;

#[Queueable]
class SendWelcomeEmailJob extends Job
{
    public function __construct(public User $user){}

    /**
     * Execute the job.
     *
     * @return void
     */
    public function handle(): void
    {
        info("Email has been sent to $this->user->email");
    }

    /**
     * Handle a job failure.
     *
     * @param \Throwable $exception
     * @return void
     */
    public function failed(\Throwable $exception): void
    {
        //
    }
}
```

### Run the Worker
To start processing queued jobs, run the worker using the following command:
```bash
php pool queue:run
```

> 💡 Your job class is now ready to work as a queueable job. However, if you remove or comment out the `#[Queueable]` attribute, it will run as a `synchronous` job, meaning it will execute immediately without being queued.

### Customize `#[Queueable]` Attributes
You can customize how a job behaves in the queue by configuring the `#[Queueable]` attribute directly on the job class:
```php
#[Queueable(
    tries: 3,
    retryAfter: 60,
    delayFor: 300,
    timeout: 60,
    onQueue: 'email',
)]
class SendWelcomeEmailJob extends Job
{
    //
}
```

From **v4.1.0** the attribute also accepts `priority`, `onConnection`, and `backoff`:
```php
#[Queueable(
    tries: 5,
    onQueue: 'email',
    priority: 20,             // v4.1.0 - run before lower priority jobs
    onConnection: 'redis',    // v4.1.0 - store this job in Redis
    backoff: [10, 60, 300],   // v4.1.0 - seconds to wait between retries
)]
class SendWelcomeEmailJob extends Job
{
    //
}
```

Now run the queue worker like this way
```bash
php pool queue:run --queue=email
```

Uncomment and adjust these values as needed to control the job's queueing behavior.

### Job Properties
Each job in Doppar queue can be configured with the following properties:
| Property      | Type   | Default     | Description                                   |
| ------------- | ------ | ----------- | --------------------------------------------- |
| `$tries`      | int    | 1           | Maximum number of attempts.                   |
| `$retryAfter` | int    | 60          | Seconds to wait before retrying a failed job. |
| `$queueName`  | string | `'default'` | The queue to which the job will be pushed.    |
| `$jobDelay`   | int    | 0           | Delay in seconds before executing the job.    |
| `$timeout`    | int(second)    | null        | Allows each job to define its own maximum execution time.|
| `$priority` *(v4.1.0)*   | int    | 0   | Higher runs first within a queue, from `-100` to `100`. See [Job Priority](#job-priority). |
| `$connection` *(v4.1.0)* | ?string | null | The [connection](#queue-connections-and-drivers) to store the job on. `null` uses the default. |
| `$backoff` *(v4.1.0)*    | int\|array | null | Seconds to wait between retries. See [Retry Backoff](#retry-backoff). |

### Per-Job Execution Timeouts
You can customize a job’s behavior using the `#[Queueable]` attribute. Each property can be used individually or combined, allowing you to fine-tune how a specific job is executed.

For example, the `timeout` property lets a job define its own maximum execution time, providing fine-grained control beyond the worker-level timeout.
```php
use Doppar\Queue\Attributes\Queueable;

#[Queueable(timeout: 60)]
class SendEmailJob extends Job
{
    public function handle(): void
    {
        // This job will timeout after 60 seconds
        // If a job exceeds its defined execution timeout,
        // It is considered as a failed job
    }
}
```
All available Queueable properties can be used `together`, `partially`, or `individually`, depending on the needs of the job.

### Retry Jobs
Sometimes jobs may fail due to temporary issues, like network errors or external service downtime. Doppar queue allows you to automatically retry failed jobs a configurable number of times. You can specify the number of attempts for each job using the `$tries` property in your job class.
```php
<?php

namespace App\Jobs;

use Doppar\Queue\Job;

class SendWelcomeEmailJob extends Job
{
    /**
     * The number of times the job may be attempted.
     *
     * @var int
     */
    public $tries = 3;
}
```

### Retry Delay
You can control how long the queue should wait before retrying a failed job using the `$retryAfter` property. This allows temporary issues to resolve before the job is attempted again, preventing immediate repeated failures.
```php
<?php

namespace App\Jobs;

use Doppar\Queue\Job;

class SendWelcomeEmailJob extends Job
{
    /**
     * The number of seconds to wait before retrying.
     *
     * @var int
     */
    public $retryAfter = 60;
}
```

### Retry Backoff
> **Available from v4.1.0**

A fixed `$retryAfter` waits the same time after every failure. When a service is struggling, it is usually better to wait longer each time. Set `$backoff` to a list of seconds, indexed by attempt. The last value is used again for any further attempts:
```php
class SyncInventoryJob extends Job
{
    public $tries = 5;

    /**
     * Wait 10s after the 1st failure, 60s after the 2nd, and 300s after the 3rd and later.
     *
     * @var array<int, int>
     */
    public $backoff = [10, 60, 300];
}
```

A single number is a fixed wait, the same as `$retryAfter`. When `$backoff` is not set, `$retryAfter` is used, so existing jobs behave as before.

### With Queue and Delay
You can specify a custom queue and set a delay for job execution. This is useful when you want to separate jobs by type or control when they are processed.
```php
(new ProcessVideo($videoPath))
    ->onQueue('videos')
    ->delayFor(300)
    ->dispatch();
```

In this example, the job is pushed to the videos queue and will execute after a `300-second` delay. To start processing this specific queue, run:
```bash
php pool queue:run --queue=videos
```

### Using Static Helper
Doppar provides a convenient static helper to dispatch jobs immediately without manually instantiating them:
```php
SendWelcomeEmail::dispatchNow($user);
```
This creates a new instance of the `SendWelcomeEmail` job and pushes it to the queue in a single line. Ideal for quick dispatching when you don’t need to customize the job further.

### Dispatch to a Specific Queue
You can assign a job to a particular queue using the `dispatchOn` method. This is useful for separating jobs by type or priority:
```php
(new GenerateReport($data))->dispatchOn('reports');
```

The job is sent to the reports queue, allowing you to run workers dedicated to specific queues for better control and efficiency.

### Dispatch After a Delay
Jobs can be scheduled to run after a specific delay using `dispatchAfter`. This is perfect for sending reminders or delayed notifications:
```php
(new SendReminder($user))->dispatchAfter(3600); // 1 hour
```
Here, the job will be executed 1 hour later, without blocking your application or requiring manual scheduling.

### Force Queue
To ensure a job is always pushed to the queue—regardless of whether it uses the `#[Queueable]` attribute—you can explicitly force it to queue:
```php
(new GenerateReport($data))->forceQueue();
```

### Dispatch as Sync
To force a job to run immediately—without being queued—you can dispatch it synchronously:
```php
SendReminder::dispatchSync($user);
```

## Dispatching with Static API
You can now dispatch jobs directly using static methods, eliminating the need to manually instantiate job objects. This makes the process simpler, more readable, and flexible.
```php
$jobId = SendEmailJob::dispatchWith($user);
```
The `SendEmailJob` will work as like
- If the job class uses the `#[Queueable]` attribute, it will be queued
- If the job class does not have the `#[Queueable]` attribute, it will run synchronously

### Dispatch Job Synchronously
You can force a job to run immediately, bypassing the queue, by using the `queueAsSync` method:
```php
SendEmailJob::queueAsSync($user);
```
The job executes instantly without being pushed to any queue.

### Dispatch Job to a Specific Queue
You can force a job to always be queued on a specific queue using the `queueOn` method:
```php
// Queue the job on the 'high-priority' queue
$jobId = SendEmailJob::queueOn('high-priority', $user);

// Queue another job on the 'reports' queue
$jobId = GenerateReport::queueOn('reports', $reportData);
```
Dispatches the job to a specified queue, regardless of whether the job class has the `#[Queueable]` attribute. Ensures the job is always queued.

### Dispatch Job with Delay
You can schedule a job to be queued after a specified delay using the `queueAfter` method:
```php
// Queue the job to run 5 minutes later (300 seconds)
$jobId = SendEmailJob::queueAfter(300, $user);

// Queue another job to run 10 minutes later
$jobId = GenerateReport::queueAfter(600, $reportData);
```
Dispatches the job to the queue after a delay (in seconds), regardless of the `#[Queueable]` attribute.

## Queue Connections and Drivers
> **Available from v4.1.0**

Before v4.1.0 the queue always used the database. It now works through *drivers*, and a *connection* is a driver plus its settings. Doppar ships three drivers:

| Driver     | Stores jobs in | Best for |
| ---------- | -------------- | -------- |
| `database` | The `queue_jobs` and `failed_jobs` tables | Getting started, and apps that already run a database. This is the default. Works on MySQL, PostgreSQL, and SQLite. |
| `redis`    | Redis | High throughput and many workers. Needs `predis/predis`. |
| `memory`   | The current PHP process | Tests. Jobs are lost when the process ends. |

### Configuration
Connections are defined in `runtime/config/queue.php`:
```php
return [
    // The connection used when a job does not ask for one.
    'default' => env('QUEUE_CONNECTION', 'database'),

    'connections' => [

        'database' => [
            'driver' => 'database',
            'connection' => null,           // database connection; null uses your default
            'table' => 'queue_jobs',
            'failed_table' => 'failed_jobs',
            'lease' => 90,
        ],

        'redis' => [
            'driver' => 'redis',
            'connection' => env('REDIS_URL', 'redis://127.0.0.1:6379'),
            'options' => [
                'parameters' => [
                    'password' => env('REDIS_PASSWORD', null),
                    'database' => env('REDIS_DB', 0),
                ],
            ],
            'prefix' => '{doppar_queue}',   // keep the braces
            'lease' => 90,
        ],

        'memory' => [
            'driver' => 'memory',
            'lease' => 90,
        ],
    ],
];
```

`lease` is how many seconds a worker may hold a job before another worker can take it over. See [Leases and Crash Recovery](#leases-and-crash-recovery).

To switch your whole application, set the default connection in your `env.toml`:
```bash
QUEUE_CONNECTION = "redis"
```

### Redis Driver
Install the Redis client, then select the `redis` connection:
```bash
composer require predis/predis
```

The Redis driver runs every state change as a single Lua script, so claiming, releasing, and failing a job are atomic without any locking on your side. It uses only core Redis commands and needs no Redis modules. Every key shares one hash tag (the braces in `prefix`) so that the scripts only touch a single Cluster slot, but the driver has only been tested against a single Redis server.

Failed jobs are stored in Redis too, so `queue:failed`, `queue:retry`, and `queue:flush` work the same way on every driver.

> Give each application its own `prefix` when several applications share one Redis server.

### Choosing a Connection
A job can choose its own connection:
```php
(new SendWelcomeEmailJob($user))
    ->onConnection('redis')
    ->dispatch();
```

Or set it on the job class, or with the attribute:
```php
class SendWelcomeEmailJob extends Job
{
    public ?string $connection = 'redis';
}

#[Queueable(onConnection: 'redis')]
class GenerateReportJob extends Job {}
```

Without a choice, the job goes to the default connection. A chained job uses the connection of the job before it unless it sets its own.

A worker processes one connection, the default unless you pass `--connection`:
```bash
php pool queue:run --connection=redis --queue=emails
```

> A worker only sees jobs on its own connection. If you send jobs to `redis`, run a worker with `--connection=redis`.

### Working with a Connection Directly
The `Queue` facade forwards to the default connection, and `Queue::connection()` gives you any connection's driver:
```php
use Doppar\Queue\Facades\Queue;

Queue::size('emails');                         // default connection
Queue::size('emails', 'redis');                // a named connection
Queue::connection('redis')->stats('emails');   // the driver itself
```

## Job Priority
> **Available from v4.1.0**

Within one queue, jobs normally run in the order they arrived. Give a job a priority to let it jump ahead. Higher numbers run first, from `-100` (lowest) to `100` (highest), and the default is `0`. Jobs with the same priority still run oldest first:
```php
(new SendPasswordResetJob($user))->withPriority(90)->dispatch();
(new SendNewsletterJob($user))->withPriority(-10)->dispatch();
```

Or set it on the job:
```php
#[Queueable(priority: 90)]
class SendPasswordResetJob extends Job {}
```

Values outside the range are clamped to it.

### Processing Several Queues in Order
A worker can take several queues, separated by commas. It always looks at them in the order given and only moves on to the next queue when the earlier ones are empty:
```bash
php pool queue:run --queue=high,default,low
```

Use priority to order jobs *inside* a queue, and a queue list to order the queues themselves.

## Unique Jobs
> **Available from v4.1.0**

Sometimes a second copy of a job is pointless: rebuilding the same report twice, or syncing the same record again while the first sync is still waiting. Override `uniqueId()` to give the job a key. While a job with the same key is waiting or running, dispatching another is refused:
```php
class RebuildReportJob extends Job
{
    public function __construct(public int $reportId) {}

    public function uniqueId(): ?string
    {
        return (string) $this->reportId;
    }

    public function handle(): void
    {
        // ...
    }
}

$first = (new RebuildReportJob(7))->dispatch();   // queued, returns the job id
$again = (new RebuildReportJob(7))->dispatch();   // refused, returns null
(new RebuildReportJob(8))->dispatch();            // different key, queued
```

The key is freed as soon as the job finishes, fails, or is cleared, so the report can be queued again afterwards. Keys belong to the job class, so two different job classes can use the same id without clashing. Returning `null` from `uniqueId()`, which is the default, means the job is not unique.

Uniqueness is enforced by the queue backend itself, so it works across many workers and web servers without a separate cache or lock.

> A retry of a failed job (`queue:retry`) is refused if an identical unique job was queued in the meantime. The failed job stays in the failed list so you do not lose it.

## Bulk Dispatching
> **Available from v4.1.0**

To queue many jobs at once, use `Queue::pushMany()`. It sends jobs in as few round trips as the driver allows, which is much faster than dispatching one at a time:
```php
use Doppar\Queue\Facades\Queue;

$jobs = [];

foreach ($users as $user) {
    $jobs[] = new SendNewsletterJob($user);
}

$queued = Queue::pushMany($jobs);   // number of jobs stored
```

Jobs keep their order. Each job goes to its own connection and queue, and duplicates of [unique jobs](#unique-jobs) are skipped and not counted. `pushMany()` always queues the jobs; it does not run them synchronously, whether or not the class has `#[Queueable]`.

## Leases and Crash Recovery
> **Available from v4.1.0**

When a worker takes a job, it holds a *lease* on it for `lease` seconds (90 by default). While the lease is held, no other worker can take the job. If the worker finishes, the job is deleted. If the worker dies — killed, out of memory, server restart — the lease runs out and the job becomes available to another worker, with its attempt count kept.

This means a crash never loses a job and never leaves it stuck. It also means the job may run more than once (a worker could die after doing the work but before deleting the job), so write jobs that are safe to run again.

Some details worth knowing:
- **A crash loop is stopped.** If a job keeps killing its worker, its attempts eventually run out. Instead of running it again, the worker moves it to the failed jobs with a `MaxAttemptsExceededException`.
- **Set `lease` above your longest job.** Otherwise a slow job's lease can run out while it is still running, and a second worker starts it again.
- **Jobs with a timeout renew their lease automatically.** A job that sets `timeout` (or `#[Queueable(timeout: ...)]`) runs in its own process, and the worker extends the lease every 20 seconds while it runs. Such a job can run longer than `lease` without being started twice. Keep `lease` at 30 seconds or more when you rely on this.
- **A worker that lost its lease cannot interfere.** If a slow worker finishes after another worker took over its job, its attempt to delete or fail the job is refused, so it cannot remove the other worker's job. The worker logs that the lease was lost and does not run follow-up work such as the next job in a chain. The worker that owns the job does that.

> Jobs without a `timeout` run inside the worker process and are not renewed. Set a `timeout` on long jobs, or raise `lease`.

## Inspecting Queues
> **Available from v4.1.0**

`Queue::stats()` breaks a queue down by state:
```php
use Doppar\Queue\Facades\Queue;

Queue::stats('emails');
// ['ready' => 12, 'delayed' => 3, 'reserved' => 2]
```

- **ready** — can be taken by a worker right now (this includes jobs whose lease ran out)
- **delayed** — waiting for their delay or their retry backoff
- **reserved** — being worked on under a live lease

`Queue::size('emails')` counts ready and delayed jobs together. To list the queues that currently hold jobs, use `Queue::connection()->queues()`. The same numbers are shown by `php pool queue:monitor`.

## Custom Drivers
> **Available from v4.1.0**

You can add your own backend, such as Amazon SQS, by implementing `Doppar\Queue\Contracts\QueueDriver` and registering it, for example in a launcher:
```php
use Doppar\Queue\Facades\Queue;

Queue::extend('sqs', function (array $config, \Closure $clock) {
    return new SqsQueueDriver($config, $clock);
});
```

Then use it in `runtime/config/queue.php`:
```php
'connections' => [
    'sqs' => ['driver' => 'sqs', 'region' => 'eu-west-1'],
],
```

The driver receives the connection's config, including its `name`, and a `$clock` closure that returns the current time. Use the clock instead of `time()` so the driver can be tested without waiting. A driver must provide these guarantees:

- Two workers never receive the same job at the same time.
- Within a queue, higher priority runs first, then the oldest job.
- A claimed job is invisible to other workers until its lease expires, then it can be claimed again with its attempts kept.
- `delete`, `release`, `extend`, and `fail` only succeed for the current claim. Use the claim's `attempts` value as the check.
- A job with a unique key is refused while another job with the same key is waiting or running.

The package's own test suite checks all of these for every built-in driver with one shared test class, `QueueDriverContract` in the package's `tests/Contract` directory. It is a good starting point: extend it, implement `makeDriver()`, and the same checks run against your driver.

## Testing
> **Available from v4.1.0**

The `memory` driver keeps jobs in the current process, so tests do not need a database or Redis. Point the default connection at it and use the queue as usual:
In your test environment's `env.toml`:
```bash
QUEUE_CONNECTION = "memory"
```

Then in a test:
```php
use Doppar\Queue\Facades\Queue;

(new SendWelcomeEmailJob($user))->forceQueue();   // always queues, with or without #[Queueable]

$this->assertSame(1, Queue::size());

$job = Queue::pop();                              // claim it, as a worker would
$this->assertNotNull($job);

Queue::delete($job);
```

You can also run a worker for a single job, without starting a daemon:
```php
use Doppar\Queue\QueueWorker;

$worker = new QueueWorker(app(\Doppar\Queue\QueueManager::class));

$worker->runNextJob('default');   // returns true if a job was processed
```

## Job Chaining
Job Chaining allows you to execute multiple jobs sequentially, where each job runs only after the previous one succeeds. This makes it perfect for multi-step workflows where order and dependencies matter.
### How Job Chaining Works
- A chain is created with a list of jobs.
- Only the first job is pushed to the queue.
- After each job completes, the next job in the chain is automatically dispatched.
- If any job fails:
  - The chain stops immediately
  - Remaining jobs are never executed
  - Optional failure callbacks can handle errors globally for the chain

This ensures reliable sequential execution while avoiding partial state issues caused by failed intermediate jobs.

### Basic Example
```php
use Doppar\Queue\Drain;

Drain::conduct([
    new DownloadJob($url),
    new ProcessJob($path),
    new UploadJob($file),
    new NotifyJob($userId),
])->dispatch();
```
What happens step by step
- `DownloadJob` executes.
- `ProcessJob` executes only if DownloadJob succeeds.
- `UploadJob` executes only if ProcessJob succeeds.
- `NotifyJob` executes only if UploadJob succeeds.

### Instance Method Chaining
Doppar allows you to create a job chain directly from a job instance using the `chain()` method. This can make your code more expressive, especially when starting from a specific job object.
```php
$chainId = (new Job1())->chain([
    new Job2(),
    new Job3(),
])->dispatch();
```
If any job fails, the chain stops immediately, and subsequent jobs are never executed.

### Job Chaining With Configuration
Doppar allows you to customize job chains with queue options, delays, and other configurations before dispatching
```php
use Doppar\Queue\Drain;

Drain::conduct([
    new Job1(),
    new Job2()
])
->onQueue('priority')  // Specify the queue name
->delayFor(60)         // Delay execution by 60 seconds
->dispatch();
```
If any job fails, the chain stops, respecting failure handling rules as like before.

### Job Chaining With Callbacks
Doppar allows you to attach global success and failure handlers to a job chain. This makes it easy to handle completion logic or errors at the chain level, without modifying individual jobs.
```php
use Doppar\Queue\Drain;

Drain::conduct([
    new Job1(),
    new Job2(),
    new Job3()
])
->onQueue('priority')
->delayFor(60)
->then(fn() => echo "Done!")
->catch(fn($job, $ex, $index) => Log::error($ex))
->dispatch();
```
The `catch` callback is executed with:
- `$job` — the job that failed
- `$ex` — the thrown exception
- `$index` — position of the failed job in the chain

If all jobs succeed, the `then` callback is executed.

### Synchronous Job Chaining
By default, Doppar queues dispatch jobs asynchronously. Sometimes, you may want to execute a chain immediately, blocking the current process until all jobs complete.
```php
use Doppar\Queue\Drain;

Drain::conduct([
    new Job1(),
    new Job2()
])
->dispatchSync(); // Blocks until all jobs complete
```
> **Note:** `dispatchSync()` runs the chain immediately in the current process and blocks until all jobs complete.

### Running the Queue Worker
To start processing jobs from the default queue, simply run:
```bash
php pool queue:run
```

The worker will continuously poll the queue and execute jobs as they arrive.

### Custom Options
You can customize the worker’s behavior with the following options:
```bash
php pool queue:run --queue=emails --sleep=3 --memory=256 --timeout=3600 --limit=1000
```

- `--queue` – Process jobs from a specific queue (e.g., emails). From v4.1.0 you can pass several queues separated by commas, such as `high,default,low`; they are tried in that order.
- `--connection` – *(v4.1.0)* The [queue connection](#queue-connections-and-drivers) to process. Defaults to the default connection.
- `--sleep` – Number of seconds to wait between polling when no jobs are available.
- `--memory` – Maximum memory in MB before the worker automatically restarts.
- `--timeout` – Maximum execution time in seconds before the worker stops.
- `--limit` — Maximum number of jobs the worker will process before exiting

This allows you to run workers efficiently in production and tailor them to different job types or workloads.

### Available Options
These options let you customize how the worker polls the queue, manages resources, and handles long-running tasks efficiently.
| Option      | Description                             | Default   |
| ----------- | --------------------------------------- | --------- |
| `--queue`   | Name of the queue to process, or a comma separated list in priority order (list: v4.1.0) | `default` |
| `--connection` | *(v4.1.0)* Queue connection to process | the default connection |
| `--sleep`   | Seconds to wait when the queue is empty | `3`       |
| `--memory`  | Maximum memory in MB before restarting  | `128`     |
| `--timeout` | Maximum execution time in seconds, `0` for no limit | `0` |
| `--limit`   | Maximum number of jobs the worker will process before exiting| `unlimited`|

### Running Multiple Workers
You can run multiple workers simultaneously to process different queues in parallel. This is useful for prioritizing tasks or separating workloads:
```bash
# Terminal 1 – High priority queue
php pool queue:run --queue=high-priority &

# Terminal 2 – Default queue
php pool queue:run --queue=default &

# Terminal 3 – Low priority queue
php pool queue:run --queue=low-priority &
```

With Redis, workers on different servers can safely share the same queues: a job is only ever handed to one worker at a time.

## Job Lifecycle
When using the Doppar queue system, every job follows a structured lifecycle from dispatch to completion. Understanding this lifecycle helps you manage retries, failures, and queue processing more effectively.

The following diagram illustrates how Doppar handles single jobs and job chains, from dispatch to completion or failure:

```bash
┌─────────────────────────────────────────────────────────────────┐
│                    JOB DISPATCH LAYER                           │
└─────────────────────────────────────────────────────────────────┘
                             │
                 ┌───────────┴───────────┐
                 │                       │
                 ▼                       ▼
        ┌────────────────┐      ┌────────────────┐
        │  Single Job    │      │   Job Chain    │
        │   Dispatch     │      │     Drain      │
        └────────┬───────┘      └────────┬───────┘
                 │                       │
                 │                       │
                 └───────────┬───────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│          QUEUE STORAGE - Database, Redis or Memory              │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ Each job keeps: queue, payload, attempts, priority,      │   │
│  │ available_at, lease (reserved until), unique key         │   │
│  │ (v4.1.0: priority, lease and unique key)                 │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     WORKER PROCESSING                           │
│  ┌────────────────────────────────────────────────────────┐     │
│  │ • Pop the best job (priority, then oldest)             │     │
│  │ • Take a lease on it - mark as processing              │     │
│  │ • Increment attempts counter                           │     │
│  │ • Fail it if attempts already exceed tries             │     │
│  │ • Execute with timeout protection, renewing the lease  │     │
│  └────────────────────────────────────────────────────────┘     │
└────────────────────────────┬────────────────────────────────────┘
                             │
                   ┌─────────┴─────────┐
                   │                   │
                SUCCESS             FAILURE
                   │                   │
                   ▼                   ▼
         ┌──────────────────┐  ┌──────────────────┐
         │ Job Completed    │  │ Job Failed       │
         │ Successfully     │  │ with Exception   │
         └────────┬─────────┘  └────────┬─────────┘
                  │                     │
                  ▼                     ▼
         ┌──────────────────┐  ┌──────────────────┐
         │ 1. Delete from   │  │ Check Retry      │
         │    the queue     │  │ Logic            │
         │ 2. Check if      │  └────────┬─────────┘
         │    chained       │           │
         └────────┬─────────┘  ┌────────┴────────┐
                  │            │                 │
                  │         attempts < tries?    │
                  │            │                 │
                  │      ┌─────┘                 └─────┐
                  │      │ YES                      NO │
                  │      ▼                             ▼
                  │ ┌──────────────┐      ┌──────────────────┐
                  │ │ Release Job  │      │ Mark as Failed   │
                  │ │ Back to      │      │ • Move to the    │
                  │ │ Queue after  │      │   failed store   │
                  │ │ retryAfter / │      │ • Delete from    │
                  │ │ backoff      │      │   the queue      │
                  │ └──────────────┘      │                  │
                  │                       │ • Call failed()  │
                  │                       │   callback       │
                  │                       └──────────────────┘
                  │
        ┌─────────┴─────────┐
        │ Is job chained?   │
        └─────────┬─────────┘
                  │
          ┌───────┴───────┐
          │ YES           │ NO
          ▼               ▼
    ┌──────────────┐  ┌──────────────┐
    │ Check Chain  │  │ Job Complete │
    │ Status       │  │     End      │
    └──────┬───────┘  └──────────────┘
           │
    ┌──────┴──────┐
    │ More jobs   │
    │ in chain?   │
    └──────┬──────┘
           │
    ┌──────┴──────┐
    │ YES         │ NO
    ▼             ▼
┌──────────┐  ┌──────────────────┐
│ Dispatch │  │ Chain Complete   │
│ Next Job │  │ • Call then()    │
│  →chainA │  │   callback       │
│  →chainB │  │ • End            │
└──────────┘  └──────────────────┘
```

This diagram clearly shows the end-to-end lifecycle of single jobs and chained jobs, including retries, failures, and chain completion callbacks.

If a worker dies while a job is being processed, the diagram does not reach SUCCESS or FAILURE. Instead the job's lease runs out and the job returns to the queue for another worker (v4.1.0). See [Leases and Crash Recovery](#leases-and-crash-recovery).

## Queue Commands
Doppar provides a set of CLI commands to manage and monitor your queues efficiently. These commands allow you to run workers, inspect failed jobs, retry or flush jobs, and view queue statistics.

### Process Jobs
Processes jobs in the queue one by one. You can specify options such as queue name, sleep interval, memory limit, and execution timeout.
```bash
# Run the default queue
php pool queue:run

# Run a specific queue with custom options
php pool queue:run --queue=emails --sleep=3 --memory=256 --timeout=3600

# v4.1.0 - run several queues in priority order, on the redis connection
php pool queue:run --queue=high,default,low --connection=redis
```

### List Failed Jobs
Lists all jobs that have failed and been recorded in the failed jobs table. Useful for monitoring and debugging job issues.
```bash
# List all failed jobs
php pool queue:failed

# v4.1.0 - list the failed jobs of another connection
php pool queue:failed --connection=redis
```

### Delete Failed Jobs
Deletes failed jobs from the failed jobs table. You can delete a specific job by ID or all failed jobs if no ID is provided.
```bash
# Delete a specific failed job
php pool queue:flush --id=5

# Delete all failed jobs
php pool queue:flush

# v4.1.0 - target another connection
php pool queue:flush --connection=redis
```

### Retry Failed Jobs
Retries failed jobs either by ID or all failed jobs if no ID is specified. Jobs will be pushed back to the queue for reprocessing.
```bash
# Retry a specific failed job
php pool queue:retry --id=3

# Retry all failed jobs
php pool queue:retry

# v4.1.0 - target another connection
php pool queue:retry --connection=redis
```

A retried job starts again with a fresh attempt count. From v4.1.0, a failed [unique job](#unique-jobs) is not retried while an identical one is already queued, and it stays in the failed list.

### Monitor Queue Statistics
Displays statistics for every queue that holds jobs, plus the number of failed jobs. Useful for monitoring queue health and performance.
```bash
# Monitor all queues
php pool queue:monitor

# v4.1.0 - monitor another connection
php pool queue:monitor --connection=redis
```

From v4.1.0 the table has these columns:
| Column | Meaning |
| ------ | ------- |
| **Ready** | Jobs a worker can take right now |
| **Delayed** | Jobs waiting for their delay or retry backoff |
| **Processing** | Jobs currently held by a worker |

## Production Setup
To run Doppar queue workers in production reliably, it’s recommended to use Supervisor to manage worker processes. Supervisor ensures that workers automatically restart if they fail and allows you to run multiple processes in parallel.

### Create Supervisor Configuration
Create a file at `/etc/supervisor/conf.d/queue-worker.conf` with the following contents:
```bash
[program:queue-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/pool queue:run --sleep=3 --memory=256
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=4
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/worker.log
stopwaitsecs=3600
```

### Start Supervisor
Reload Supervisor to apply the new configuration and start the workers:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start queue-worker:*
```


### Choosing a Backend for Production
The `database` connection is simple and works well for moderate volume. For high volume or many workers, use the [Redis driver](#redis-driver) (v4.1.0): it takes the queue load off your database and handles many workers well.

Whatever the driver, set `lease` above your longest job, use a `timeout` on long jobs, and start each worker with the `--connection` it should serve. Workers on the same connection can run on several servers at once.

### Monitor Workers
You can check the status of your Doppar queue workers managed by Supervisor with:
```bash
sudo supervisorctl status queue-worker:*
```
This displays all running worker processes, their current state, and uptime.

### Automated Monitoring
To keep track of queue statistics regularly, you can create a cron job that runs the Doppar queue monitor every 5 minutes:
```bash
*/5 * * * * php /var/www/html/pool queue:monitor >> /var/log/queue-monitor.log
```

This will append queue statistics, such as ready and failed jobs, to the log file for easy monitoring and analysis.
