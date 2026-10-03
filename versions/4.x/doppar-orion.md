---
title: Processes
description: Doppar process manager
meta:
  - name: keywords
    content: Process manager
---

## Processes
### Introduction
Doppar Processes provides a powerful and expressive abstraction for running and managing system-level commands and scripts from within your PHP application. Built on top of the Symfony Process Component, it gives you a fluent interface for executing commands, handling their output, managing timeouts, and controlling processes without having to work directly with the underlying process APIs.

Processes supports both synchronous and asynchronous execution, allowing you to choose whether your application should wait for a command to finish or continue working while a long-running process executes. Output can be captured, streamed in real time, or disabled entirely when it is not needed, giving you control over both process behavior and resource usage.

For applications that need to execute multiple commands, Doppar provides command pipelines and concurrent process execution. Pipelines allow the output of one command to flow into the next, while process pools make it possible to run multiple independent commands in parallel with configurable concurrency limits. Asynchronous processes can also be monitored while running, allowing applications to inspect incremental output, detect conditions, and enforce execution timeouts.

Doppar Processes also includes built-in safeguards for command execution, including protection against potentially unsafe command injection patterns. Together with configurable output handling, timeout management, process pooling, and fluent command configuration, these features provide a structured way to integrate system-level operations into Doppar applications.

## Installation
To get started with the Doppar Orion Process component, simply install it via Composer:
```bash
composer require doppar/orion
```

Doppar does not support package auto-discovery, you need to manually register the launcher to enable the Process facade.

In your `runtime/config/app.php` (or equivalent configuration file), add the following to the launchers array:
```php
'launchers' => [
    // Other launchers
    \Doppar\Orion\OrionLauncher::class,
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

## Writing Commands
A command can be a string or an array of arguments.

A string is split into the program and its arguments the way a shell reads quotes, but it is **never run through a shell**. Words are separated by whitespace, and quotes keep what is inside them together:
```php
Process::ping('git commit -m "fix the bug"')->execute();     // ['git', 'commit', '-m', 'fix the bug']
Process::ping("printf '%s|%s' 'a b' c")->execute();           // ['printf', '%s|%s', 'a b', 'c']
```

Because there is no shell, nothing in the command is interpreted. `$HOME`, `*`, `~`, `;`, `|` and `>` are ordinary characters of an argument:
```php
Process::ping('echo $HOME')->execute()->getOutput();   // "$HOME\n", the variable is not expanded
```

Use an array when an argument contains spaces or characters you would rather not quote. Each element is passed to the program exactly as it is:
```php
Process::ping(['git', 'commit', '-m', 'fix the bug'])->execute();
```

The rules for strings are:
- Whitespace separates arguments, and several spaces in a row do not create empty arguments.
- Single quotes keep everything inside them as it is.
- Double quotes keep everything as it is, except `\"` and `\\`.
- Outside quotes, a backslash escapes a space, a quote or another backslash. Anywhere else it is a normal character, so Windows paths such as `C:\tools\php` work.
- `""` is an empty argument.

A command with a quote that is never closed throws an `InvalidArgumentException` before anything runs.

> If you need pipes, redirects or variables, run the shell yourself and pass your script as one argument: `['sh', '-c', 'ls | wc -l']`. Never build that script from user input.

## Inspecting the Result
The result returned by `execute()` and `waitForCompletion()` tells you what happened:
```php
$result = Process::ping('ls -la')->execute();

$result->getOutput();       // the standard output
$result->getError();        // the error output
$result->getExitCode();     // 0 on success, -1 when the process has no exit code (for example it is still running)
$result->wasSuccessful();   // true when the exit code is 0
$result->failed();          // true when it ran and did not succeed
$result->timedOut();        // true when it was stopped for exceeding its timeout
$result->getSignal();       // the signal that ended it, or null
$result->getCommandLine();  // the command that ran
$result->getDuration();     // how long it ran, in seconds
$result->getExitCodes();    // [0]; for a pipeline, the exit code of each command
```

Read the output as lines or as JSON:
```php
$result = Process::ping('composer show --format=json')->execute();

$result->json();                          // the decoded output
$result->json('installed.0.name');        // one value, dot notation allowed
$result->json('missing.key', 'default');  // a default when it is not there

Process::ping('git status --short')->execute()->lines();   // ['M  src/a.php', '?? b.php']
```

`json()` throws a `JsonException` when the output is not valid JSON, and `lines()` returns an empty array when there is no output.

## Throwing on Failure
A failed command does not throw by default. Call `throw()` to turn a failure into an exception, so it cannot be ignored:
```php
Process::ping('composer install')->execute()->throw();
```

`throw()` returns the result when the command succeeded, so you can keep chaining:
```php
$output = Process::ping('git rev-parse HEAD')->execute()->throw()->getOutput();
```

For a single process it throws Symfony's `ProcessFailedException`, whose message contains the command, the exit code and the output. For a pipeline it throws a `RuntimeException` with the same information. `throwIf()` only throws when a condition holds:
```php
$result->throwIf(fn ($result) => $result->getExitCode() !== 2);   // exit code 2 is allowed
```

## Command Injection Protection
Doppar Orion takes command injection seriously and includes built-in safeguards to prevent dangerous or malformed command execution.

Commands given to `ping`, to a pipeline's `add` and to a pool's `add` are checked. A command, or an argument of an array command, is rejected when it contains `;`, `|`, `&`, a backtick, `$(`, `${`, `<`, `>`, a line break or a NUL byte.

On top of that, none of these commands are run through a shell, so even a character that is not on the list cannot start another command.

> If an argument legitimately needs one of those characters, such as the PHP code in `php -r 'echo 1;'`, pass it as an array to `ProcessService::create([...])`. That entry point does not check the command, so only use it for commands you wrote yourself, never for user input.

