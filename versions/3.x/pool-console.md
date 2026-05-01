---
title: Pool Console
description: Doppar Pool Console page
meta:
  - name: keywords
    content: Pool Console
---

## Pool Console
### Introduction
Pool Console is Doppar's built-in command-line interface (CLI), designed to streamline development and automate repetitive tasks. The entry point is the `pool` script located at the root of your project. It provides a wide range of built-in commands and lets you register your own custom commands to run from the terminal.

To see every available command grouped by category, run:
```bash
php pool list
```

## Create Console Commands
Beyond the built-in commands, you can define your own. Custom commands live inside the `app/Schedule/Commands` directory. If the directory does not yet exist, Doppar creates it automatically when you generate your first command.

Use the `make:command` generator to scaffold a new command class:
```bash
php pool make:command MailSendCommand
```

This creates `app/Schedule/Commands/MailSendCommand.php` with the boilerplate already filled in. Open the file and set a short, descriptive value for `$name` — this is the string you will type in the terminal to invoke your command. The `$description` is shown in `php pool list`.

The `handle()` method is where all command logic lives. It is called every time the command runs.
```php
<?php

namespace App\Schedule\Commands;

use Phaseolies\Console\Schedule\Command;

class MailSendCommand extends Command
{
    /**
     * The name of the console command.
     *
     * @var string
     */
    protected $name = 'mail:send';

    /**
     * The console command description.
     *
     * @var string
     */
    protected $description = 'Send pending emails to users';

    /**
     * Execute the console command.
     *
     * @return int
     */
    public function handle(): int
    {
        $this->info('Sending emails...');

        // your logic here

        $this->displaySuccess('All emails sent.');

        return Command::SUCCESS;
    }
}
```

## Running Your Command
Once a command class exists, you can run it from the terminal immediately — no additional registration is needed. Use the `$name` value you defined in the class:
```bash
php pool mail:send
```

Doppar discovers all commands in `app/Schedule/Commands` automatically at boot time. You will see your command listed in `php pool list` as soon as the file is saved.

## Auto-Registration
There is no manual registration step in Doppar. When Pool boots, it scans `app/Schedule/Commands`, resolves every command class through the service container, and makes them available to the CLI. This means you can create a command file and run it right away without touching any configuration file.

## Command Arguments
Arguments are positional values the user passes after the command name. You declare them inside `$name` using curly braces.

### Required Argument
```php
protected $name = 'mail:send {status}';
```
Running this command without the argument produces an error:
```bash
php pool mail:send pending
```
Here `pending` is bound to the `status` argument.

### Optional Argument
Add `?` after the argument name to make it optional:
```php
protected $name = 'mail:send {status?}';
```
When the argument is omitted, `$this->argument('status')` returns `null`.
```bash
# With argument
php pool mail:send pending

# Without argument (status will be null)
php pool mail:send
```

## Command Options
Options are named flags prefixed with `--`. Unlike arguments, options can appear in any order and are always optional unless your logic enforces them.

### Option with a Value
Declare an option that expects a value using `{--name=}`:
```php
protected $name = 'mail:send {status} {--limit=}';
```
Run the command by passing both:
```bash
php pool mail:send pending --limit=50
```
Inside `handle()`, retrieve the value:
```php
$limit = $this->option('limit'); // "50"
```

### Boolean Flag Option
Declare an option without `=` to create a true/false switch:
```php
protected $name = 'mail:send {status} {--force}';
```
```bash
# Flag present — $this->option('force') returns true
php pool mail:send pending --force

# Flag absent — $this->option('force') returns false
php pool mail:send pending
```

### Option with a Default Value
You can set a default value by placing it after the `=`:
```php
protected $name = 'mail:send {status} {--limit=100}';
```
If the user does not pass `--limit`, `$this->option('limit')` returns `"100"`.

### Combining Arguments and Options
You can mix multiple arguments and options freely:
```php
protected $name = 'mail:send {status} {--limit=100} {--force}';
```
```bash
php pool mail:send pending --limit=25 --force
```

## Command Input Descriptions
Attach a human-readable description to any argument or option by appending `:` followed by the description text. Doppar displays these descriptions in the command's `--help` output. For long signatures, break across lines for readability.
```php
protected $name = 'mail:send
                  {status : The delivery status to filter by (e.g. pending, failed)}
                  {--limit= : Maximum number of emails to send in one run}
                  {--force : Skip confirmation and send immediately}';
```

## Retrieving Input
Inside `handle()`, use `argument()` to read positional arguments and `option()` to read named options. If a key is not provided, both methods return `null`.
```php
public function handle(): int
{
    $status = $this->argument('status');   // "pending"
    $limit  = $this->option('limit');     // "25" or null
    $force  = $this->option('force');     // true or false

    $this->info("Sending {$status} emails (limit: {$limit})...");

    return Command::SUCCESS;
}
```

Call `argument()` or `option()` without a key to get all values as an array:
```php
$allArguments = $this->argument();
$allOptions   = $this->option();
```

