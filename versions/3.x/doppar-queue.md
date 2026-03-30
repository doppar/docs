---
title: Queue
description: Doppar queue manager
meta:
  - name: keywords
    content: queue manager
---

## Queue
### Introduction
A queue is a background job processing system that allows tasks to run asynchronously instead of blocking your main application flow. It improves performance, distributes workload efficiently, and ensures time-consuming tasks are handled reliably in the background. Queues are essential for scaling modern applications and maintaining smooth user experiences.

Doppar queue system offers robust features including multiple queue support for organizing jobs by priority or category, automatic retry logic with configurable attempts and delays, and comprehensive failed job tracking for easier debugging. It supports delayed execution for scheduling jobs in the future, graceful shutdown handling to safely stop workers, and built-in memory management that automatically restarts workers when limits are exceeded.

 - [Features](#features)
 - [Installation](#installation)
 - [Quick Start](#quick-start)
 - [Job Chaining](#job-chaining)
 - [Job Lifecycle](#job-lifecycle)
 - [Queue Commands](#queue-commands)
 - [Production Setup](#production-setup)

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
| **Dynamic Queue Assignment**          | Assign jobs to queues dynamically at runtime.                                                 |
| **Worker Options**                    | Configure queue workers with `--queue`, `--sleep`, `--memory`, `--timeout`, and `--limit`.    |

## Installation
You may install Doppar Queue via the composer require command:
```bash
composer require doppar/queue
```

### Register Provider
Next, register the Queue service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the QueueServiceProvider to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Queue\QueueServiceProvider::class,
],
```

This step ensures that Doppar knows about Queue and can load its functionality when the application boots.

### Publish Configuration
Now we need to publish the configuration files by running this pool command.
```php
php pool vendor:publish --provider="Doppar\Queue\QueueServiceProvider"
```

Now run migrate command to migrate queue related tables
```php
php pool migrate
```

This creates two tables:
- `queue_jobs` - Stores pending and processing jobs
- `failed_jobs` - Stores failed jobs for debugging

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

Now run the queue worker like this way
```bash
php pool queue:run --queue=email
```

Uncomment and adjust these values as needed to control the job's queueing behavior.

### Job Properties
Each job in Doppar queue can be configured with the following properties:
| Property      | Type   | Default     | Description                                   |
| ------------- | ------ | ----------- | --------------------------------------------- |
| `$tries`      | int    | 3           | Maximum number of retry attempts.             |
| `$retryAfter` | int    | 60          | Seconds to wait before retrying a failed job. |
| `$queueName`  | string | `'default'` | The queue to which the job will be pushed.    |
| `$jobDelay`   | int    | 0           | Delay in seconds before executing the job.    |
| `$timeout`    | int(second)    | null        | Allows each job to define its own maximum execution time.|

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

- `--queue` – Process jobs from a specific queue (e.g., emails).
- `--sleep` – Number of seconds to wait between polling when no jobs are available.
- `--memory` – Maximum memory in MB before the worker automatically restarts.
- `--timeout` – Maximum execution time in seconds before the worker stops.
- `--limit` — Maximum number of jobs the worker will process before exiting

This allows you to run workers efficiently in production and tailor them to different job types or workloads.

### Available Options
These options let you customize how the worker polls the queue, manages resources, and handles long-running tasks efficiently.
| Option      | Description                             | Default   |
| ----------- | --------------------------------------- | --------- |
| `--queue`   | Name of the queue to process            | `default` |
| `--sleep`   | Seconds to wait when the queue is empty | `3`       |
| `--memory`  | Maximum memory in MB before restarting  | `128`     |
| `--timeout` | Maximum execution time in seconds       | `3600`    |
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
│                   QUEUE STORAGE - Database                      │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │ queue_jobs:                                              │   │
│  │ • id, queue, payload, attempts, reserved_at,             │   │
│  │   available_at, created_at                               │   │
│  └──────────────────────────────────────────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     WORKER PROCESSING                           │
│  ┌────────────────────────────────────────────────────────┐     │
│  │ • Pop job from queue                                   │     │
│  │ • Reserve job - mark as processing                     │     │
│  │ • Increment attempts counter                           │     │
│  │ • Check others configuration like delayFor             │     │
│  │ • Execute with timeout protection                      │     │
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
         │    queue_jobs    │  │ Logic            │
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
                  │ │ Back to      │      │ • Move to        │
                  │ │ Queue with   │      │   failed_jobs    │
                  │ │ Delay        │      │ • Delete from    │
                  │ └──────────────┘      │   queue_jobs     │
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

### Dynamic Queue Assignment
You can assign a specific queue to a job by setting the `$queueName` property in the job class. This is useful for categorizing jobs by type or priority:
```php
class SendInvoiceEmail extends Job
{
    /**
     * The name of the queue the job should be sent to.
     *
     * @var string
     */
    public $queueName = 'billing';

    // Job logic here...
}
```
When dispatched, this job will automatically be pushed to the `billing` queue.

For jobs that need dynamic routing based on runtime conditions, you can use the `onQueue()` method:
```php
(new ProcessOrder($order))
    ->onQueue($order->priority === 'high' ? 'urgent' : 'standard')
    ->dispatch();
```

Here, the job will be sent to the `urgent` queue if the order is high priority, otherwise it goes to the `standard` queue. This allows flexible queue management depending on job-specific criteria.

## Queue Commands
Doppar provides a set of CLI commands to manage and monitor your queues efficiently. These commands allow you to run workers, inspect failed jobs, retry or flush jobs, and view queue statistics.

### Process Jobs
Processes jobs in the queue one by one. You can specify options such as queue name, sleep interval, memory limit, and execution timeout.
```bash
# Run the default queue
php pool queue:run

# Run a specific queue with custom options
php pool queue:run --queue=emails --sleep=3 --memory=256 --timeout=3600
```

### List Failed Jobs
Lists all jobs that have failed and been recorded in the failed jobs table. Useful for monitoring and debugging job issues.
```bash
# List all failed jobs
php pool queue:failed
```

### Delete Failed Jobs
Deletes failed jobs from the failed jobs table. You can delete a specific job by ID or all failed jobs if no ID is provided.
```bash
# Delete a specific failed job
php pool queue:flush 5

# Delete all failed jobs
php pool queue:flush
```

### Retry Failed Jobs
Retries failed jobs either by ID or all failed jobs if no ID is specified. Jobs will be pushed back to the queue for reprocessing.
```bash
# Retry a specific failed job
php pool queue:retry 3

# Retry all failed jobs
php pool queue:retry
```

### Monitor Queue Statistics
Displays statistics about the queues, including pending jobs, failed jobs, and active workers. Useful for monitoring queue health and performance.
```bash
# Monitor all queues
php pool queue:monitor
```

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

This will append queue statistics, such as pending and failed jobs, to the log file for easy monitoring and analysis.
