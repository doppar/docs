---
title: Reactive Model Properties
description: Doppar #[Watches] attribute documentation — property-level reactive observation
meta:
- name: keywords
  content: Watches, Reactive, Model, Reactive, Doppar, Property Observer
---

## Reactive Model

### Introduction

Reactive Model provides a convenient way to react to changes on individual model properties using the `#[Watches]` attribute. When a watched property is modified and `save()` or `update()` succeeds, Doppar automatically detects the change and invokes the registered watcher with the old value, the new value, and the complete model instance.

This is property-level observation. Unlike Doppar's `#[Hook]` system, which responds to model lifecycle events such as `before_updated` or `after_updated`, `#[Watches]` focuses on which specific property changed and how its value changed.

Reactive Model removes the need for manually checking dirty attributes or adding model hooks solely to respond to a particular property change. The `#[Watches]` attribute defines the relationship between a model property and its watcher, allowing the reactive behavior to remain explicit, reusable, and separated from the model's core logic.

Watchers can also be combined with conditional rules, multiple watchers on the same property, dependency injection, and multiple watched properties within the same model. This makes Reactive Model suitable for application behaviors such as responding to status changes, triggering notifications, performing validation-related actions, updating related services, or reacting to significant data changes

## Generating a Watcher

Use the `make:watcher` pool command to scaffold a new watcher class:

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

class Order extends Model
{
    protected $creatable = ['user_id', 'status', 'total', 'notes'];

    #[Watches(OrderStatusChanged::class)]
    protected $status;
}
```

That's the entire contract: `#[Watches(OrderStatusChanged::class)]` tells Doppar to watch the `$status` column. Whenever `status` changes and the row is successfully saved, Doppar instantiates `OrderStatusChanged` and calls its `handle()` method with the old value, the new value, and the saved model — no hook, no launcher entry, no manual dirty-checking required.

From here, `#[Watches]` extends in two independent directions:

- **One property, several watchers** — stack multiple `#[Watches]` attributes on the same property; see [Multiple Watches on One Property](#multiple-watches-on-one-property).
- **Several watched properties** — add `#[Watches]` to more than one property on the same model, each firing independently; see [Multiple Watched Properties](#multiple-watched-properties).

Watches can also be made conditional, so they only fire when a rule you define returns `true` — covered next, in [Conditional Watches](#conditional-watches).

## Writing a Watcher

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

class OrderStatusChanged
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

    // Receives ($old, $new)
    // fires watcher only when $new exceeds the threshold
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

The condition class is resolved via `app(FraudThreshold::class)`, so constructor dependencies are injected automatically — exactly like a watcher.

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

## Watcher Resolution via Dependency Injection

Both watcher classes and class-based condition classes are resolved through Doppar's DI container using `app(ClassName::class)`. This means:

- Constructor dependencies are injected automatically
- Singleton bindings are respected — a singleton watcher receives the same instance every time
- You can bind an interface in a launcher and use the interface name as the watcher class

```php
// In a launcher
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