## Dependency Injection
Doppar resolves command dependencies automatically through the service container. You can inject them via the constructor, the `handle()` method, or both.

### Constructor Injection
Use constructor injection when a dependency is needed throughout the command's lifecycle.
```php
<?php

namespace App\Schedule\Commands;

use App\Services\MailService;
use Phaseolies\Console\Schedule\Command;

class MailSendCommand extends Command
{
    protected $name = 'mail:send {status} {--limit=100}';
    protected $description = 'Send pending emails to users';

    /**
     * @param MailService $mailService
     */
    public function __construct(
        private MailService $mailService
    ) {
        parent::__construct();
    }

    /**
     * @return int
     */
    public function handle(): int
    {
        $status = $this->argument('status');
        $limit  = (int) $this->option('limit');

        $this->mailService->send($status, $limit);

        $this->displaySuccess('Done.');

        return Command::SUCCESS;
    }
}
```
Run it:
```bash
php pool mail:send pending --limit=50
```

> Always call `parent::__construct()` when you override the constructor.

### Handle Method Injection
For dependencies only needed during execution, inject them directly into `handle()`. The container resolves them automatically.
```php
use App\Services\MailService;
use App\Repositories\UserRepository;

public function handle(
    MailService $mailService,
    UserRepository $userRepository
): int {
    $users = $userRepository->getPending();
    $mailService->sendBulk($users);

    return Command::SUCCESS;
}
```

### Binding an Interface to a Concrete Class
Doppar supports PHP 8 attribute-based binding via `#[Bind]`. Apply it to a parameter in `handle()` to resolve an interface to a specific implementation, without touching your global container configuration.
```php
use App\Repositories\UserRepository;
use App\Repositories\UserRepositoryInterface;

public function handle(
    #[Bind(UserRepository::class)] UserRepositoryInterface $userRepository
): int {
    $users = $userRepository->getPending();

    return Command::SUCCESS;
}
```

> `#[Bind]` is only supported on `handle()` method parameters. It cannot be used in the constructor.

## Exit Codes
`handle()` must return an integer exit code. Doppar provides two named constants:

| Constant           | Value | Meaning               |
| ------------------ | ----- | --------------------- |
| `Command::SUCCESS` | `0`   | Completed successfully |
| `Command::FAILURE` | `1`   | Completed with errors  |

If `handle()` returns nothing, Doppar treats it as a success (`0`).
```php
public function handle(): int
{
    if (!$this->canRun()) {
        $this->displayError('Prerequisites not met.');
        return Command::FAILURE;
    }

    // do work

    return Command::SUCCESS;
}
```

## Console Output Helpers
Doppar provides a set of output helper methods to write formatted messages to the console.

### Basic Output
Write a plain line of text:
```php
$this->line('Processing user records...');
```

Write a styled line using Symfony Console color tags:
```php
$this->line('Warning!', 'fg=yellow;options=bold');
$this->line('Critical!', 'fg=red;options=bold');
```

Write one or more blank lines:
```php
$this->newLine();    // one blank line
$this->newLine(2);   // two blank lines
```

### Semantic Shortcuts
Use these when you want consistent, recognisable formatting:
```php
$this->info('Starting import job...');      // plain text
$this->comment('This may take a while.');   // muted / comment style
$this->error('Database connection failed.'); // red error text
```

### Labelled Output
These methods print a coloured label followed by your message, making status lines visually distinct in busy output:
```php
$this->displaySuccess('Import completed — 1,200 rows written.');
$this->displayError('Validation failed on row 42.');
$this->displayWarning('API rate limit is approaching.');
$this->displayInfo('Running in dry-run mode. No changes will be saved.');
```

Console output:
```bash
 SUCCESS  Import completed — 1,200 rows written.
 ERROR    Validation failed on row 42.
 WARNING  API rate limit is approaching.
         Running in dry-run mode. No changes will be saved.
```

## Measuring Execution Time
When optimising or profiling commands, it is helpful to measure how long a section of code takes. Doppar provides helpers that time your code and format the result automatically.

### Manual Timer
Capture a start time before your work and pass it to `displayExecutionTime()` when finished:
```php
public function handle(): int
{
    $start = microtime(true);

    // ... do work ...

    $this->displayExecutionTime($start);

    return Command::SUCCESS;
}
```
Output:
```bash
⏱ Time: 1.2340s (1234012 μs)
```

### Timed Callback with `executeWithTiming()`
Wrap your logic in a closure. Doppar measures it automatically, catches `RuntimeException`, and always prints the elapsed time — even on failure:
```php
public function handle(): int
{
    return $this->executeWithTiming(function () {
        $this->displayInfo('Starting report generation...');

        // ... do work ...

        $this->displaySuccess('Report generated.');

        return Command::SUCCESS;
    });
}
```

