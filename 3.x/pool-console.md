---
title: Pool Console
description: Doppar Pool Console page
meta:
  - name: keywords
    content: Pool Console
---

## Pool Console
### Introduction
Pool Console is Doppar's built-in command-line interface (CLI), designed to streamline development and automate common tasks. Located at the root of your application as the pool script, it provides a wide range of helpful commands to assist you during the application development process.

To view a full list of available Pool commands, simply run:
```bash
php pool list
```
This command will display all registered commands grouped by functionality, making it easy to discover what’s available as you build with Doppar.

## Create Console Commands
Beyond the default commands that come with Doppar, you have the flexibility to define your own custom commands. These are generally located within the `app/Schedule/Commands` directory of your project.

To generate a new command, use the `make:command` command provided by Doppar. This will scaffold a new command class inside the `app/Schedule/Commands` folder. If the directory doesn't already exist, don't worry—Doppar will automatically create it the first time you execute the command:

```bash
php pool make:command InvoiceProcessCommand
```
Once your command has been generated, be sure to set meaningful values for the `name` and `description` properties within the class. These values help identify and describe your command when it appears in the list view. The `name` property also serves to define how the command should be invoked, including any expected input parameters.

The `handle` method is where the core logic of your command should reside. This method is executed when your command is run, and it's the ideal place to implement the functionality specific to your scheduling task.

> Let's take a look at an example command. Note that we are able to request any dependencies we need via the command's contructor method

```php
<?php

namespace App\Schedule\Commands;

use Phaseolies\Console\Schedule\Command;

class InvoiceProcessCommand extends Command
{
    /**
     * The name of the console command.
     *
     * @var string
     */
    protected $name = 'doppar:invoice-process-command';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Command description';

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle(): int
    {
        return Command::SUCCESS
    }
}
```

This is how doppar provides a structured way to define a console command. Each command is represented by a PHP class and must define a mandatory command name.

## Dependency Injection
Doppar supports dependency injection in console commands, allowing services and other classes to be automatically resolved from the service container. Dependencies can be injected via the `constructor` or directly into the `handle()` method.

### Constructor Injection
You may inject dependencies through the command constructor. This is useful when the dependency is required throughout the lifecycle of the command.
```php
use App\Services\InvoiceService;
use Phaseolies\Console\Schedule\Command;

class InvoiceProcessCommand extends Command
{
    /**
     * Create a new command instance.
     *
     * @param InvoiceService $invoiceService
     */
    public function __construct(
        private InvoiceService $invoiceService
    ) {
        parent::__construct();
    }

   /**
     * Execute the console command
     *
     * @return int
     */
    public function handle(): int
    {
        $this->invoiceService->process();
    }
}
```

Dependencies are resolved automatically by the container.

> Always call `parent::__construct()` when overriding the constructor.

### Method Injection (Handle Injection)
Doppar also allows dependencies to be injected directly into the `handle()` method. This is useful for command-specific dependencies that are only needed during execution.
```php
use App\Models\User;
use App\Repositories\PaymentRepository;
use App\Http\Controllers\ProductController;

public function handle(
    User $user,
    PaymentRepository $paymentRepository,
    ProductController $productController
): int {
    // All dependencies are automatically resolved
}
```

To promote better code reuse and maintainability, it's recommended to keep your console commands minimal by offloading the core logic to dedicated application services. In the example provided, instead of embedding the full processing logic inside the command, we delegate the actual work—such as handling pending invoices—to the injected `InvoiceService` class.

This approach helps keep your command focused on orchestration, while your domain logic remains testable and reusable across different parts of the application.

> You can freely combine both approaches in the same command:

## Binding Services to Abstraction
Doppar console commands support PHP 8 attribute-based binding using the `#[Bind]` attribute. This allows you to bind an abstraction (interface) to a concrete implementation directly within the `handle()` method.

This approach keeps your commands explicit, reduces container boilerplate, and provides the same developer experience available in Doppar HTTP controllers.

Apply the `#[Bind]` attribute to a parameter in your command’s `handle()` method to specify which concrete class should be resolved for an interface.
```php
use App\Repositories\PaymentRepository;
use App\Repositories\PaymentRepositoryInterface;

public function handle(
    #[Bind(PaymentRepository::class)] PaymentRepositoryInterface $paymentRepository
): int {
    // All dependencies are automatically resolved
}
```

Use #`[Bind]` when you need explicit, localized interface resolution. or widely shared bindings, continue using your application's global container configuration.

> The `#[Bind]` attribute is not supported for constructor injection. Use it only for handle method injection.

