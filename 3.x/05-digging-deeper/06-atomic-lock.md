---
title: Atomic Lock
description: Doppar Atomic Lock page
meta:
  - name: keywords
    content: Atomic Lock, Cache
---

## Atomic Lock
### Introduction
The Atomic Lock feature in the Doppar Framework provides a powerful mechanism for managing synchronization and concurrency within your application. It ensures that a specific resource or process is accessed by only one execution thread (or request) at a time — preventing race conditions, inconsistent states, and data corruption.

Atomic locks are built on doppar default caching abstraction, which integrates with Symfony’s cache adapters. This allows locks to be distributed across multiple servers while remaining efficient and consistent.

In simpler terms, Atomic Lock ensures that only one worker can perform a specific task at a given time — all others must wait or fail gracefully.

> Your selected cache driver will be used for application locking mechanism.

## Why Atomic Lock

In concurrent systems, it’s common for multiple processes to attempt the same operation simultaneously — for example:

- Processing the same queue job multiple times.
- Updating a shared cache entry.
- Running a scheduled task that should never overlap.
- Performing critical database migrations or imports.

Without synchronization, these operations can cause data inconsistencies or duplicate work. Atomic Locks solve this problem by introducing a mutual exclusion mechanism:

- When a lock is acquired, other processes trying to acquire the same lock will be blocked until it’s released or expired.
- Locks are automatically released after a given time-to-live (TTL) to prevent deadlocks.
- Ownership verification ensures that only the process that created a lock can release it.

### Doppar’s Atomic Lock ensures:

- Atomicity — lock acquisition and release are atomic operations.
- Distributed Safety — locks work across multiple nodes or workers.
- Automatic Expiration — locks expire after the configured duration.
- Restorable Ownership — previously acquired locks can be safely restored and validated.

## Features (at-a-glance)

### Atomic Locking
* **Owner-based locking**: locks store an owner identifier (PID + random bytes) so only the creator may release them.
* **Blocking and non-blocking modes**:
    * Non-blocking: try once and return success/failure immediately.
    * Blocking: retry acquisition until timeout with configurable backoff (supports exponential backoff).
* **Configurable TTL**: every lock has a duration; if it is not refreshed or released it will expire automatically to prevent deadlocks.

### Safety & Reliability
* **Stale lock detection & cleanup**: automatically detects expired/stale locks and allows safe reclamation.
* **Process-safe unique owner IDs**: uses a combination of process ID (PID) and random bytes to ensure uniqueness across processes and servers.
* **Shutdown hooks & destructors** are used to attempt clean release of locks during normal process termination.
* **Fail-safe expiry** prevents permanent deadlocks when processes die unexpectedly.

### Lock Recovery
* **Safe restoration** after restarts: you can restore a lock only if ownership data matches and the lock has not expired.
* **Ownership validation** prevents unauthorized restoration or takeover of locks.
* **Expiration checks** ensure only valid, not-yet-expired locks can be recovered.

## Basic Usage
The Doppar Atomic Lock ensures exclusive access to critical sections by leveraging atomic cache operations.
Here’s a simple example using the `locked()` method.
```php
use Phaseolies\Support\Facades\Cache;

$lock = Cache::locked('foo', 10);

if ($lock->get()) {
    // Performing important operations
    $lock->release();
} else {
    // Could not acquire lock
}

```
This acquires a lock named `foo` for `10` seconds, executes protected logic, and then releases it.
If another process already holds the lock, it immediately fails to acquire it.

## Blocking Locks
When multiple processes may contend for the same resource, you can use `blocking mode` to wait until a lock becomes available rather than failing immediately.
```php
use Phaseolies\Support\Facades\Cache;

$lock = Cache::locked('report:daily', 10);

if ($lock->block(5)) { // Wait up to 5 seconds for the lock
    try {
        // Perform your critical or exclusive operation
        $this->processOrder();
    } finally {
        $lock->release();
    }
} else {
    // Could not acquire the lock within the timeout
    // Skipped process order — another process holds the lock.
}
```

This waits for up to `5` seconds to acquire the lock before proceeding. If the lock is acquired, only this process will execute the protected operation for the next 10 seconds (or until it’s released).

## Lock Restoration
Doppar supports safe lock restoration, allowing a process to regain ownership of a previously acquired lock after a crash, restart, or delayed execution.

Ownership validation ensures that only the original owner can restore the lock, and expired locks cannot be restored.
```php
use Phaseolies\Support\Facades\Cache;

// Original process acquires a lock
$originalLock = Cache::locked('important-resource', 10);

if ($originalLock->get()) {
    $ownerId = $originalLock->getOwner();

    // Later, restore the lock using the same owner ID
    $restoredLock = Cache::restoreLock('important-resource', $ownerId);

    // Validate ownership
    if ($restoredLock->isOwnedByCurrentProcess()) {
        $this->performCriticalTask();
        $restoredLock->release();
    } else {
        // Cannot restore lock: ownership mismatch.
    }
} else {
    // Could not acquire lock initially.
}
```

This `restoreLock()` checks the cached lock data to ensure the stored owner matches the provided owner ID.

## Ownership Validation
Each lock created by Doppar includes a unique owner identifier — a combination of process ID (PID) and random bytes.
This guarantees that only the original owner can release or restore the lock, protecting against accidental or malicious unlock attempts.
```php
$lock = Cache::locked('resource', 60);

if ($lock->get()) {
    echo "Lock owner: " . $lock->owner() . "\n";
    echo "Is owned by current process: " . $lock->isOwnedByCurrentProcess() . "\n";

    // Do work...

    $lock->release();
}
```

The `release()` internally checks if the current process’s owner ID matches the stored one. If not, Doppar refuses the release to prevent cross-process interference.

## Lock Feature Testing
This example demonstrates basic lock acquisition, blocking behavior, and release/re-acquire mechanics using Doppar’s atomic locks.
```php
use Phaseolies\Support\Facades\Cache;

echo "Testing Lock Feature:\n";

// Test 1: Basic lock acquisition
$lock1 = Cache::locked('test-lock', 5);
if ($lock1->get()) {
    echo "✓ Lock acquired successfully\n";

    // Test 2: Cannot acquire same lock twice
    $lock2 = Cache::locked('test-lock', 5);
    if (!$lock2->get()) {
        echo "✓ Second lock attempt correctly failed\n";
    }

    // Test 3: Release and re-acquire
    $lock1->release();
    if ($lock2->get()) {
        echo "✓ Lock re-acquired after release\n";
        $lock2->release();
    }
} else {
    echo "✗ Failed to acquire lock\n";
}

// Test 4: Blocking lock
echo "Testing blocking lock (will wait up to 2 seconds):\n";
$lock3 = Cache::locked('blocking-test', 10);
if ($lock3->block(2)) {
    echo "✓ Blocking lock acquired\n";
    $lock3->release();
} else {
    echo "✗ Blocking lock timeout\n";
}

echo "All tests completed!\n";
```

This test setup is ideal for verifying atomic locking behavior, blocking operations, and proper release handling.