### Shorthand with `withTiming()`
A convenience wrapper around `executeWithTiming()` that also prints an optional success message:
```php
public function handle(): int
{
    return $this->withTiming(function () {
        // ... do work ...
    }, 'All records processed successfully.');
}
```
Output:
```bash
 SUCCESS  All records processed successfully.

⏱ Time: 2.0001s (2000123 μs)
```

## Interactive Input
Commands can prompt the user for input at runtime rather than relying solely on arguments and options. This is useful for confirmations, credentials, or anything that should not be hardcoded in a command call.

### Asking a Question
`ask()` prints a prompt and waits for the user to type a response. The second parameter is the default value returned if the user presses Enter without typing anything:
```php
$name = $this->ask('What is your name?', 'Anonymous');
$this->info("Hello, {$name}!");
```

### Hidden Input
`secret()` hides the user's input as they type — ideal for passwords or API keys:
```php
$token = $this->secret('Enter your API token:');
$this->info('Token accepted (' . strlen($token) . ' characters).');
```

### Confirmation
`confirm()` prints a yes/no prompt. The second parameter sets the default (`true` = yes):
```php
$confirmed = $this->confirm('Are you sure you want to delete all records?', false);

if (!$confirmed) {
    $this->displayWarning('Operation cancelled.');
    return Command::FAILURE;
}
```

### Full Example
```php
public function handle(): int
{
    $name = $this->ask('Your name?', 'Anonymous');
    $this->info("Hello, {$name}!");

    $password = $this->secret('Enter your password:');
    $this->info('Password received (' . strlen($password) . ' characters).');

    $proceed = $this->confirm('Continue with the operation?', true);

    if (!$proceed) {
        $this->displayError('Operation cancelled.');
        return Command::FAILURE;
    }

    $this->displaySuccess('Operation complete.');

    return Command::SUCCESS;
}
```
Console session:
```bash
Your name? [Anonymous]: Alice
Hello, Alice!

Enter your password:
Password received (10 characters).

Continue with the operation? [Y/n]:
 SUCCESS  Operation complete.
```

## Choice-Based Input
When you want to restrict input to a predefined set of options, use `choice()` for a single selection or `multipleChoice()` for selecting several options at once.

### Single Choice
```php
$env = $this->choice(
    'Select deployment environment',
    ['local', 'staging', 'production'],
    0 // default index (0 = 'local')
);

$this->info("Deploying to: {$env}");
```
Console session:
```bash
Select deployment environment:
  [0] local
  [1] staging
  [2] production
> 2
Deploying to: production
```

### Multiple Choice
```php
$modules = $this->multipleChoice(
    'Which modules should be enabled?',
    ['api', 'admin', 'web', 'docs']
);

$this->info('Enabling: ' . implode(', ', $modules));
```
Console session:
```bash
Which modules should be enabled?
  [0] api
  [1] admin
  [2] web
  [3] docs
> 0,2
Enabling: api, web
```

## Progress Bars
For long-running operations, a progress bar gives users visual feedback so they know the command is still working. Create one with `createProgressBar()`, advance it inside your loop, and call `finish()` when done.
```php
public function handle(): int
{
    $items = range(1, 100);

    $this->info('Processing ' . count($items) . ' items...');

    $bar = $this->createProgressBar(count($items));

    foreach ($items as $item) {
        // process $item
        usleep(50000); // simulate work
        $bar->advance();
    }

    $bar->finish();
    $this->newLine();

    $this->displaySuccess('All items processed.');

    return Command::SUCCESS;
}
```
Console output:
```bash
Processing 100 items...
100/100 [████████████████████████████████████████] 100%
 SUCCESS  All items processed.
```

## Display Tables
Use `createTable()` to render structured data in a clean grid. This is useful for listing records, configurations, or any data with rows and columns.
```php
public function handle(): int
{
    $table = $this->createTable();

    $table->setHeaders(['ID', 'Name', 'Email', 'Status']);

    $table->addRow([1, 'Alice', 'alice@example.com', 'active']);
    $table->addRow([2, 'Bob',   'bob@example.com',   'inactive']);
    $table->addRow([3, 'Carol', 'carol@example.com', 'active']);

    $table->render();

    return Command::SUCCESS;
}
```
Console output:
```bash
+----+-------+-------------------+----------+
| ID | Name  | Email             | Status   |
+----+-------+-------------------+----------+
| 1  | Alice | alice@example.com | active   |
| 2  | Bob   | bob@example.com   | inactive |
| 3  | Carol | carol@example.com | active   |
+----+-------+-------------------+----------+
```

## Invoking Commands Programmatically
You can run any Pool command from inside your application — for example, inside a controller, route, or service — using the `Pool` facade. Pass the full command string exactly as you would type it in the terminal:
```php
use Phaseolies\Support\Facades\Pool;

Pool::call('mail:send pending --limit=100');
```
The call is dispatched as a background process. The method returns metadata about the spawned process, including its PID and start time:
```json
{
  "pid": 29886,
  "start_time": 1748450010
}
```
This is useful for triggering background jobs from user interactions, webhooks, or other runtime events without blocking the HTTP response.