## Exit Codes
In Doppar, if your command's handle method completes without returning a value, it will automatically exit with a code of `0`, signaling that the operation was `successful`. However, you can explicitly control the exit status by returning an integer from the handle method. For example, returning 1 typically indicates that something went wrong during execution:

You can generate error like this
```php
$this->error('Something went wrong.');
```

There is also two methods you can use:
```php
$this->comment('Something went wrong.');
$this->info('Something went wrong');
```

## Console Output Helpers
Doppar console command provides some helepr method. These helper methods make it easier to display formatted messages, write output lines, and measure execution time in your console commands.
```php
$this->line('Hello World'); 
$this->line('Important!', 'fg=red;options=bold');
```
Console output:
```php
Hello World
Important!   // in bold red
```

Writes one or more blank lines.
```php
$this->newLine(2);
```
Adds two empty lines to the console.

Displays a green `SUCCESS` label followed by the message
```php
$this->displaySuccess('Message goes here');
```

Displays a red `ERROR` and `WARNING` label followed by the message.
```php
$this->displayError('Port must be an integer');
$this->displayWarning('Disk space is running low');
```

## Showing Execution Time
Sometimes, especially during development or when optimizing performance, it’s useful to know how long a command or specific operation takes to run. Displaying execution time can help identify slow processes, measure improvements after optimizations, and debug performance bottlenecks.

With the `displayExecutionTime()` helper, you can easily show the elapsed time in both seconds and microseconds, giving you a precise measurement right in your console output.

Shows the elapsed execution time in seconds and microseconds.
```php
$start = microtime(true);
// ... your code ...
$this->displayExecutionTime($start);
```

Executes a callback, automatically measuring execution time and catching RuntimeExceptions.
If an exception is caught, it displays the error and still shows timing.
```php
return $this->executeWithTiming(function () {
    $this->displayInfo('Doing work...');
    sleep(1);
    $this->displaySuccess('Work complete!');
    return 0;
});
```

A shorthand wrapper for `executeWithTiming()` that optionally shows a success message after completion.
```php
return $this->withTiming(function () {
    sleep(2);
}, 'Process finished successfully.');
```
Output
```php
 SUCCESS  Process finished successfully.

⏱ Time: 2.0001s (2000123 μs)
```

## Command Arguments
In Doppar, you can define input parameters for your commands directly within the `$name` property using curly braces. These parameters allow users to pass dynamic values when running the command. For example, the snippet below registers a required argument called status:
```php
/**
 * The name of the console command.
 *
 * @var string
 */
protected $name = 'doppar:invoice-process-command {status}';
```
Here, status must be provided when the command is executed. This allows you to tailor the command's behavior based on user input at runtime, enabling more flexible and interactive console operations.

You can also make arguments optional by adding `?` after argument text:
```php
// Optional argument...
'doppar:invoice-process-command {status?}'
```

## Command Options
In Doppar, options provide additional flexibility by allowing users to supply optional input values when executing a console command. You can define these options within the $name property using curly braces and the `--` prefix. Here's an example that includes both a required argument and an optional flag:
```php
protected $name = 'doppar:invoice-process-command {status} {--process_per_minute=}';
```
In this case:
- `status` is a required argument.
- `--process_per_minute` is an optional parameter that expects a value.

To run this command with both values, you would execute:
```bash
php pool doppar:invoice-process-command pending --process_per_minute=10
```
This setup lets you fine-tune how the command behaves without altering its core logic—ideal for adding customizable behavior like batch size, verbosity, or filters.

If you call like this 
```bash
php pool doppar:invoice-process-command pending --process_per_minute
```
> In this example, the --process_per_minute switch may be specified when calling the Pool command. If the `--process_per_minute` switch is passed, the value of the option will be `true`. Otherwise, the value will be `false`:

## Command Input Descriptions
In Doppar, you can attach helpful descriptions to your command's arguments and options by appending a colon `(:)` followed by the description. This improves clarity when viewing the command's help output. If your command signature gets lengthy, you can break it into multiple lines for better readability.
```php
/**
 * The name of the console command.
 *
 * @var string
 */
protected $name = 'doppar:invoice-process-command
                   {status: The status of the invoice}
                   {--process_per_minute: How many invoice should be processed per minute}';
```

## Command I/O
#### Retrieving Input
During command execution in Doppar, you can retrieve user-supplied input values using the `argument` and `option` methods. These methods allow you to easily work with the data passed to your command. If a specified key isn't provided by the user, these methods will simply return null.

Here’s a sample usage inside the handle method:
```php
/**
 * Execute the console command.
 */
public function handle(): int
{
    $status = $this->argument('status');
}
```
Options may be retrieved just as easily as arguments using the option method.
```php
$processPerMinute = $this->option('process_per_minute')
```

