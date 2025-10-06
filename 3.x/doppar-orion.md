---
title: Processes
description: Doppar process manager
meta:
  - name: keywords
    content: Process manager
---

## Processes
### Introduction
The Doppar Framework provides a powerful and expressive abstraction over the [Symfony Process Component](https://symfony.com/doc/current/components/process.html) via the Process facade. This feature enables developers to run and manage system-level commands and scripts with ease—whether synchronously, asynchronously, or in concurrent pools.

### Key Features
- Simple command execution: Run shell commands using a clean, fluent interface.
- Output handling: Stream, capture, or suppress command output based on your needs.
- Asynchronous processing: Execute long-running commands without blocking the main thread.
- Timeout management: Define command execution time limits and handle timeouts gracefully.
- Process pooling: Launch and manage multiple processes concurrently with customizable concurrency limits.
- Command pipelines: Chain multiple commands together with easy piping.

## Installation
To get started with the Doppar Orion Process component, simply install it via Composer:
```bash
composer require doppar/orion
```

Doppar does not support package auto-discovery, you need to manually register the service provider to enable the Process facade.

In your `config/app.php` (or equivalent configuration file), add the following to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Orion\OrionServiceProvider::class,
],
```
Once installed and registered, the Process facade will be available for use, giving you access to a rich set of tools for managing shell commands, pipelines, asynchronous processes, and concurrent execution within your Doppar application.

## Execute Process
To invoke a process, you may use the `ping` and `execute` methods offered by the Process facade. The execute method will run the given command and wait for it to finish executing before returning a result instance. Let's take a look at how to invoke a basic, synchronous process and inspect its result:
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::ping('ls -la')->execute();

return $result->getOutput();
```
You may also access the error output of the process using the `getError` method:
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::ping('ls -la')->execute();

return $result->getError();
```

## Command Injection Protection
Doppar Orion takes command injection seriously and includes built-in safeguards to prevent dangerous or malformed command execution.
#### Example: Unsafe Command
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::ping('rm -rf /; echo "hacked"')->execute();
```

> [!CAUTION]
> InvalidArgumentException <br>
> Potential command injection detected: rm -rf /; echo "hacked"

## Handling Process Output
In some cases, you may wish to interact with the output of a process in real time—such as streaming logs to the browser or logging incremental output to a file. You can accomplish this by using the withOutputHandler method before executing the process.

This method accepts a callback that receives both the output type (stdout or stderr) and the output content.
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::ping('ls -la')
    ->withOutputHandler(function (string $type, string $output) {
        echo $output;
    })
    ->execute();

return $result;
```
The output handler will be called for every chunk of output as it becomes available, making it ideal for long-running or verbose commands.

> ℹ️ The $type parameter will be either 'stdout' or 'stderr', allowing you to distinguish between normal and error output.

This provides you with fine-grained control over how process output is handled in real time—without having to wait for the process to finish.

## Silent Execution
If your process generates a large amount of output that you do not need to capture or process, you can conserve memory by disabling output retrieval entirely. Doppar provides a pingSilently method for this purpose.

This is especially useful when running background jobs, scripts, or imports where output is irrelevant.
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::pingSilently()->execute('bash importer.sh');

return $result;
```
By default, Doppar captures both stdout and stderr. However, when using `pingSilently`, no output will be stored in memory or returned—reducing overhead for high-volume or long-running processes.

> ⚠️ Since output is not retained, calls to getOutput() or getError() on the result will return
`LogicException
Output has been disabled..`

Use this option when you want performance over verbosity.

## Command Pipelines
Doppar allows you to run multiple commands as a pipeline, passing the output of one command directly into the next—just like using pipes (|) in the terminal. To accomplish this, use the pipeline method followed by one or more add calls to define the sequence.
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::pipeline()
    ->add('cat server.php')
    ->add('grep -i "doppar"')
    ->execute();

if ($result->wasSuccessful()) {
    // The pipeline executed successfully.
}
```
Each add method represents a step in the pipeline. The output of the previous command is passed as input to the next. This allows you to chain together complex shell behavior in a safe and structured way.

> ✅ Unlike raw shell piping (|), Doppar securely manages the execution of each step to prevent injection and isolate command logic.

You can check if the entire pipeline was successful using the wasSuccessful() method.

## Asynchronous Processes
If you need to execute a process without blocking the main thread—such as handling large files, long-running scripts, or background tasks—you can run it asynchronously using the `asAsync` method.

This allows your application to continue performing other tasks while the process runs in the background.
```php
use Doppar\Orion\Support\Facades\Process;

$process = Process::ping('bash import.sh')
    ->withTimeout(120)
    ->asAsync();
```

Once started, you can monitor the process in a non-blocking loop:
```php
while ($process->isRunning()) {
    // Optionally check incremental output or perform other work
    // $output = $process->getLatestOutput();
    // $error = $process->getLatestError();
}
```
Finally, you can wait for the process to complete and retrieve the result:
```php
$result = $process->waitForCompletion();

