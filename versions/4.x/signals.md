---
title: Signals
description: Doppar signals - settle-aware domain notifications that wait for a database transaction to commit
meta:
  - name: keywords
    content: signals, signal bus, DeliverOnSettle, emitOnSettle, transaction, commit, rollback, nested transactions, listeners, priority, domain events
---

## Signals

### Introduction

Applications often need to react to something that just happened: send a receipt after an invoice is paid, refresh a search index after an order is stored, notify a warehouse after stock changes. If that reaction fires **inside** a database transaction, it can announce work that is later rolled back, and the customer gets a receipt for a payment that never happened.

A **signal** is a small object describing something that happened. Doppar's signal bus delivers it to your listeners **synchronously**, and when the signal is emitted inside a transaction it can hold it back until the transaction has **settled**, meaning the outermost transaction really committed. If the transaction rolls back, the signal is simply discarded.

```php
use Phaseolies\Support\Facades\DB;
use Phaseolies\Support\Facades\Signal;

DB::transaction(function () use ($invoice) {
    $invoice->markPaid();

    // InvoicePaid has #[DeliverOnSettle], so it is held until the
    // transaction commits and dropped if it rolls back.
    Signal::emit(new InvoicePaid($invoice->id));
});
```

### Why Use Signals

- *Commit-safe* → listeners only run for work that was actually committed
- *Rollback-safe* → a rolled back transaction leaves no signal behind
- *Nesting-aware* → inner transactions hand their signals to the parent, and an inner rollback only drops its own
- *Per connection* → every database connection settles on its own
- *Predictable* → synchronous, priority ordered, no queue, no background process

## Defining a Signal

A signal is any plain PHP object. It carries the data its listeners need and nothing else. Scaffold one with the `make:signal` command:

```bash
php pool make:signal InvoicePaid
```

This creates `App\Signals\InvoicePaid`. Add `--settle` to hold the signal until the transaction commits:

```bash
php pool make:signal InvoicePaid --settle
```

This adds `#[DeliverOnSettle]` to the class. To wait for a specific connection:

```bash
php pool make:signal ReportStored --connection=analytics
```

This adds `#[DeliverOnSettle(connection: 'analytics')]`. Nested names such as `Billing/InvoicePaid` create the matching folder and namespace.

`--connection` implies `--settle`. The command refuses to overwrite an existing class. The class is created in `src/Signals`; add the constructor properties your listeners need:

```php
<?php

namespace App\Signals;

use Phaseolies\Signals\Attributes\DeliverOnSettle;

#[DeliverOnSettle]
class InvoicePaid
{
    public function __construct(public readonly int $invoiceId) {}
}
```