#### Example: Unsafe Command
```php
use Doppar\Orion\Support\Facades\Process;

$result = Process::ping('rm -rf /; echo "hacked"')->execute();
```

> InvalidArgumentException Potential command injection detected: `rm -rf /; echo "hacked"`

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

> ℹ️ The $type parameter will be either `'out'` or `'err'`, allowing you to distinguish between normal and error output.

The handler also works with asynchronous processes. It receives the output while the process runs, and you can give `waitForCompletion()` a handler of its own for the output that arrives while you wait.

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
Doppar allows you to run multiple commands as a pipeline, passing the output of one command into the next, as the pipe (|) does in a terminal. To accomplish this, use the pipeline method followed by one or more add calls to define the sequence.
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
Each add method represents a step in the pipeline. The output of the previous command is passed as input to the next. Commands are read like any other command (see [`Writing Commands`](#writing-commands)): quotes work, and there is no shell, so nothing in a command is expanded or interpreted.

> ✅ Because each step is run as its own process and never through a shell, a command cannot start another one, whatever it contains.

The result describes the whole pipeline:
```php
$result = Process::pipeline()
    ->add('ls /var/www /nonexistent')
    ->add('sort')
    ->execute();

$result->getOutput();      // the output of the last command
$result->getError();       // the error output of every command, in order
$result->getExitCode();    // 0, or the exit code of the last command that failed
$result->getExitCodes();   // one exit code per command: [2, 0]
$result->getCommandLine(); // 'ls /var/www /nonexistent | sort'
$result->getDuration();    // seconds
```

A failure early in the pipeline is not hidden by a later command that succeeds: the exit code is the one of the last command that failed, like a shell with `set -o pipefail`. Call `$result->throw()` to turn a failure into an exception.

Options apply to every command in the pipeline:
```php
Process::pipeline()
    ->inDirectory(base_path())
    ->withEnvironment(['LANG' => 'C'])
    ->withTimeout(30)      // seconds for each command; no timeout by default
    ->add('cat storage/logs/app.log')
    ->add('grep ERROR')
    ->execute();
```

A command that runs longer than its timeout is stopped and a `Symfony\Component\Process\Exception\ProcessTimedOutException` is thrown.

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

`waitForCompletion()` enforces the timeout. If you only poll `isRunning()` in a loop, nothing stops the process when its timeout passes, so call `verifyTimeout()` in the loop (see [`Timeout Verification`](#timeout-verification)).

### Stopping a Process
An asynchronous process can be stopped or signalled while it runs:
```php
$process = Process::ping('bash import.sh')->asAsync();

$process->getPid();            // the process id, or null once it has ended
$process->signal(15);          // send a signal (15 is SIGTERM)
$process->stop(10);            // ask it to end, and kill it if it is still running after 10 seconds
```

`stop()` returns the exit code. `isRunning()` and `getPid()` on a process that was never started throw a `LogicException` that says to call `execute()` or `asAsync()` first, and the same goes for the other methods that need a started process.

To look at an asynchronous process without waiting for it, use `result()`. Its exit code is `-1` until the process has finished:
```php
$result = $process->result();
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

## Timeouts
A process is stopped when it runs longer than 60 seconds, and so is each process in a pool. A pipeline has no limit unless you set one. Change the limit with `withTimeout()`, which accepts whole or fractional seconds:
```php
Process::ping('bash import.sh')->withTimeout(300)->execute();
Process::ping('ping-service')->withTimeout(0.5)->execute();
```

Remove the limit when a command is meant to run for as long as it needs:
```php
Process::ping('bash import.sh')->withoutTimeout()->execute();
```
`withTimeout(null)` and `withTimeout(0)` do the same.

An idle timeout stops a process that produces no output for too long, however long it has been running:
```php
Process::ping('bash import.sh')
    ->withTimeout(600)
    ->withIdleTimeout(30)
    ->execute();
```

When a timeout is exceeded, a `Symfony\Component\Process\Exception\ProcessTimedOutException` is thrown. It extends `RuntimeException`, so `catch (\Exception $e)` still catches it.

## Timeout Verification
When running asynchronous processes, it’s important to enforce time limits to avoid runaway commands. Polling `isRunning()` does not stop a process that is over its limit, so Doppar provides the verifyTimeout() method, which you can call periodically to check if the process has exceeded its timeout, stop it, and throw an exception if so.
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
- If the process runs longer than allowed, verifyTimeout() stops it and throws a `ProcessTimedOutException`, which you can catch and handle gracefully.

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
    'ls -la ' . schema_path(),
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
    ->inDirectory(__DIR__)
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
- `withOutputHandler(callable)` receives the result of each process once, as it finishes.
- `withTimeout(seconds)` limits how long each process may run.
- `withEnvironment(array)` and `inDirectory(string)` apply to every process. The working directory is optional.
- `add(string|array)` queues commands to run in the pool.
- `start()` begins processing all queued commands.
- `waitForAll()` blocks until all commands finish and returns their results, keyed by the order they were added.

This approach is ideal for batch jobs that benefit from parallelism but require resource control and real-time feedback.

#### Timeouts in a Pool
Each process in a pool may run for 60 seconds by default. A process that exceeds its timeout is stopped, its result reports `timedOut()` and `failed()`, and the output handler is called for it like for any other. The other processes carry on, and the slot it used is given to the next command, so one hung command cannot hold up the pool.
```php
$results = Process::pool()
    ->withTimeout(30)
    ->add('bash import-1.sh')
    ->add('bash import-2.sh')
    ->waitForAll();

foreach ($results as $result) {
    if ($result->timedOut()) {
        // this command took longer than 30 seconds and was stopped
    }
}
```

Every result also knows how long its command ran: `$result->getDuration()`.