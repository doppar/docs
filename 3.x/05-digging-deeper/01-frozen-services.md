---
title: Frozen Services
description: Doppar Frozen Services page
meta:
  - name: keywords
    content: Frozen Services, Immutable, Boot-time Frozen Services, #[Immutable]
---

## Frozen Services

### Introduction

Frozen Services is a container-level feature in Doppar that enforces a clear lifecycle boundary for your application's singleton services. A service marked with the `#[Immutable]` attribute is fully configurable during the application boot phase — your service providers can set it up from config files, environment variables, databases, or any external source — and then **permanently frozen** once the container completes its build.

After freezing, any attempt to mutate a property on that service throws an `ImmutableViolationException` immediately, regardless of where in the application the mutation is attempted — controllers, middleware, event listeners, closures, or anywhere else.

This is not a convention. It is not a code-review guideline. The immutability contract is enforced at the **object level** by the container itself, and it works across every injection path Doppar supports.

### What Is Frozen Service

A Frozen Service is a singleton service whose state becomes permanently read-only after the application finishes booting.

During the boot phase, the service can be fully configured from configuration files, environment variables, databases, or any external source. Once the container completes building the service, Doppar freezes the instance and guarantees that its public state cannot change for the rest of the request lifecycle.

This creates a strict and enforceable boundary:
- *Boot phase* → configuration is allowed
- *Runtime phase* → mutation is impossible
- *Guarantee* → every consumer receives a consistent, trustworthy instance

They eliminate an entire class of bugs caused by accidental shared-state mutation.

### Example with Payment Service
Below is a simple frozen service that holds payment configuration. It is configured once during boot and then safely shared everywhere.
```php
<?php

namespace App\Services;

use Phaseolies\DI\Attributes\Immutable;
use Phaseolies\DI\Concerns\EnforcesImmutability;

#[Immutable]
class PaymentService
{
    use EnforcesImmutability;

    public string $gateway = 'stripe';
    public float  $taxRate = 0.08;

    public function calculateTotal(float $amount): float
    {
        return round($amount + ($amount * $this->taxRate), 2);
    }
}
```

### Boot-Time Configuration
During the service provider boot phase, the service is still mutable and can be configured normally:
```php
public function boot(): void
{
    $payment = new PaymentService();

    // ✓ Allowed during boot
    $payment->gateway = config('payment.gateway');
    $payment->taxRate = config('payment.tax_rate');

    $this->app->singleton(PaymentService::class, fn () => $payment);
}
```
At the moment the container finishes building, Doppar automatically freezes the service.

### Runtime Behavior
Anywhere else in the application, the service is safe to read but impossible to mutate:
```php
public function checkout(PaymentService $payment)
{
    $total = $payment->calculateTotal(100); // ✓ works

    $payment->taxRate = 0.0; // ✗ ImmutableViolationException
}
```

This guarantee ensures that every part of your application sees the exact same, correctly configured payment behavior — with no risk of silent state corruption.

### The Service Lifecycle

Every frozen service passes through three phases:

**Phase 1 — Boot.** The service is constructed and configured. All properties are writable. Service providers can assign values from `config()`, `env()`, or any external source. This is the only window where mutation is permitted.

**Phase 2 — Freeze.** The container calls `freeze()` immediately after `build()` completes. Every public property's value is snapshotted into a hidden internal map and then unset from the object. After this point, all read access routes through `__get()` and all write access routes through `__set()`.

**Phase 3 — Runtime.** The frozen service is injected into controllers, closure routes, middleware, and all other consumers. Property reads work transparently and identically to before. Any write attempt on any property throws `ImmutableViolationException` with the class name and property name.

```
Boot phase      → mutable, configurable by service providers
Freeze point    → container calls freeze(), properties snapshotted
Runtime phase   → reads work normally, writes throw immediately
```

---

## Defining a Frozen Service

Add the `#[Immutable]` attribute to your class and use `EnforcesImmutability`. Declare your properties as `public` exactly as you normally would. No other changes are required on the service class itself.

```php
<?php

namespace App\Services;

use Phaseolies\DI\Attributes\Immutable;
use Phaseolies\DI\Concerns\EnforcesImmutability;

#[Immutable]
class PaymentService
{
    use EnforcesImmutability;

    public string $gateway  = 'stripe';
    public float  $taxRate  = 0.08;
    public bool   $liveMode = false;

    public function charge(float $amount): array
    {
        $tax   = round($amount * $this->taxRate, 2);
        $total = round($amount + $tax, 2);

        return [
            'gateway' => $this->gateway,
            'amount'  => $amount,
            'tax'     => $tax,
            'total'   => $total,
            'status'  => 'charged',
        ];
    }

    public function getGateway(): string
    {
        return $this->gateway;
    }

    public function isLive(): bool
    {
        return $this->liveMode;
    }
}
```

The `#[Immutable]` attribute signals intent. `EnforcesImmutability` provides the freeze mechanism. The container activates it automatically — no additional wiring is needed.

## Configuring a Frozen Service

The boot window is where you configure a frozen service with values from your environment. Inside your service provider's `boot()` method, the service is still mutable. Once you register the singleton, the container calls `freeze()` and the service is permanently locked.

The boot window is the only time configuration should occur.
```php
<?php

namespace App\Providers;

use App\Services\PaymentService;
use Phaseolies\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        $payment = new PaymentService();

        $payment->gateway  = config('payment.gateway');
        $payment->taxRate  = config('payment.tax_rate');
        $payment->liveMode = env('APP_ENV') === 'production';

        $this->app->singleton(PaymentService::class, fn() => $payment);
    }
}
```
After registration, the instance is permanently locked and reused everywhere.

After freeze, the service carries exactly the values you configured. Every subsequent injection of `PaymentService` anywhere in your application receives the same frozen, correctly-configured instance.