The `#[DeliverOnSettle]` attribute declares that this signal must wait for the open transaction to commit. Without the attribute, a signal is delivered the moment it is emitted (see [`Emitting Signals`](#emitting-signals)).

To wait for a transaction on a specific connection:

```php
#[DeliverOnSettle(connection: 'analytics')]
class ReportStored
{
    public function __construct(public readonly int $reportId) {}
}
```

## Registering Listeners

Listeners are registered once, while the application boots, from the `launch()` method of a service launcher. Pass the signal class and the listener to `Signal::on()`:

```php
<?php

namespace App\Launchers;

use App\Listeners\SendReceipt;
use App\Signals\InvoicePaid;
use Phaseolies\Launchers\ServiceLauncher;
use Phaseolies\Support\Facades\Signal;

class AppLauncher extends ServiceLauncher
{
    public function launch(): void
    {
        Signal::on(InvoicePaid::class, SendReceipt::class);
    }
}
```

When a signal has several listeners, use the `priority` argument to control the order. A higher number runs earlier:

```php
public function launch(): void
{
    Signal::on(InvoicePaid::class, SendReceipt::class);

    Signal::on(InvoicePaid::class, function (InvoicePaid $signal) {
        info("Invoice {$signal->invoiceId} paid");
    }, priority: 10);
}
```

A listener can be:

| Form | Example |
|------|---------|
| A closure or any callable | `fn (InvoicePaid $signal) => ...` |
| An invokable class name | `SendReceipt::class` (resolved through the container, calls `__invoke`) |
| A `[class, method]` pair | `[SendReceipt::class, 'handle']` |
| An `[object, method]` pair | `[$receipt, 'handle']` |

Class based listeners are built through the container for every delivery, so their constructor dependencies are injected and no state leaks between deliveries.

```php
<?php

namespace App\Listeners;

use App\Signals\InvoicePaid;

class SendReceipt
{
    public function __invoke(InvoicePaid $signal): void
    {
        // The invoice is committed at this point.
    }
}
```

A listener that cannot be called later is rejected immediately with an `InvalidArgumentException`, when it is registered, rather than failing at delivery time.

### Priorities

Listeners with a **higher priority run first**. Listeners with the same priority run in the order they were registered. The default priority is `0`, and negative priorities are allowed.

```php
Signal::on(InvoicePaid::class, $third, priority: -5);
Signal::on(InvoicePaid::class, $first, priority: 10);
Signal::on(InvoicePaid::class, $second);
```

Here `$first` runs first, `$second` runs with the default priority of `0`, and `$third` runs last.

### Listening to a Parent or an Interface

A listener registered for a parent class or an interface also receives every signal that extends or implements it.

```php
Signal::on(AuditableSignal::class, $audit);
```

If `InvoicePaid` and `InvoiceVoided` both extend `AuditableSignal`, `$audit` receives both.

Priorities are compared across all matching listeners, so a high priority listener on an interface runs before a lower priority one on the concrete class.

## Emitting Signals

| Method | When listeners run |
|--------|--------------------|
| `Signal::emit($signal)` | Right away, unless the signal class has `#[DeliverOnSettle]` and a transaction is open |
| `Signal::emitOnSettle($signal, $connection = null)` | After the open transaction on that connection settles. Delivered right away if none is open |
| `Signal::emitNow($signal)` | Right away, always, even inside a transaction |

You may also use the `signal()` helper. Pass it a signal to emit it, which is the same as calling `Signal::emit()`:

```php
signal(new InvoicePaid($invoice->id));
```

Call it without an argument to get the signal bus:

```php
signal()->on(InvoicePaid::class, $listener);
```

`emitOnSettle()` lets you bind any signal to a transaction without the attribute, and it returns `true` when the signal was held back or `false` when it was delivered immediately:

```php
$held = Signal::emitOnSettle(new StockChanged($sku), connection: 'warehouse');
```

## Waiting for a Transaction

Signals that wait are tied to the transaction that is open on their connection at the moment they are emitted. In the examples below, every signal class carries `#[DeliverOnSettle]`; a plain signal would be delivered immediately by `emit()`.

```php
use Phaseolies\Support\Facades\DB;
use Phaseolies\Support\Facades\Signal;

DB::transaction(function () {
    $order = Order::create([
        'customer_id' => $customerId,
        'total' => $total,
    ]);

    Signal::emit(new OrderPlaced($order->id));
});
```

`OrderPlaced` is not delivered while the closure runs. Once the transaction commits, its listeners run exactly once. If the closure throws, the transaction rolls back and the signal is discarded, so no listener ever sees an order that was never saved.

What you can rely on:

- A held signal is delivered **once**, **after** the outermost commit, in the order it was emitted.
- By the time a listener runs, the transaction is already closed and the data is visible. A listener may open its own transaction, and signals it emits in there settle on that new transaction.
- If the transaction rolls back, or the connection is dropped, every signal it was holding is discarded.
- With no open transaction on the connection, the signal is delivered immediately.
- `DB::transaction($callback, attempts: 3)` retries discard the signals of a failed attempt before the next one starts, so only the successful attempt delivers.
- The `#[Transaction]` controller attribute behaves the same, because it uses the same transaction layer.

## Nested Transactions

Doppar nests transactions with savepoints. Signals follow the same structure (`A`, `B`, `C` and `D` all carry `#[DeliverOnSettle]`):

```php
DB::transaction(function () {
    Signal::emit(new A());

    DB::transaction(function () {
        Signal::emit(new B());
    });

    try {
        DB::transaction(function () {
            Signal::emit(new C());

            throw new \RuntimeException('inner failed');
        });
    } catch (\RuntimeException) {
        // The inner transaction was rolled back.
    }

    Signal::emit(new D());
});
```

What happens, step by step:

| Step | Result |
|------|--------|
| The first inner transaction commits | `B` moves up to the outer transaction, nothing is delivered yet |
| The second inner transaction rolls back | `C` is discarded |
| The outer transaction commits | `A`, `B` and `D` are delivered, in the order they were emitted |

- Committing an inner level hands its signals to the parent level.
- Rolling back a level discards its signals **and** the signals of every inner level that already committed into it.
- Only the outermost commit delivers anything.
- If the outer transaction rolls back after inner levels committed, nothing is delivered.

## Multiple Connections

Each connection has its own transaction state, so each settles independently. A signal waits for the transaction of **one** connection: the one named by `#[DeliverOnSettle(connection: ...)]`, by `emitOnSettle(..., connection: ...)`, or the default connection when none is named.

```php
use Phaseolies\Support\Facades\DB;
use Phaseolies\Support\Facades\Signal;

DB::connection('billing')->transaction(function () {
    DB::connection('analytics')->transaction(function () {
        Signal::emitOnSettle(new PaymentStored(), 'billing');
        Signal::emitOnSettle(new ReportStored(), 'analytics');
    });
});
```

The two transactions commit separately, so the signals are delivered separately:

| Signal | Waits for | Delivered when |
|--------|-----------|----------------|
| `ReportStored` | `analytics` | The inner transaction commits |
| `PaymentStored` | `billing` | The outer transaction commits |

A rollback on one connection never discards the signals waiting on another. A signal bound to a connection with no open transaction is delivered immediately, even if a different connection has one open.

If a single signal depends on two connections committing, emit it from the code that runs after the outer one commits, or from a listener of a signal bound to the later connection.

## Listener Failures

Failure handling depends on **when** the signal is delivered:

| Delivery | A listener throws | Remaining listeners |
|----------|-------------------|---------------------|
| Immediate (a signal that is not held, or `emitNow`) | The exception **propagates** to the code that emitted the signal | Do not run |
| After a commit (a held signal) | **Not propagated**, it is reported | Still run |

After a commit the data is already permanent, so letting a listener exception escape would make a successful commit look like a failure and would stop the other listeners and the other pending signals. Instead each failure is reported and delivery continues.

A failure is reported in this order:

1. The reporter set with `Signal::reportFailuresUsing()`, if any.
2. Otherwise the `log` service, as an `error` entry with the exception in its context.
3. If there is no reporter or log service, or the reporter or logger itself throws, a best-effort `error_log()` line. Failures are never silently dropped, and reporting can never break the commit.

```php
Signal::reportFailuresUsing(function (\Throwable $failure, object $signal): void {
    report_to_tracker($failure, ['signal' => $signal::class]);
});
```

If a listener must not be lost when it fails, do the durable part inside the transaction instead, or have the listener hand the work to a queue.

## Long-Running Workers

Listeners are registered once and live as long as the process. Pending signals are held per connection and are cleared when the transaction commits, rolls back, or the connection is disconnected, so nothing carries over from one request or job to the next. Class based listeners are rebuilt for each delivery, so they cannot keep state between requests either.

## Limitations

- Signals are **synchronous**. There is no built-in queue; to process a signal in the background, have a listener dispatch a job after the commit.
- Only transactions started through Doppar (`DB::transaction()`, `DB::beginTransaction()`, `#[Transaction]`) are tracked. A transaction opened directly on a raw PDO object is not seen.
- A signal waits for one connection's transaction, see [`Multiple Connections`](#multiple-connections).

## API Reference

| Method | Returns | Description |
|--------|---------|-------------|
| `Signal::on(string $signal, callable\|array\|string $listener, int $priority = 0)` | `SignalBus` | Register a listener, higher priority runs first |
| `Signal::forget(?string $signal = null)` | `SignalBus` | Remove the listeners of a signal, or all of them |
| `Signal::hasListeners(object\|string $signal)` | `bool` | Determine whether any listener would receive the signal |
| `Signal::emit(object $signal)` | `void` | Emit, honoring `#[DeliverOnSettle]` |
| `Signal::emitOnSettle(object $signal, ?string $connection = null)` | `bool` | Hold until the connection's transaction settles, `true` if it was held |
| `Signal::emitNow(object $signal)` | `void` | Deliver right away, ignoring transactions |
| `Signal::reportFailuresUsing(?callable $reporter)` | `SignalBus` | Customize how listener failures after a commit are reported |
| `signal(?object $signal = null)` | `SignalBus` or `void` | Get the bus, or emit the given signal |

| Attribute | Target | Description |
|-----------|--------|-------------|
| `#[DeliverOnSettle(?string $connection = null)]` | Signal class | Wait for the open transaction on the connection (default connection when `null`) |