## Interactive Input
In addition to retrieving arguments and options, Doppar commands can interactively prompt users for input at runtime. This is particularly useful for commands that require confirmation, credentials, or additional configuration.

Doppar provides several helper methods for interactive input. For example, If the user presses Enter without typing anything, the optional default value will be returned.
```php
$name = $this->ask('What is your name?', 'Anonymous');

$this->info("Hello, {$name}!");
```

See full basic example with `confirm`, `secret` and `ask`
```php
// Ask for simple input
$name = $this->ask('What is your name?', 'Anonymous');
$this->info("Hello, {$name}!");

// Ask for hidden input (e.g., password)
$password = $this->secret('Enter your password:');
$this->info("Password received with " . strlen($password) . " characters");

// Confirm before proceeding
$shouldContinue = $this->confirm('Do you wish to continue?', true);

if (!$shouldContinue) {
    $this->error('Operation cancelled!');
    return 1;
}

$this->info('Continuing operation...');
```

Example console output
```bash
What is your name? [Anonymous]: Alice
Hello, Alice!

Enter your password:
Password received with 10 characters

Do you wish to continue? [Y/n]:
Continuing operation...
```

## Choice-Based Input
In addition to simple questions, Doppar allows you to prompt users to choose from a list of options — either a single option or multiple options.
This is useful when you want to guide users toward predefined configurations or settings.

Example using `choice()` and `multipleChoice()`
```php
// Single-choice question
$theme = $this->choice(
    'Select a theme',
    ['light', 'dark', 'auto'],
    0 // Default choice by index (0 = 'light')
);
$this->info("Selected theme: {$theme}");

// Multiple-choice question
$features = $this->multipleChoice(
    'Select features to install',
    ['api', 'admin', 'web', 'mobile', 'docs']
    // No default array — leave it out or pass null
);
$this->info("Selected features: " . implode(', ', $features));
```

Example console output
```bash
Select a theme:
  [0] light
  [1] dark
  [2] auto
> 1
Selected theme: dark

Select features to install:
  [0] api
  [1] admin
  [2] web
  [3] mobile
  [4] docs
> 0,2,4
Selected features: api, web, docs
```

## Progress Bars
When performing long-running operations, Doppar provides an easy way to display a progress bar to give users visual feedback on the command’s progress.

The progress bar can be advanced manually as your task runs, and finished once all items are processed.
```php
$this->info('Processing items...');

// Create a progress bar with 50 steps
$progressBar = $this->createProgressBar(50);

// Simulate work
for ($i = 0; $i < 50; $i++) {
    usleep(100000); // 0.1 second delay to simulate processing
    $progressBar->advance();
}

// Finish the progress bar
$progressBar->finish();
$this->newLine();

$this->info('All items processed successfully!');
```

Example console output
```bash
Processing items...
50/50 [██████████████████████████████████████████████████] 100%
All items processed successfully!
```

## Display Table
Doppar allows you to easily format and display tabular data in the console using the `createTable()` helper.
This is especially useful for listing users, tasks, configurations, or any data that fits a grid layout.

Example of rendering a table
```php
// Create a new table instance
$table = $this->createTable();

// Define headers
$table->setHeaders(['ID', 'Name', 'Email']);

// Add rows
$table->addRow([1, 'John Doe', 'john@example.com']);
$table->addRow([2, 'Jane Smith', 'jane@example.com']);

// Render the table in the console
$table->render();
```

Example console output
```bash
+----+------------+-------------------+
| ID | Name       | Email             |
+----+------------+-------------------+
| 1  | John Doe   | john@example.com  |
| 2  | Jane Smith | jane@example.com  |
+----+------------+-------------------+
```
## Register Commands
There’s no need to manually register your custom commands in Doppar. Once you've created a command using the `make:command` utility, Doppar automatically detects and loads it without requiring additional configuration. This keeps your setup lean and allows you to focus on building functionality rather than wiring it up.

When Pool boots, all the commands in your application will be resolved by the service container and registered with Pool.

## Invoking Commands via Code
In some scenarios, you might want to run a Doppar console command from within your application — such as inside a route, controller, or service. This can be accomplished using the call method on the Pool facade. You can pass the command name (or class name) as the first argument and an array of parameters as the second. The method will return the command’s exit code:
```php
use Phaseolies\Support\Facades\Pool;

Pool::call('doppar:invoice-process-command pending --process_per_minute=100');
```
This approach is useful for triggering background tasks or deferred jobs directly from user interactions or other runtime conditions.

When call method triggered, it returns metadata about the execution, including the process ID, command signature, status, and start time.
```json
{
  "pid": 29886,
  "start_time": 1748450010
}
```
This makes it easy to track the execution state of any command initiated programmatically.