### Using Default Values

If your service does not require runtime configuration, default property values are captured by `freeze()` automatically. No service provider configuration is needed.

```php
#[Immutable]
class PaymentService
{
    use EnforcesImmutability;

    public string $gateway  = 'stripe';
    public float  $taxRate  = 0.08;
    public bool   $liveMode = false;
}
```

You can read the data as like normal behaviour.
```php
$payment = app(PaymentService::class);

$payment->gateway; // ✓ 'stripe'
$payment->taxRate; // ✓ 0.08
```

## Injecting Frozen Services

Frozen services are injected exactly like any other service. Type-hint them anywhere Doppar resolves dependencies and the container delivers the correctly-typed, frozen instance.

Assume we want to write the data for `$payment->taxRate` and it will cause error. You will see `ImmutableViolationException` error.

```php
use Phaseolies\Utilities\Attributes\Route;

#[Route(uri: 'charge', methods: ['POST'])]
public function process(PaymentService $payment)
{
    $currentGateway = $payment->gateway;   // ✓ reads always work
    $result = $payment->charge(100); // ✓ method calls always work

    $payment->taxRate = 0.0; // ✗ ImmutableViolationException
}
```

## Handling Mutation Attempts

When code attempts to write to a frozen service property, Doppar throws `ImmutableViolationException`. You can catch it explicitly wherever needed.

```php
use Phaseolies\DI\Exceptions\ImmutableViolationException;

#[Route(uri: 'charge', methods: ['POST'])]
public function process(PaymentService $payment): array
{
    try {
        $payment->taxRate = 0.0; // ✗ throws
    } catch (ImmutableViolationException $e) {
        return [
            'success'     => false,
            'error'       => $e->getMessage(),
            'currentRate' => $payment->taxRate, // 0.08
        ];
    }

    return ['success' => true];
}
```

The exception message is always actionable. It includes the fully-qualified class name and the property that was targeted:

```
ImmutableViolationException:
Cannot mutate property $taxRate on immutable service [App\Services\PaymentService].
Services marked #[Immutable] are frozen after instantiation.
Configure this service in a ServiceProvider during boot, not at runtime.
```

### What Is Blocked

The following operations all throw `ImmutableViolationException` on a frozen service:

```php
$payment = app(PaymentService::class); // frozen

$payment->gateway = 'paypal';   // ✗ write to declared property
$payment->taxRate = 0.0;        // ✗ write to declared property
$payment->liveMode = true;      // ✗ write to declared property
$payment->newProp = 'anything'; // ✗ dynamic property assignment
unset($payment->gateway);       // ✗ unset
clone $payment;                 // ✗ clone
```

### What Is Always Allowed

```php
echo $payment->gateway;          // ✓ read
$rate = $payment->taxRate;       // ✓ read
isset($payment->gateway);        // ✓ isset check
$payment->charge(100.00);        // ✓ method calls
$payment->isLive();              // ✓ method calls
$payment->isFrozen();            // ✓ frozen state check
```

## Checking Frozen State

Any service that extends `ImmutableService` or uses the `EnforcesImmutability` trait exposes an `isFrozen()` method. You can call it at any time to check whether the container has frozen that instance.

```php
$payment = new PaymentService();
$payment->isFrozen(); // false — constructed manually, not yet frozen

$payment = app(PaymentService::class);
$payment->isFrozen(); // true — resolved through the container
```

This is primarily useful in diagnostics, debugging, and test assertions. Production code generally does not need to check `isFrozen()` directly — if the service was resolved through the container, it is frozen.

---

## Frozen Services vs PHP `readonly`

PHP 8.2 introduced `readonly class`, which also prevents property mutation after construction. Doppar's Frozen Services and PHP's `readonly` are complementary — they solve different problems and serve different use cases.

| | `readonly class` | `#[Immutable]` Frozen Service |
| --- | --- | --- |
| **Frozen at** | Immediately on `new` — no post-construction writes ever | After container `boot()` — configurable window before freeze |
| **Configure from `config()` / `env()`** | ✗ Impossible after `new` | ✓ Standard pattern during boot |
| **ServiceProvider configuration** | ✗ Cannot assign post-construction | ✓ Designed for this |
| **Default property values** | ✓ | ✓ |
| **Constructor promotion** | ✓ | ✓ |
| **Exception on mutation** | Generic PHP `Error` | Typed `ImmutableViolationException` with class + property |
| **Works at runtime injection** | ✓ | ✓ |
| **Guard level** | PHP engine (fastest) | Object-level via `__set()` |

The critical distinction is the **boot window**. With `readonly`, all values must be supplied at the time of `new`. You cannot configure a `readonly` service from runtime configuration:

### Mutating After Singleton Registration

A common mistake is attempting to configure a frozen service after it has been registered as a singleton. By the time any code outside the service provider runs, the service is already frozen.

```php
// ✓ Correct — configure before registering
public function boot(): void
{
    $payment = new PaymentService();
    $payment->gateway = config('payment.gateway'); // writable
    $this->app->singleton(PaymentService::class, fn() => $payment);
}

// ✗ Wrong — service is already frozen when this runs
$payment = app(PaymentService::class);
$payment->gateway = 'paypal'; // ImmutableViolationException
```

### Applying #[Immutable] to Models

Doppar models are designed for mutation — `$user->name = 'Alice'` and `$user->save()` are their primary operations. The container detects model classes and bypasses the freeze mechanism for them automatically. You do not need to worry about accidentally freezing a model, but you should not apply `#[Immutable]` to model classes intentionally.

Frozen Services give your application a clear and enforced architectural contract: services are configured once during the boot phase and are then guaranteed to be in a consistent, known state for the entire request lifecycle — not by convention, but by the framework.