// Inspect result if needed
dd($result);
```

## Async Process with Output Monitoring
When running a process asynchronously, you may want to monitor its output and errors in real time. Doppar supports this by letting you retrieve the latest output and error streams during execution.
```php
use Doppar\Orion\Support\Facades\Process;

$process = Process::ping('bash import.sh')
    ->withTimeout(120)
    ->asAsync();

while ($process->isRunning()) {
    echo $process->getLatestOutput();
    echo $process->getLatestError();

    sleep(1); // Throttle the loop to avoid high CPU usage
}
```
#### How It Works
`getLatestOutput()` returns any new standard output since the last call. `getLatestError()` returns any new error output. The loop keeps polling while the process runs, allowing your app to react to output incrementally.

> ⚠️ Remember to add a short delay (like sleep(1)) to avoid excessive CPU usage during the polling loop.

This pattern is perfect for tailing logs, streaming command progress, or interactive command execution.

## Waiting for a Condition
Sometimes you need to keep an asynchronous process running until a specific output or condition is met. Doppar offers the until method, which accepts a callback to check the output continuously and stop when your condition is fulfilled.
```php
use Doppar\Orion\Support\Facades\Process;

$process = Process::ping('bash import.sh')->asAsync();

$process->until(function (string $type, string $output) {
    return $output === 'Ready...';
});
```
The callback receives the output type (stdout or stderr) and the latest output chunk. Returning true from the callback signals Doppar to stop waiting. The process continues running asynchronously until the condition is satisfied.

## Timeout Verification
When running asynchronous processes, it’s important to enforce time limits to avoid runaway commands. Doppar provides the verifyTimeout() method, which you can call periodically to check if the process has exceeded its timeout and throw an exception if so.
```php
use Doppar\Orion\Support\Facades\Process;

$process = Process::ping('sleep 10') // This command runs for 10 seconds
    ->withTimeout(2)                  // Set timeout to 2 seconds
    ->asAsync();

try {
    while ($process->isRunning()) {
        $process->verifyTimeout(); // Throws if the timeout is exceeded

        // Perform other work or wait before next check
        sleep(1);
    }

    $result = $process->waitForCompletion();

    dd($result);
} catch (\Exception $e) {
    // Handle timeout exception
    dd("Process timed out: " . $e->getMessage());
}
```
#### How It Works
- withTimeout(seconds) defines the maximum allowed execution time.
- Calling verifyTimeout() checks if the timeout has been exceeded.
- If the process runs longer than allowed, verifyTimeout() throws an exception which you can catch and handle gracefully.

> This pattern helps ensure your app remains responsive and avoids stuck or long-running processes.

## Concurrent Process
Doppar allows you to manage multiple processes concurrently using process pools. This lets you run several commands in parallel while controlling concurrency and managing results collectively.
```php
use Doppar\Orion\Support\Facades\Process;

$pool = Process::pool()
    ->inDirectory(__DIR__)
    ->add('bash import-1.sh')
    ->add('bash import-2.sh')
    ->add('bash import-3.sh')
    ->start();

while (!empty($pool->getRunningProcesses())) {
    // You can perform other tasks or monitor progress here
}

$results = $pool->waitForAll();

dd($results);
```
Process pools are ideal for batch jobs, parallel imports, or running multiple independent scripts simultaneously.

Doppar provides a simple method to run multiple commands concurrently and retrieve their results individually using the asConcurrently method.
```php
use Doppar\Orion\Support\Facades\Process;

[$first, $second, $third] = Process::asConcurrently([
    'ls -la',
    'ls -la ' . database_path(),
    'ls -la ' . storage_path(),
], __DIR__);

echo $first->getOutput();
```
#### How It Works
- Pass an array of commands to asConcurrently.
- Optionally specify the working directory for all commands.
- The method returns an array of result objects corresponding to each command.
- You can then access output and status of each command independently.

## Concurrency Control and Output Handling
You can manage multiple concurrent processes in a pool while limiting how many run at the same time. Additionally, you can attach an output handler to react whenever a process finishes.
```php
use Doppar\Orion\Support\Facades\Process;

$pool = Process::pool()
    ->withConcurrency(3) // Run up to 3 processes simultaneously
    ->inDirectory(_DIR__)
    ->withOutputHandler(function ($result) {
        echo "Process completed with exit code: " . $result->getExitCode() . "\n";
    });

$pool->add('command1')
    ->add('command2')
    ->add('command3')
    ->add('command4');

$results = $pool->start()->waitForAll();

dd($results);
```
#### Key Features
- `withConcurrency(int)` controls how many processes run at once.
- `withOutputHandler(callable)` receives the result of each completed process.
- `add(string)` queues commands to run in the pool.
- `start()` begins processing all queued commands.
- `waitForAll()` blocks until all commands finish and returns their results.

This approach is ideal for batch jobs that benefit from parallelism but require resource control and real-time feedback.