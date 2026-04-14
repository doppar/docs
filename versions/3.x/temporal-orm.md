---
title: Temporal Time-Travel ORM
description: Doppar Temporal Time-Travel ORM documentation
meta:
- name: keywords
content: Temporal, Time-Travel, ORM, History, Audit, Snapshot
---

- [Introduction](#introduction)
- [How It Works](#how-it-works)
- [Marking a Model as Temporal](#marking-a-model-as-temporal)
- [Attribute Options](#attribute-options)
- [Running the Migration](#running-the-migration)
- [History Table Structure](#history-table-structure)
- [Time-Travel Queries](#time-travel-queries)
- [Querying History](#querying-history)
- [Diffing Two Points in Time](#diffing-two-points-in-time)
- [Rewinding a Record](#rewinding-a-record)
- [Restoring a Record](#restoring-a-record)
- [Actor Tracking](#actor-tracking)
- [Utility Methods](#utility-methods)
- [Driver Support](#driver-support)

## Temporal ORM

### Introduction

Doppar's **Temporal Time-Travel ORM** gives every model a complete, automatic audit history. Every time a record is created, updated, or deleted, a full snapshot of its state is written to a companion history table. You can then query what any record looked like at any point in the past — down to the microsecond — without changing a single line of your existing model or controller code.

This is useful for:
- **Audit logs** — who changed what and when
- **Compliance** — reproduce the exact state of a record at any date
- **Debugging** — understand how data evolved over time
- **Undo/restore** — roll any record back to a prior state in one call

## How It Works

When you mark a model with the `#[Temporal]` attribute, Doppar automatically:

1. Registers lifecycle hooks (`after_created`, `after_updated`, `after_deleted`) on the model.
2. After every write, captures the full row as a JSON snapshot and writes it — along with metadata — into a `{table}_history` table.
3. Exposes a fluent time-travel API (`::at()`, `->history()`, `->diff()`, `->rewindTo()`, `->restoreTo()`) directly on your model.

No extra code is required in your controllers or services. The history is built silently and automatically.

## Marking a Model as Temporal

Add the `#[Temporal]` attribute to any model class:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Temporal\Attributes\Temporal;

#[Temporal]
class Contract extends Model
{
    protected $table = 'contracts';

    protected $creatable = ['title', 'status', 'amount', 'client_id'];

    protected $timeStamps = true;
}
```

That single attribute is everything Doppar needs. No trait to pull in, no interface to implement, no observer to register — the framework handles the rest automatically.

## Attribute Options

The `#[Temporal]` attribute accepts two optional parameters.

### `suffix`

Controls the name of the history table. By default it is `_history`, so a model backed by the `contracts` table will use `contracts_history`.

```php
#[Temporal(suffix: '_audit')]
class Contract extends Model
{
    //
}
```

With this configuration the history table will be named `contracts_audit` instead.

### `trackActor`

When set to `true`, Doppar records the authenticated user's primary key in the `actor` column of every history row. This lets you answer the question *"who made this change?"* without a separate audit service.

```php
#[Temporal(trackActor: true)]
class Contract extends Model
{
    //
}
```

> **Note:** The `actor` column must exist in the history table. Run `php pool migrate:temporal` after enabling `trackActor: true` — Doppar will add the column to any existing history table automatically.

Both options can be combined:

```php
#[Temporal(suffix: '_audit', trackActor: true)]
class Contract extends Model
{
    //
}
```

## Running the Migration

The `migrate:temporal` command scans your `app/Models` directory, finds every model marked with `#[Temporal]`, and creates the corresponding history table.

```bash
php pool migrate:temporal
```

Example output:

```
🕐 Temporal Migration — connection: pgsql

  Model        : App\Models\Contract
  Base table   : contracts
  History table: contracts_history
  → Created successfully.

  Model        : App\Models\User
  Base table   : users
  History table: users_history
  → Created successfully.

Done. Created: 2
```

The command is **idempotent** — running it again on a table that already exists simply skips it. If a table exists but a new `actor` column is needed (because you added `trackActor: true` after the initial migration), the command adds the column without recreating the table:

```
  Model        : App\Models\User
  Base table   : users
  History table: users_history
  → Added missing `actor` column.
```

### Available Options

| Option | Description |
|--------|-------------|
| `--connection=` | Use a specific database connection (defaults to `database.default`) |
| `--show` | Dry-run: print the SQL statements without executing them |
| `--path=` | Path to scan for models (defaults to `app/Models`) |

Preview the SQL that would be executed
```bash
php pool migrate:temporal --show
```

Use a secondary database connection
```bash
php pool migrate:temporal --connection=pgsql_reports
```

Scan a non-default models directory
```bash
php pool migrate:temporal --path=app/Domain/Models
```

## History Table Structure

For each temporal model, Doppar creates a history table with the following columns:

| Column | Type | Description |
|--------|------|-------------|
| `history_id` | auto-increment integer | Unique identifier for each history row |
| `record_id` | integer | The primary key of the original record |
| `action` | varchar(10) | One of: `created`, `updated`, `deleted` |
| `valid_from` | high-precision datetime | When this state became current (microsecond precision) |
| `snapshot` | JSON / JSONB / TEXT | The full row state at the time of the write |
| `changed_cols` | JSON / JSONB / TEXT | (updates only) Array of column names that changed |
| `actor` | varchar(255) | (optional) PK of the authenticated user who made the change |

The exact column types vary by database driver:

| Column | MySQL | PostgreSQL | SQLite |
|--------|-------|------------|--------|
| `history_id` | `BIGINT UNSIGNED AUTO_INCREMENT` | `BIGSERIAL` | `INTEGER AUTOINCREMENT` |
| `record_id` | `BIGINT UNSIGNED` | `BIGINT` | `INTEGER` |
| `valid_from` | `DATETIME(6)` | `TIMESTAMPTZ` | `TEXT` |
| `snapshot` | `JSON` | `JSONB` | `TEXT` |
| `changed_cols` | `JSON` | `JSONB` | `TEXT` |

Two indexes are created automatically: one on `record_id` for fast per-record lookups, and one on `valid_from` for fast time-range queries.

## Time-Travel Queries

Use the static `::at($datetime)` method to get a query builder scoped to a specific point in time. All results come from the history table and are reconstructed from JSON snapshots — they reflect exactly what the record looked like at that moment.

### Finding a Record at a Point in Time
What did contract 42 look like on 1 January 2024?
```php
$contract = Contract::at('2024-01-01')->find(42);

echo $contract->status; // 'draft'
echo $contract->amount; // 500
```

If the record did not exist at the given datetime (or was deleted before it), `find()` returns `null`.

### Getting All Records at a Point in Time
All contracts that existed and were not deleted as of 1 June 2024
```php
$contracts = Contract::at('2024-06-01')->get();
```

### Filtering with `where`

All standard `where` conditions work on the time-travel builder. Conditions are applied in PHP after the snapshots are loaded from the history table, since the historical state lives inside a JSON column.

```php
$activeContracts = Contract::at('2024-06-01')
    ->where('status', 'active')
    ->get();
```

Supported operators: `=`, `!=`, `<>`, `>`, `>=`, `<`, `<=`, `IS NULL`, `IS NOT NULL`, `IN`, `NOT IN`, `BETWEEN`, `NOT BETWEEN`, `LIKE`, `NOT LIKE`.

### Ordering and Limiting

```php
$topContracts = Contract::at('2024-06-01')
    ->where('status', 'active')
    ->orderBy('amount', 'DESC')
    ->limit(10)
    ->get();
```

### Getting the First Match

```php
$contract = Contract::at('2024-01-15')
    ->where('client_id', 7)
    ->first();
```

### Datetime Formats

`::at()` accepts any of the following formats:

| Input | Interpreted as |
|-------|---------------|
| `'2024-01-01'` | `2024-01-01 23:59:59.999999` (end of day) |
| `'2024-01-01 15:00'` | `2024-01-01 15:00:59.999999` (end of minute) |
| `'2024-01-01 15:00:00'` | `2024-01-01 15:00:00.999999` (end of second) |
| `'2024-01-01 15:00:00.123456'` | `2024-01-01 15:00:00.123456` (exact) |

Date-only strings default to end-of-day so that `::at('2024-01-01')` captures everything that happened on that date.

## Querying History

Call `->history()` on any loaded model instance to retrieve its full chronological audit trail. Each item in the returned collection is an instance of the same model class, populated from the stored snapshot, with additional metadata properties attached.

```php
$contract = Contract::find(42);

foreach ($contract->history() as $entry) {
    echo $entry->__action;       // 'created' | 'updated' | 'deleted'
    echo $entry->__valid_from;   // '2024-01-15 09:23:11.482910+00'
    echo $entry->__history_id;   // 3
    echo $entry->status;         // value at that point in time
    echo $entry->amount;         // value at that point in time

    // For 'updated' entries only — which columns changed
    print_r($entry->__changed_cols); // ['status', 'amount']

    // Only present when trackActor: true
    echo $entry->__actor;        // 1  (the user ID who made the change)
}
```

### Virtual Metadata Properties

Each history entry carries these read-only virtual properties:

| Property | Type | Description |
|----------|------|-------------|
| `__history_id` | int | The row's ID in the history table |
| `__action` | string | `created`, `updated`, or `deleted` |
| `__valid_from` | string | High-precision timestamp of when this state was recorded |
| `__changed_cols` | array\|null | Columns that changed (updates only, `null` otherwise) |
| `__actor` | mixed\|null | PK of the user who triggered the change (requires `trackActor: true`) |

Example: Rendering an Audit Timeline

```php
$contract = Contract::find(42);

foreach ($contract->history() as $entry) {
    $label = match ($entry->__action) {
        'created' => 'Contract was created',
        'updated' => 'Contract was updated — changed: ' . implode(', ', $entry->__changed_cols ?? []),
        'deleted' => 'Contract was deleted',
    };

    echo "[{$entry->__valid_from}] {$label}" . PHP_EOL;
}
```

Output:
```
[2024-01-15 09:23:11.482910+00] Contract was created
[2024-03-02 14:05:33.119204+00] Contract was updated — changed: status, amount
[2024-06-18 11:47:55.804312+00] Contract was updated — changed: status
```

## Diffing Two Points in Time

The `->diff($from, $to)` method compares the state of a record at two different points in time and returns a structured diff.

```php
$contract = Contract::find(42);

$diff = $contract->diff('2024-01-01', '2024-06-01');
```

The return value is an associative array with the following keys:

| Key | Description |
|-----|-------------|
| `from` | The `$from` datetime string as provided |
| `to` | The `$to` datetime string as provided |
| `changes` | Columns whose value changed, each with a `from` and `to` sub-key |
| `added` | Columns present in the `to` snapshot but absent in `from` |
| `removed` | Columns present in the `from` snapshot but absent in `to` |

```php
$diff = $contract->diff('2024-01-01', '2024-06-01');
```
Output:
```php
[
  'from'    => '2024-01-01',
  'to'      => '2024-06-01',
  'changes' => [
    'status' => ['from' => 'draft',  'to' => 'active'],
    'amount' => ['from' => 500,      'to' => 750],
  ],
  'added'   => [],
  'removed' => [],
]
```

If no snapshot exists at either datetime (e.g. the record was created after `$from`, or deleted before `$to`), `diff()` returns `null`:

```php
$diff = $contract->diff('2020-01-01', '2024-06-01');

if ($diff === null) {
    echo 'No snapshot found for one or both dates.';
}
```

### Practical Example: Showing a Change Summary

```php
$contract = Contract::find(42);
$diff = $contract->diff('2024-01-01', '2024-12-31');

if ($diff) {
    foreach ($diff['changes'] as $column => $change) {
        echo "{$column}: {$change['from']} → {$change['to']}" . PHP_EOL;
    }
}
```

Output:
```
status: draft → active
amount: 500 → 750
```

## Rewinding a Record

`->rewindTo($datetime)` returns a **new, unsaved** model instance populated with the state of the record at the given point in time. The original model is not modified.

```php
$contract = Contract::find(42);

// Get an unsaved clone of the record as it was on 1 January 2024
$old = $contract->rewindTo('2024-01-01');

if ($old === null) {
    echo 'No snapshot found at that date.';
}

echo $old->status; // 'draft'
echo $old->amount; // 500

// The live record is unchanged
echo $contract->status; // 'active' (current live value)
```

The rewound instance is a fully hydrated model — you can inspect its properties, serialize it to JSON, or pass it anywhere a model is expected. All original attributes are marked as dirty, which means calling `->save()` on the rewound instance will perform a full `UPDATE` of the live record to the historical state.

> **Note:** `->rewindTo()` does not persist anything. It is a read-only operation that gives you the historical state as a usable model object. To persist the rollback, use `->restoreTo()`.

## Restoring a Record

`->restoreTo($datetime)` rewinds the record to a past state and **immediately saves it** to the database. It is equivalent to calling `->rewindTo($datetime)->save()`.

```php
$contract = Contract::find(42);

// Roll the record back to its state on 1 January 2024 and save it
$success = $contract->restoreTo('2024-01-01');

if ($success) {
    echo 'Contract restored successfully.';
} else {
    echo 'No snapshot found at that date, or save failed.';
}
```

`restoreTo()` returns `true` on success and `false` if no snapshot existed at the given datetime or if the underlying `save()` call failed.

Because `restoreTo()` goes through the normal `save()` path, all model hooks run normally — including the temporal `after_updated` hook, which means the restoration itself is recorded as a new history entry.

### Full Undo Example
This example shows how to roll back a contract to its state from yesterday:
```php
// The contract was incorrectly updated — roll it back to yesterday
$contract = Contract::find(42);
$contract->restoreTo(now()->subDay()->toDateString());

// The history table now has an additional 'updated' entry
// showing the record returned to its previous state
foreach ($contract->history() as $entry) {
    echo "[{$entry->__action}] {$entry->__valid_from}" . PHP_EOL;
}
```

Output:
```
[created] 2024-01-15 09:23:11.482910+00
[updated] 2024-06-18 11:47:55.804312+00   ← the bad change
[updated] 2024-06-19 08:02:44.113900+00   ← the restoration
```

## Actor Tracking

When `trackActor: true` is set on the attribute, Doppar automatically resolves the currently authenticated user and stores their primary key in the `actor` column of every history row.

```php
#[Temporal(trackActor: true)]
class Contract extends Model
{
    //
}
```

After adding `trackActor: true` to an existing model, run the migration command to add the `actor` column to the existing history table:

```bash
php pool migrate:temporal
```

The actor is resolved by calling `auth()->user()->getKey()` at the moment the snapshot is written. If no user is authenticated (e.g. during a console command or a job), the `actor` column is stored as `null`.

### Reading the Actor from History

```php
$contract = Contract::find(42);

foreach ($contract->history() as $entry) {
    if ($entry->__actor) {
        $user = User::find($entry->__actor);
        echo "{$entry->__action} by {$user->name} at {$entry->__valid_from}" . PHP_EOL;
    }
}
```

Output:
```
created by alice at 2024-01-15 09:23:11.482910+00
updated by bob at 2024-03-02 14:05:33.119204+00
updated by alice at 2024-06-18 11:47:55.804312+00
```

### Resilient Actor Detection

If your application does not use authentication or the auth system throws during a console command, Doppar catches the exception silently and stores `null` for the actor. Your application will never crash because of actor resolution.

## Utility Methods

### `isTemporal()`

Returns `true` if the model class is marked with `#[Temporal]`.

```php
$contract = Contract::find(42);

if ($contract->isTemporal()) {
    echo 'This model has time-travel enabled.';
}
```

### `historyTable()`

Returns the name of the history table for the model.

```php
$contract = new Contract();

echo $contract->historyTable(); // 'contracts_history'
```

## Driver Support

The Temporal ORM works with all three database drivers supported by Doppar. The history table DDL and the `valid_from` timestamp format are adjusted automatically per driver — no configuration needed.

| Feature | MySQL | PostgreSQL | SQLite |
|---------|-------|------------|--------|
| History table | `CREATE TABLE IF NOT EXISTS` | `CREATE TABLE IF NOT EXISTS` | `CREATE TABLE IF NOT EXISTS` |
| Timestamp precision | `DATETIME(6)` — microseconds | `TIMESTAMPTZ` — microseconds with timezone | `TEXT` — microseconds as ISO string |
| JSON column | `JSON` | `JSONB` (indexed binary JSON) | `TEXT` |
| Auto-increment | `BIGINT AUTO_INCREMENT` | `BIGSERIAL` | `INTEGER AUTOINCREMENT` |
| Column introspection (actor check) | `SHOW COLUMNS` | `information_schema.columns` | `PRAGMA table_info` |
| Table existence check | `SHOW TABLES LIKE ?` | `to_regclass()` | `sqlite_master` |

> **PostgreSQL tip:** Because the history table uses `JSONB`, you can query the `snapshot` column directly with native PostgreSQL JSON operators (`->`, `->>`, `@>`) if you ever want to build custom history queries outside of Doppar.

## Important Notes

### `withoutHook()` disables temporal snapshots

If you use `Model::withoutHook()->create(...)` or `Model::withoutHook()->save()`, Doppar's lifecycle hooks are suppressed — which means the temporal snapshot for that operation is **not** recorded. Use `withoutHook()` only when you intentionally want to bypass all hooks, including the history capture.

Temporal snapshot IS recorded
```php
User::create([
    'name' => 'Alice',
    'email' => 'alice@example.com'
]);
```

Temporal snapshot is NOT recorded
```php
User::withoutHook()->create([
    'name' => 'Alice',
    'email' => 'alice@example.com'
]);
```

### Deleted records are excluded from time-travel queries

`::at()`, `->rewindTo()`, and `->restoreTo()` automatically exclude records whose last action before the requested datetime was `deleted`. A deleted record will not appear in `::at()->get()`, and `::at()->find($id)` will return `null` for it.

The `->history()` method is the only API that includes `deleted` entries, since it returns the full raw audit trail without filtering.

### The `created` snapshot is recorded before the primary key is written back

Doppar fires `after_created` before the auto-increment primary key is written back to the model's attributes. The Temporal system handles this transparently by falling back to `PDO::lastInsertId()` so that the `record_id` in the history table is always correct.

### History table rows are append-only

The Temporal system never modifies or deletes history rows. Every state transition is a new insert. This guarantees the integrity of your audit trail — you can always trust that the history reflects exactly what happened.
