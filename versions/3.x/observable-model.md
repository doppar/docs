---
title: Observable Model Properties
description: Doppar #[Watches] attribute documentation — property-level reactive observation
meta:
- name: keywords
  content: Watches, Observable, Model, Reactive, Doppar, Property Observer
---

- [Introduction](#introduction)
- [How Watches Work](#how-watches-work)
- [Generating a Watcher](#generating-a-watcher)
- [Defining a Watch](#defining-a-watch)
  - [Why protected?](#why-protected)
- [Writing a watcher](#writing-a-watcher)
- [Conditional Watches](#conditional-watches)
  - [Method-Based Condition](#method-based-condition)
  - [Class-Based Condition](#class-based-condition)
- [Multiple Watches on One Property](#multiple-watches-on-one-property)
- [Multiple Watched Properties](#multiple-watched-properties)
- [watcher Resolution via Dependency Injection](#watcher-resolution-via-dependency-injection)
- [Execution Order](#execution-order)
- [Watches vs Hooks](#watches-vs-hooks)
- [Bypassing Watches](#bypassing-watches)
- [Testing with Watches](#testing-with-watches)
- [API Reference](#api-reference)

## Observable Model

### Introduction

Observable Model Properties let you declare reactive watchers directly on model properties using the `#[Watches]` PHP attribute. When a watched property is modified and `save()` or `update()` succeeds, Doppar automatically detects the change and invokes your watcher — passing the old value, the new value, and the full model instance.

This is **property-level** observation. It is fundamentally different from Doppar's own `#[Hook]` system (which fires on lifecycle events like `before_updated`). Watches react to which specific column changed and what its new value is.

None of the above requires a hook, a service provider entry, or an explicit `if ($this->isDirtyAttr('status'))` check. The attribute is the contract.

## How Watches Work

Doppar's watch system is **fully attribute-driven** and **reflection-cached**. Here is what happens internally from boot to execution:

**Boot phase (once per class)**

When a model class is instantiated for the first time, `registerHooks()` is called. This in turn calls `registerWatchesAttributes()`, which uses PHP's `ReflectionClass` to scan every property on the model for `#[Watches]` attributes. The result — a flat list of `[property, watcher, when]` triples — is stored in a static per-class cache (`$watchesAttributeCache`). Every subsequent instantiation of that class reads from the cache without triggering reflection again.

**Save phase (every save)**

When `save()` or `update()` runs and the database UPDATE succeeds, Doppar computes the set of dirty attributes and calls `firePropertyWatches($dirty)`. For each dirty attribute, `WatchesHandler::fireForDirty()` checks whether any watches are registered for that property on the current model class. If watches exist, it evaluates the optional condition and, if it passes, resolves the watcher class from the DI container and calls `handle($old, $new, $model)`.

Watches fire **after** `after_updated` lifecycle hooks and **before** `$originalAttributes` is reset. This means both the pre-save and post-save values are accessible in every watcher.

**Key guarantees:**
- Watches only fire on a successful database write — if the UPDATE query fails, no watcher is called
- Watches respect `withoutHook()` — disabling hooks also disables watches for that operation
- Reflection runs at most once per class per process lifetime regardless of how many instances are created
- watchers are resolved via `app()` so the full DI container is available

## Generating a Watcher

Use the `make:watcher` pool command to scaffold a new watcher watcher class:

```bash
php pool make:watcher ProductStatusWatcher
```

This creates `app/Watchers/ProductStatusWatcher.php`:

```php
<?php

namespace App\Watchers;

use Phaseolies\Database\Entity\Model;

class ProductStatusWatcher
{
    /**
     * Handle the watched property change.
     *
     * @param mixed $old
     * @param mixed $new
     * @param Model $model
     * @return void
     */
    public function handle(mixed $old, mixed $new, Model $model): void
    {
        //
    }
}
```

The command outputs the file path and class name on success, and exits with a non-zero code (without overwriting) if the file already exists.

## Defining a Watch

Place `#[Watches]` on a model property, passing the fully-qualified watcher class name as the first argument.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\Watches;
use App\Watchers\OrderStatusChanged;
use App\Watchers\TriggerFraudReview;

class Order extends Model
{
    protected $table = 'orders';

    protected $creatable = ['user_id', 'status', 'total', 'notes'];

    // Fires whenever 'status' changes
    // And the row is successfully saved
    #[Watches(OrderStatusChanged::class)]
    protected $status;

    // Fires whenever 'total' changes
    // Conditionally (see Conditional Watches)
    #[Watches(TriggerFraudReview::class, when: 'isFraudRisk')]
    protected $total;
}
```

The attribute is repeatable — you can stack multiple `#[Watches]` on the same property (see [Multiple Watches on One Property](#multiple-watches-on-one-property)).

### Why protected?

Watched properties **must be declared `protected` or `private`**, not `public`. The reason is identical to cast properties:

When a property is `public`, PHP resolves `$model->property` as a direct property access and bypasses `__get()`. Since Doppar stores all data in the internal `$attributes` array and never initialises typed properties directly, a `public` typed property would be uninitialized and throw a `TypeError`.

With `protected` or `private`, external reads go through `__get()` and external writes go through `__set()`, both of which route correctly through Doppar's attribute system.

```php
// Correct — protected, routing goes through __get / __set
#[Watches(OrderStatusChanged::class)]
protected $status;

// Also correct — private works the same way
#[Watches(NotifyAccounting::class)]
private $total;

// Wrong — public bypasses magic methods, attribute will have no effect
#[Watches(OrderStatusChanged::class)]
public $status;
```

## Writing a watcher

A watcher is a plain PHP class with a `handle()` method. It receives:

| Parameter | Type | Description |
|---|---|---|
| `$old` | `mixed` | The attribute value **before** the change (from `originalAttributes`) |
| `$new` | `mixed` | The attribute value **after** the change (the value that was written to the database) |
| `$model` | `Model` | The full model instance, fully populated and post-save |

```php
<?php

namespace App\Watchers;

use Phaseolies\Database\Entity\Model;

class OrderStatusChanged
{
    public function handle(mixed $old, mixed $new, Model $model): void
    {
        // $old  — previous status e.g. 'pending'
        // $new  — new status     e.g. 'shipped'
        // $model — the Order instance that was just saved

        info("Order #{$model->id} status changed: {$old} → {$new}");

        if ($new === 'shipped') {
            app('mailer')->send(new ShipmentConfirmation($model));
        }

        if ($new === 'cancelled') {
            app('inventory')->restoreStock($model);
        }
    }
}
```

The watcher class is resolved through Doppar's DI container (`app(WatcherClass::class)`), so you can type-hint constructor dependencies and they will be injected automatically.

```php
<?php

namespace App\Watchers;

use Phaseolies\Database\Entity\Model;
use App\Services\FraudScorer;
use App\Services\AlertService;

class TriggerFraudReview
{
    public function __construct(
        private FraudScorer  $scorer,
        private AlertService $alerts,
    ) {}

    public function handle(mixed $old, mixed $new, Model $model): void
    {
        $score = $this->scorer->evaluate($model);

        if ($score->isSuspicious()) {
            $this->alerts->flagOrder($model->id, $score->reason());
        }
    }
}
```

## Conditional Watches

Watches fire unconditionally by default. The optional `when` parameter lets you attach a condition — the watcher is skipped unless the condition returns `true`.

Two forms are supported:

| Form | `when` value | Condition evaluated as |
|---|---|---|
| Method on the model | `'methodName'` | `$model->methodName($old, $new): bool` |
| Condition class | `ConditionClass::class` | `app(ConditionClass::class)->evaluate($old, $new, $model): bool` |

### Method-Based Condition

Declare a public or protected method on the model. The method receives the old value and the new value and **must return `bool`**.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\Watches;
use App\Watchers\TriggerFraudReview;
use App\Watchers\NotifyLargeWithdrawal;

class Order extends Model
{
    #[Watches(TriggerFraudReview::class, when: 'isFraudRisk')]
    protected $total;

    // Receives ($old, $new) — fires watcher only when $new exceeds the threshold
    public function isFraudRisk(mixed $old, mixed $new): bool
    {
        return (float) $new > 10000;
    }
}
```

This will not fire the watcher
```php
$order = Order::find(1);

$order->total = 5000;
$order->save();
// isFraudRisk(old, 5000) → false → TriggerFraudReview NOT called
```

This will fire the watcher
```php
$order->total = 15000;
$order->save();
// isFraudRisk(old, 15000) → true → TriggerFraudReview::handle() fires
```

The condition method is called with the **original value and the new value** exactly as they exist in the database row — after sanitisation and casting. You do not need to read dirty attributes manually.

> **Note:** If the method does not exist on the model, Doppar throws a `RuntimeException` at runtime. If it exists but does not return a `bool`, a `RuntimeException` is also thrown. These are programming errors, not recoverable states.

### Class-Based Condition

For conditions that need injected dependencies, belong to multiple models, or are complex enough to warrant their own unit tests, implement `WatchConditionInterface`.

```php
<?php

namespace Phaseolies\Database\Entity\Watches;

use Phaseolies\Database\Entity\Model;

interface WatchConditionInterface
{
    public function evaluate(mixed $old, mixed $new, Model $model): bool;
}
```

Create a condition class and reference it by class name in the `when` parameter:

```php
<?php

namespace App\Conditions;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Watches\WatchConditionInterface;
use App\Services\RiskConfigService;

class FraudThreshold implements WatchConditionInterface
{
    public function __construct(private RiskConfigService $config) {}

    public function evaluate(mixed $old, mixed $new, Model $model): bool
    {
        $limit = $this->config->getFraudLimit($model->currency ?? 'USD');

        return (float) $new > $limit;
    }
}
```

Now the usage example on model. The condition class is resolved from the DI container — dependencies are injected.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\Watches;
use App\Watchers\TriggerFraudReview;
use App\Conditions\FraudThreshold;

class Order extends Model
{
    #[Watches(TriggerFraudReview::class, when: FraudThreshold::class)]
    protected $total;
}
```

The condition class is resolved via `app(FraudThreshold::class)`, so constructor dependencies are injected automatically — exactly like a watcher watcher.

> **Note:** If the class named in `when` exists but does not implement `WatchConditionInterface`, Doppar throws a `RuntimeException` identifying the offending class. If the class does not exist, Doppar falls back to treating the string as a model method name.

## Multiple Watches on One Property

`#[Watches]` is a repeatable attribute. Stack as many as needed on a single property. All registered watches are evaluated in declaration order.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\Watches;
use App\Watchers\NotifyAccounting;
use App\Watchers\TriggerFraudReview;
use App\Watchers\UpdateRevenueReport;
use App\Conditions\FraudThreshold;

class Order extends Model
{
    // All three watches fire when 'total' changes
    // The second fires only when the condition passes
    #[Watches(NotifyAccounting::class)]
    #[Watches(TriggerFraudReview::class, when: FraudThreshold::class)]
    #[Watches(UpdateRevenueReport::class)]
    protected $total;
}
```

## Multiple Watched Properties

A model can have any number of watched properties. Watches are property-scoped — changing `status` only fires `status` watches; changing `total` only fires `total` watches.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\Watches;
use App\Watchers\OrderStatusChanged;
use App\Watchers\TriggerFraudReview;
use App\Watchers\ShippingAddressUpdated;
use App\Watchers\OrderNoteLogged;
use App\Conditions\FraudThreshold;

class Order extends Model
{
    protected $creatable = ['user_id', 'status', 'total', 'shipping_address', 'notes'];

    #[Watches(OrderStatusChanged::class)]
    protected $status;

    #[Watches(TriggerFraudReview::class, when: FraudThreshold::class)]
    protected $total;

    #[Watches(ShippingAddressUpdated::class)]
    protected $shipping_address;

    #[Watches(OrderNoteLogged::class)]
    protected $notes;
}
```

When multiple watched properties are dirty in a single save(), Doppar fires watches for every dirty property. Each property's watches are evaluated independently before moving to the next.
```php
$order = Order::find(1);

// Only modifying status → only OrderStatusChanged fires
$order->status = 'processing';
$order->save();

// Only modifying total → only TriggerFraudReview fires (if condition passes)
$order->total = 8000;
$order->save();

// Modifying multiple properties at once → multiple watches may fire
$order->status = 'shipped';
$order->total  = 15000;
$order->save();
// → OrderStatusChanged::handle()    fires (status changed)
// → TriggerFraudReview::handle()    fires (total changed, condition passed)
```

## watcher Resolution via Dependency Injection

Both watcher classes and class-based condition classes are resolved through Doppar's DI container using `app(ClassName::class)`. This means:

- Constructor dependencies are injected automatically
- Singleton bindings are respected — a singleton watcher receives the same instance every time
- You can bind an interface in a service provider and use the interface name as the watcher class

```php
// In a service provider
use App\Watchers\OrderStatusChanged;

public function register(): void
{
    $this->app->singleton(
        OrderStatusChanged::class,
        fn() => new OrderStatusChanged(config('mail.order_team'))
    );
}
```

In the model — the singleton is resolved, not re-instantiated
```php
#[Watches(OrderStatusChanged::class)]
protected $status;
```

## Execution Order

Within a single `save()` call, operations happen in this order:

```
save() call
├── fireBeforeHooks('updated')       ← #[Hook('before_updated')] methods
├── getDirtyAttributes()             ← compute what changed
├── UPDATE query                     ← write to database
├── fireAfterHooks('updated')        ← #[Hook('after_updated')] methods / Temporal snapshots
├── firePropertyWatches($dirty)      ← #[Watches] watchers  ← here
└── $originalAttributes = $attributes  ← reset dirty tracking
```

Watches always fire **after** all lifecycle hooks and **after** the database write. They have access to:
- The final persisted values via `$model->attributes`
- The original pre-save values via `$model->getOriginalAttributes()`
- The full model instance with all relationships and metadata

## Watches vs Hooks

Both `#[Watches]` and `#[Hook]` react to model lifecycle events. They serve different purposes.

| Feature | `#[Hook]` | `#[Watches]` |
|---|---|---|
| **Placement** | On model methods | On model properties |
| **Granularity** | Lifecycle event (e.g. any update) | Specific column change |
| **Receives** | Model instance only | Old value, new value, model instance |
| **Condition** | Method name returning bool | Method name or `WatchConditionInterface` class |
| **Use case** | Cross-cutting logic on every save/delete | Reactive logic tied to a specific field value |
| **Observer location** | Inline on the model | External watcher class |

**Use `#[Hook]` when** you need to run logic any time the model is saved, updated, or deleted — regardless of which column changed. Slug generation, timestamp normalisation, and audit-log snapshotting are good examples.

**Use `#[Watches]` when** you need to react to a *specific column* changing to *a specific value* — sending a shipment email when `status` becomes `'shipped'`, flagging a fraud review when `total` exceeds a threshold, or invalidating a cache key when a `slug` changes.

```php
class Order extends Model
{
    // Hook — fires on every update, column-agnostic
    #[Hook('before_updated')]
    protected function normaliseAddress(): void
    {
        $this->shipping_address = strtolower(trim($this->shipping_address));
    }

    // Watch — fires only when 'status' column specifically changes
    #[Watches(OrderStatusChanged::class)]
    protected $status;
}
```

## Bypassing Watches

Watches are tied to Doppar's hook system. Calling `withoutHook()` disables both lifecycle hooks **and** property watches for that operation.

Neither hooks nor watches fire
```php
Order::withoutHook()->update(['status' => 'cancelled', 'total' => 0]);
```

This is useful for data migrations, seeding, and admin operations where you explicitly do not want side effects.

## Testing with Watches

**Resetting the reflection cache between tests**

`resetWatchesCache()` clears the static reflection cache. Call it in `setUp()` or `tearDown()` when you swap model classes or redefine `#[Watches]` attributes between tests.

```php
protected function tearDown(): void
{
    Order::resetWatchesCache();
    WatchesHandler::flush(Order::class);
    parent::tearDown();
}
```

**Flushing the watch registry**

`WatchesHandler::flush()` removes registered watches. Pass a class name to flush only that model's watches, or call it with no argument to reset everything.

```php
use Phaseolies\Database\Entity\Watches\WatchesHandler;

// Flush one model
WatchesHandler::flush(Order::class);

// Flush all
WatchesHandler::flush();
```

**Asserting a watcher was called**

Because watchers are resolved via `app()`, you can swap them in the DI container during tests:

```php
use App\Watchers\TriggerFraudReview;

public function test_fraud_watcher_fires_when_total_exceeds_threshold(): void
{
    $watcher = $this->createMock(TriggerFraudReview::class);
    $watcher->expects($this->once())
            ->method('handle')
            ->with(
                $this->equalTo(5_000),   // $old
                $this->equalTo(15_000),  // $new
                $this->isInstanceOf(Order::class)
            );

    app()->instance(TriggerFraudReview::class, $watcher);

    $order = Order::find(1);
    $order->total = 15_000;
    $order->save();
}

public function test_fraud_watcher_does_not_fire_below_threshold(): void
{
    $watcher = $this->createMock(TriggerFraudReview::class);
    $watcher->expects($this->never())->method('handle');

    app()->instance(TriggerFraudReview::class, $watcher);

    $order = Order::find(1);
    $order->total = 5_000;
    $order->save();
}
```

**Testing a condition method directly**

Model condition methods are regular public methods, so you can unit-test them without touching the database:

```php
public function test_is_fraud_risk_returns_true_above_threshold(): void
{
    $order = new Order();

    $this->assertTrue($order->isFraudRisk(0, 15_000));
    $this->assertFalse($order->isFraudRisk(0, 5_000));
}
```

**Testing a condition class directly**

```php
public function test_fraud_threshold_condition(): void
{
    $condition = new FraudThreshold();
    $order     = new Order();

    $this->assertTrue($condition->evaluate(0, 15_000, $order));
    $this->assertFalse($condition->evaluate(0, 5_000, $order));
}
```

## API Reference

### `#[Watches]` Attribute

**Namespace:** `Phaseolies\Database\Entity\Attributes\Watches`

**Target:** `TARGET_PROPERTY` — can only be placed on class properties. Repeatable.

| Parameter | Type | Required | Description |
|---|---|---|---|
| `$watcher` | `string` | Yes | Fully-qualified class name of the watcher watcher. Must expose `handle(mixed $old, mixed $new, Model $model): void`. Scaffold with `php pool make:watcher`. |
| `$when` | `string\|null` | No | Condition. A model method name (`'methodName'`) or a class name implementing `WatchConditionInterface`. Omit to always fire. |

### `WatchConditionInterface`

**Namespace:** `Phaseolies\Database\Entity\Watches\WatchConditionInterface`

Implement this interface to write a reusable, injectable, testable watch condition.

```php
interface WatchConditionInterface
{
    public function evaluate(mixed $old, mixed $new, Model $model): bool;
}
```

### `WatchesHandler`

**Namespace:** `Phaseolies\Database\Entity\Watches\WatchesHandler`

The static registry and execution engine. You rarely call this directly — the `InteractsWithWatches` trait and the model's save pipeline call it internally. Useful in tests and advanced scenarios.

| Method | Description |
|---|---|
| `WatchesHandler::register(string $modelClass, string $property, string $watcher, ?string $when)` | Register a watch entry programmatically. Duplicate entries are silently ignored. |
| `WatchesHandler::fireForDirty(Model $model, array $dirty)` | Fire all registered watches for every property in `$dirty`. Called automatically by `save()` and `update()`. |
| `WatchesHandler::flush(?string $modelClass = null)` | Clear the registry. Pass `null` to flush all models. Used in tests. |

### `InteractsWithWatches` Trait

**Namespace:** `Phaseolies\Database\Entity\Watches\InteractsWithWatches`

Mixed into the `Model` base class. Provides:

| Method | Description |
|---|---|
| `registerWatchesAttributes()` | Scans `#[Watches]` attributes via reflection and registers them in `WatchesHandler`. Called once per class during `registerHooks()`. |
| `firePropertyWatches(array $dirty)` | Delegates to `WatchesHandler::fireForDirty()`. Called by `save()` and `update()` after a successful database write. |
| `static resetWatchesCache(?string $class = null)` | Clears the per-class reflection cache. Pass `null` to reset all. |
