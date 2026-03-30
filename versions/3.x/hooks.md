---
title: Model Hooks
description: Lifecycle hooks for Doppar ORM models
meta:
- name: keywords
  content: Model Hooks, Lifecycle, before_created, after_created, before_updated, after_updated, before_deleted, after_deleted, booting, booted
---

- [Introduction](#introduction)
- [Supported Events](#supported-events)
- [Hook Registration Formats](#hook-registration-formats)
- [Array Property Hooks](#array-property-hooks)
  - [Inline Callback](#inline-callback)
  - [Class-Based Handler](#class-based-handler)
  - [Conditional Hook](#conditional-hook)
  - [Multiple Events in One Array](#multiple-events-in-one-array)
- [Attribute Based Hooks](#attribute-based-hooks)
  - [Basic Usage](#basic-usage)
  - [Multiple Events on One Method](#multiple-events-on-one-method)
  - [Multiple Hook Methods on One Model](#multiple-hook-methods-on-one-model)
  - [Conditional Attribute Hook](#conditional-attribute-hook)
  - [Mixing Attribute and Array Hooks](#mixing-attribute-and-array-hooks)
- [Lifecycle Events In Depth](#lifecycle-events-in-depth)
  - [booting and booted](#booting-and-booted)
  - [before_created and after_created](#before_created-and-after_created)
  - [before_updated and after_updated](#before_updated-and-after_updated)
  - [before_deleted and after_deleted](#before_deleted-and-after_deleted)
- [Accessing Model State Inside Hooks](#accessing-model-state-inside-hooks)
  - [Reading Attributes](#reading-attributes)
  - [Detecting Changes](#detecting-changes)
  - [Mutating Attributes](#mutating-attributes)
- [Skipping Hooks](#skipping-hooks)
- [Execution Order](#execution-order)
- [Important Constraints](#important-constraints)

## Introduction

Model Hooks in Doppar provide a clean, expressive way to respond to lifecycle events on your models. They allow you to execute custom logic automatically when specific events occur — such as when a model is being created, updated, deleted, or booted for the first time.

Hooks are the right place for cross-cutting concerns that belong close to the model but should not live inside your controllers or service classes. Common use cases include:

- Generating slugs, UUIDs, or other derived attributes before saving
- Normalising or sanitising input values before they reach the database
- Sending notifications or dispatching jobs after a record changes
- Writing audit logs that track what changed and who changed it
- Enforcing business rules that prevent certain operations
- Clearing or invalidating caches when a record is modified
- Soft-stamping timestamps like `published_at` only when a status changes

Doppar supports two independent ways to define hooks on a model, and both can be used together on the same model:

- **Array property** — the `$hooks` array defined on the model class, supporting inline callbacks, class-based handlers, and conditional execution
- **Attributes** — `#[Hook(...)]` placed directly above model methods, keeping the hook definition co-located with its implementation

## Supported Events

Every hook is tied to one of the following eight lifecycle events:

| Event | When it fires |
|---|---|
| `booting` | Before the model class is fully initialised. Fires once per class per process, not once per instance. |
| `booted` | After the model class is fully initialised. Fires once per class per process. |
| `before_created` | Just before a new record is inserted into the database. Attribute mutations made here are persisted. |
| `after_created` | Immediately after a new record has been successfully inserted. The model's primary key is available. |
| `before_updated` | Just before an existing record is updated. Attribute mutations made here are persisted. |
| `after_updated` | Immediately after an existing record has been successfully updated. |
| `before_deleted` | Just before a DELETE query runs on the record. |
| `after_deleted` | Immediately after a record has been removed from the database. |


## Hook Registration Formats

Doppar gives you two formats. You can use either one alone, or both together on the same model.

**Format 1 — Array property.** Define a `$hooks` array on the model. Each key is an event name, the value is the handler. Useful when you want to keep hook definitions centralised at the top of the model, or when using standalone hook classes.

**Format 2 — PHP Attribute.** Place `#[Hook('event_name')]` directly above any public or protected method. The method becomes the handler. Useful when you want the hook logic visible exactly where the method is defined.

Both formats support all eight lifecycle events, conditional execution, and every feature described in this document.

## Array Property Hooks

### Inline Callback

The simplest way to register a hook is to point it at a static method on the model class using the `[ClassName::class, 'methodName']` callable format. The method receives the model instance as its only parameter.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Support\Facades\Cache;

class Post extends Model
{
    protected $table     = 'posts';
    protected $creatable = ['title', 'slug', 'status', 'user_id'];

    protected array $hooks = [
        'before_created' => [self::class, 'setDefaults'],
        'after_created'  => [self::class, 'notifyFollowers'],
        'after_deleted'  => [self::class, 'clearCache'],
    ];

    public static function setDefaults(Model $model): void
    {
        if (empty($model->status)) {
            $model->status = 'draft';
        }

        if (empty($model->user_id)) {
            $model->user_id = auth()->id();
        }
    }

    public static function notifyFollowers(Model $model): void
    {
        (new NotifyFollowersJob($model->id))->dispatch();
    }

    public static function clearCache(Model $model): void
    {
        Cache::delete('posts.all');
        Cache::delete('post.' . $model->id);
    }
}
```

You can register multiple events each pointing to a different static method:

```php
protected array $hooks = [
    'before_created' => [self::class, 'generateSlug'],
    'before_updated' => [self::class, 'generateSlug'],
    'after_created'  => [self::class, 'sendWelcomeEmail'],
    'after_updated'  => [self::class, 'invalidateCache'],
    'after_deleted'  => [self::class, 'invalidateCache'],
];
```

---

### Class-Based Handler

For reusable or complex hook logic, create a dedicated hook class. Doppar provides a `make:hook` command to scaffold one quickly:

```bash
php pool make:hook PostCreatedHook
```

This generates the following class inside `App\Hooks`:

```php
<?php

namespace App\Hooks;

use Phaseolies\Database\Entity\Model;

class PostCreatedHook
{
    public function handle(Model $model): void
    {
        //
    }
}
```

Register it on the model using the fully qualified class string:

```php
<?php

namespace App\Models;

use App\Hooks\PostCreatedHook;
use App\Hooks\PostUpdatedHook;
use App\Hooks\PostDeletedHook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected $table     = 'posts';
    protected $creatable = ['title', 'slug', 'body', 'status'];

    protected array $hooks = [
        'after_created' => PostCreatedHook::class,
        'after_updated' => PostUpdatedHook::class,
        'after_deleted' => PostDeletedHook::class,
    ];
}
```

Doppar resolves the class from the container and calls `handle(Model $model)` automatically. A real-world example of a hook class:

```php
<?php

namespace App\Hooks;

use Phaseolies\Database\Entity\Model;

class PostCreatedHook
{
    public function handle(Model $model): void
    {
        foreach ($model->author->subscribers as $subscriber) {
            (new NewPostNotifyJob($subscriber, $model))->dispatch();
        }

        AuditLog::create([
            'event'      => 'post.created',
            'model_id'   => $model->id,
            'user_id'    => auth()->id(),
            'created_at' => now(),
        ]);
    }
}
```

### Conditional Hook

Use the `handler` / `when` array format to make a hook fire only when a runtime condition is met. The `when` key must point to a static method on the model that receives the model instance and returns `bool`. When it returns `false`, the handler is silently skipped.

```php
<?php

namespace App\Models;

use App\Hooks\AuditUserHook;
use Phaseolies\Database\Entity\Model;

class User extends Model
{
    protected $table     = 'users';
    protected $creatable = ['name', 'email', 'role', 'status', 'password'];

    protected array $hooks = [
        'after_updated' => [
            'handler' => AuditUserHook::class,
            'when'    => [self::class, 'shouldAudit'],
        ],
    ];

    /**
     * Only audit when sensitive fields change.
     */
    public static function shouldAudit(Model $model): bool
    {
        return $model->isDirtyAttr('email')
            || $model->isDirtyAttr('role')
            || $model->isDirtyAttr('password');
    }
}
```

You can refine conditions further — the condition method has full access to current and original state:

```php
public static function shouldAudit(Model $model): bool
{
    // Only audit changes made by administrators
    if (auth()->user()?->role !== 'admin') {
        return false;
    }

    return $model->isDirtyAttr('email') || $model->isDirtyAttr('role');
}
```

You can also use an inline callback as the `handler` inside a conditional hook:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Invoice extends Model
{
    protected $table     = 'invoices';
    protected $creatable = ['amount', 'status', 'tenant_id', 'paid_at'];

    protected array $hooks = [
        'before_created' => [
            'handler' => [self::class, 'stampTenantId'],
            'when'    => [self::class, 'isTenantScoped'],
        ],
        'before_updated' => [
            'handler' => [self::class, 'stampPaidAt'],
            'when'    => [self::class, 'isBeingPaid'],
        ],
    ];

    public static function stampTenantId(Model $model): void
    {
        $model->tenant_id = auth()->user()->tenant_id;
    }

    public static function isTenantScoped(Model $model): bool
    {
        return auth()->check() && auth()->user()->tenant_id !== null;
    }

    public static function stampPaidAt(Model $model): void
    {
        $model->paid_at = now();
    }

    public static function isBeingPaid(Model $model): bool
    {
        return $model->isDirtyAttr('status') && $model->status === 'paid';
    }
}
```

---

### Multiple Events in One Array

You can register hooks for as many events as needed in a single `$hooks` array. Each event key independently uses any handler format — inline, class-based, or conditional:

```php
<?php

namespace App\Models;

use App\Hooks\OrderCreatedHook;
use App\Hooks\OrderUpdatedHook;
use Phaseolies\Database\Entity\Model;

class Order extends Model
{
    protected $table     = 'orders';
    protected $creatable = ['reference', 'amount', 'status', 'user_id'];

    protected array $hooks = [
        // Inline callback — runs before INSERT
        'before_created' => [self::class, 'generateReference'],

        // Class-based handler — runs after INSERT
        'after_created'  => OrderCreatedHook::class,

        // Conditional with class-based handler
        // Runs after UPDATE only when status changed
        'after_updated'  => [   
            'handler' => OrderUpdatedHook::class,
            'when'    => [self::class, 'statusChanged'],
        ],

        // Inline callback — cleanup after DELETE
        'after_deleted'  => [self::class, 'cleanupOrderFiles'],
    ];

    public static function generateReference(Model $model): void
    {
        $model->reference = 'ORD-' . strtoupper(uniqid());
    }

    public static function statusChanged(Model $model): bool
    {
        return $model->isDirtyAttr('status');
    }

    public static function cleanupOrderFiles(Model $model): void
    {
        //
    }
}
```

## Attribute Based Hooks

The `#[Hook]` attribute lets you declare a model method as a lifecycle hook handler without any `$hooks` array entry. Import the attribute class at the top of your model:

```php
use Phaseolies\Database\Entity\Attributes\Hook;
```

`#[Hook]` methods run on the live model instance. Use `$this` to read and mutate attributes — no parameter is passed to the method.

### Basic Usage

Place `#[Hook('event_name')]` directly above any public or protected method:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected $table     = 'posts';
    protected $creatable = ['title', 'slug', 'body', 'status'];

    #[Hook('before_created')]
    public function generateSlug(): void
    {
        $this->slug = str()->slug($this->title);
    }

    #[Hook('before_created')]
    public function setDefaultStatus(): void
    {
        if (empty($this->status)) {
            $this->status = 'draft';
        }
    }

    #[Hook('after_created')]
    public function notifySubscribers(): void
    {
        (new NotifySubscribersJob($this->id))->dispatch();
    }

    #[Hook('after_deleted')]
    public function clearPostCache(): void
    {
        Cache::delete('post.' . $this->id);
        Cache::delete('posts.all');
    }
}
```

When `Post::create(['title' => 'Hello World'])` is called, `generateSlug()` and `setDefaultStatus()` both fire before the INSERT. The record is saved with `slug = 'hello-world'` and `status = 'draft'` without any extra code in your controller.

### Multiple Events on One Method

The `#[Hook]` attribute is repeatable. Stack multiple attributes on a single method to register it for more than one event:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class User extends Model
{
    protected $table     = 'users';
    protected $creatable = ['name', 'email', 'username', 'avatar'];

    /**
     * Normalise the email on both create and update.
     */
    #[Hook('before_created')]
    #[Hook('before_updated')]
    public function normaliseEmail(): void
    {
        $this->email = strtolower(trim($this->email));
    }

    /**
     * Generate a default username on create.
     * Re-sync it on update if the name changes.
     */
    #[Hook('before_created')]
    #[Hook('before_updated')]
    public function syncUsername(): void
    {
        if (empty($this->username) || $this->isDirtyAttr('name')) {
            $this->username = str()->slug($this->name) . '-' . rand(100, 999);
        }
    }

    /**
     * Clear the cached profile on both update and delete.
     */
    #[Hook('after_updated')]
    #[Hook('after_deleted')]
    public function clearProfileCache(): void
    {
        Cache::delete('user.profile.' . $this->id);
    }
}
```

### Multiple Hook Methods on One Model

A model can have as many `#[Hook]`-annotated methods as it needs. Each method handles one focused responsibility:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Product extends Model
{
    protected $table     = 'products';
    protected $creatable = ['name', 'slug', 'sku', 'price', 'stock', 'status'];

    #[Hook('before_created')]
    public function generateSlug(): void
    {
        $this->slug = str()->slug($this->name);
    }

    #[Hook('before_created')]
    public function normaliseSku(): void
    {
        $this->sku = strtoupper(trim($this->sku));
    }

    #[Hook('before_created')]
    public function roundPrice(): void
    {
        $this->price = round((float) $this->price, 2);
    }

    #[Hook('before_updated')]
    public function roundPriceOnUpdate(): void
    {
        if ($this->isDirtyAttr('price')) {
            $this->price = round((float) $this->price, 2);
        }
    }

    #[Hook('after_created')]
    #[Hook('after_updated')]
    public function reindexSearch(): void
    {
        (new ReindexProductJob($this->id))->dispatch();
    }

    #[Hook('before_deleted')]
    public function preventDeleteIfHasOrders(): void
    {
        if ($this->orders()->count() > 0) {
            throw new \RuntimeException(
                "Cannot delete product #{$this->id} — it has existing orders."
            );
        }
    }

    #[Hook('after_deleted')]
    public function cleanupProductAssets(): void
    {
        Cache::delete('product.' . $this->id);
    }
}
```

### Conditional Attribute Hook

Use the `when` parameter to make an attribute hook fire only when a condition is met. Pass the name of another method on the same model as a string. That method must return `bool`. When it returns `false`, the hook is silently skipped.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Article extends Model
{
    protected $table     = 'articles';
    protected $creatable = ['title', 'slug', 'body', 'status', 'published_at', 'author_id'];

    /**
     * Only stamp published_at when status changes to 'published' for the first time.
     */
    #[Hook('before_updated', when: 'isBeingPublished')]
    public function stampPublishedAt(): void
    {
        $this->published_at = now();
    }

    public function isBeingPublished(): bool
    {
        return $this->isDirtyAttr('status')
            && $this->status === 'published'
            && empty($this->published_at);
    }

    /**
     * Only notify the author when their article transitions to 'approved'.
     */
    #[Hook('after_updated', when: 'wasJustApproved')]
    public function notifyAuthorOfApproval(): void
    {
        (new ArticleApprovedNotification($this->id, $this->author_id))->dispatch();
    }

    public function wasJustApproved(): bool
    {
        return $this->isDirtyAttr('status')
            && $this->status === 'approved'
            && $this->getOriginal('status') !== 'approved';
    }

    /**
     * Regenerate slug on create only — slug stays stable once published.
     */
    #[Hook('before_created')]
    public function generateSlug(): void
    {
        $this->slug = str()->slug($this->title);
    }

    /**
     * Only reindex if the body content actually changed.
     */
    #[Hook('after_updated', when: 'bodyChanged')]
    public function reindexArticle(): void
    {
        (new ReindexArticleJob($this->id))->dispatch();
    }

    public function bodyChanged(): bool
    {
        return $this->isDirtyAttr('body');
    }

    /**
     * Only revoke sessions when the account is being banned.
     */
    #[Hook('after_updated', when: 'isBeingBanned')]
    public function revokeActiveSessions(): void
    {
        //
    }

    public function isBeingBanned(): bool
    {
        return $this->isDirtyAttr('status')
            && $this->status === 'banned'
            && $this->getOriginal('status') !== 'banned';
    }
}
```

The `when` method always runs on the same model instance, so you have full access to `$this`, `isDirtyAttr()`, `getOriginal()`, and any relationship.

### Mixing Attribute and Array Hooks

Both formats can coexist on the same model. Array hooks execute first, followed by attribute hooks. A common pattern is to use array hooks for injected class-based handlers and attribute hooks for inline per-method logic:

```php
<?php

namespace App\Models;

use App\Hooks\UserAuditHook;
use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class User extends Model
{
    protected $table     = 'users';
    protected $creatable = ['name', 'email', 'username', 'role', 'avatar', 'status'];

    /**
     * Array hook — delegates to an injectable, testable hook class.
     * Fires first, before all attribute hooks.
     */
    protected array $hooks = [
        'after_updated' => [
            'handler' => UserAuditHook::class,
            'when'    => [self::class, 'sensitiveFieldChanged'],
        ],
    ];

    public static function sensitiveFieldChanged(Model $model): bool
    {
        return $model->isDirtyAttr('email')
            || $model->isDirtyAttr('role')
            || $model->isDirtyAttr('password');
    }

    /**
     * Attribute hooks — inline logic co-located with each method.
     * Fire after array hooks.
     */
    #[Hook('before_created')]
    #[Hook('before_updated')]
    public function normaliseEmail(): void
    {
        $this->email = strtolower(trim($this->email));
    }

    #[Hook('before_created')]
    public function setDefaultRole(): void
    {
        if (empty($this->role)) {
            $this->role = 'member';
        }
    }

    #[Hook('after_created')]
    public function sendWelcomeEmail(): void
    {
        (new SendWelcomeEmailJob($this->id))->dispatch();
    }

    #[Hook('after_updated')]
    #[Hook('after_deleted')]
    public function clearUserCache(): void
    {
        Cache::delete('user.' . $this->id);
    }
}
```

## Lifecycle Events In Depth

### booting and booted

`booting` fires before a model class is fully initialised. `booted` fires after. Both run exactly once per class per PHP process — on the first instantiation of the class — and never again for any subsequent instance of the same class.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Setting extends Model
{
    protected $table = 'settings';

    #[Hook('booting')]
    public function onBooting(): void
    {
        // Runs once when this class is first instantiated in the process
        info('Setting model class is booting.');
    }

    #[Hook('booted')]
    public function onBooted(): void
    {
        // Runs once after the class is fully ready
        info('Setting model class has booted.');
    }
}
```

::: warning
Do not use `booting` or `booted` for per-record logic. They fire only for the first instantiation in a request. Use `before_created` or `after_created` for per-row operations.
:::

### before_created and after_created

`before_created` fires just before the INSERT query. Any attribute mutations you make inside this hook are included in the INSERT — the snapshot is taken after the hook runs, not before.

`after_created` fires immediately after the record was successfully inserted. The model's primary key is populated and available.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class User extends Model
{
    protected $table     = 'users';
    protected $creatable = ['name', 'email', 'username', 'uuid', 'role', 'status'];

    /**
     * Generate a unique UUID before insert.
     * The generated uuid is included in the INSERT.
     */
    #[Hook('before_created')]
    public function assignUuid(): void
    {
        //
    }

    /**
     * Normalise email to lowercase before insert.
     */
    #[Hook('before_created')]
    public function normaliseEmail(): void
    {
        $this->email = strtolower(trim($this->email));
    }

    /**
     * Set a default role if none was provided.
     */
    #[Hook('before_created')]
    public function setDefaultRole(): void
    {
        if (empty($this->role)) {
            $this->role = 'member';
        }
    }

    /**
     * After the INSERT, $this->id is now available.
     */
    #[Hook('after_created')]
    public function sendVerificationEmail(): void
    {
        (new SendEmailVerificationJob($this->id, $this->email))->dispatch();
    }

    #[Hook('after_created')]
    public function createDefaultProfile(): void
    {
        Profile::create([
            'user_id' => $this->id,
            'avatar'  => 'default-avatar.png',
        ]);
    }
}
```

Using the array format for the same model:

```php
protected array $hooks = [
    'before_created' => [self::class, 'assignUuid'],
    'after_created'  => [self::class, 'sendVerificationEmail'],
];

public static function assignUuid(Model $model): void
{
    //
}

public static function sendVerificationEmail(Model $model): void
{
    (new SendEmailVerificationJob($model, $model))->dispatch();
}
```

::: warning
`before_created` and `after_created` are only triggered by `save()` on a new model instance, `create()`, `updateOrCreate()` when it performs an insert, and `firstOrCreate()` when it creates a new record.

They are **not** triggered by `Post::query()->insert([...])`.
:::

---

### before_updated and after_updated

`before_updated` fires just before the UPDATE query. Attribute mutations made inside this hook are included in the UPDATE — the dirty snapshot is taken after the hook runs. `after_updated` fires after the record was successfully updated.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected $table     = 'posts';
    protected $creatable = ['title', 'slug', 'body', 'status', 'published_at', 'updated_by'];

    /**
     * Regenerate slug whenever the title changes.
     */
    #[Hook('before_updated', when: 'titleChanged')]
    public function regenerateSlug(): void
    {
        $this->slug = str()->slug($this->title);
    }

    public function titleChanged(): bool
    {
        return $this->isDirtyAttr('title');
    }

    /**
     * Stamp published_at the first time status becomes 'published'.
     */
    #[Hook('before_updated', when: 'isBeingPublished')]
    public function stampPublishedAt(): void
    {
        $this->published_at = now();
    }

    public function isBeingPublished(): bool
    {
        return $this->isDirtyAttr('status')
            && $this->status === 'published'
            && $this->getOriginal('status') !== 'published';
    }

    /**
     * Track who performed the last update.
     */
    #[Hook('before_updated')]
    public function trackUpdatedBy(): void
    {
        $this->updated_by = auth()->id();
    }

    /**
     * Clear caches after every update.
     */
    #[Hook('after_updated')]
    public function invalidateCaches(): void
    {
        Cache::delete('post.' . $this->id);
        Cache::delete('posts.latest');
    }

    /**
     * Only reindex search when content actually changes.
     */
    #[Hook('after_updated', when: 'contentChanged')]
    public function triggerSearchReindex(): void
    {
        (new ReindexPostJob($this->id))->dispatch();
    }

    public function contentChanged(): bool
    {
        return $this->isDirtyAttr('title') || $this->isDirtyAttr('body');
    }
}
```

Using the array format with a class-based handler:

```php
<?php

namespace App\Models;

use App\Hooks\PostUpdatedHook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected array $hooks = [
        'after_updated' => [
            'handler' => PostUpdatedHook::class,
            'when'    => [self::class, 'contentChanged'],
        ],
    ];

    public static function contentChanged(Model $model): bool
    {
        return $model->isDirtyAttr('title') || $model->isDirtyAttr('body');
    }
}
```

```php
<?php

namespace App\Hooks;

use Phaseolies\Database\Entity\Model;

class PostUpdatedHook
{
    public function handle(Model $model): void
    {
        AuditLog::create([
            'model'     => 'Post',
            'model_id'  => $model->id,
            'changes'   => json_encode($model->getDirtyAttributes()),
            'user_id'   => auth()->id(),
        ]);
    }
}
```

::: warning
`before_updated` and `after_updated` are only triggered when updating via `save()` on an existing model instance, the model's `update()` method, or `updateOrCreate()` when it performs an update.

They are **not** triggered by `User::query()->where('id', $id)->update([...])`.
:::

---

### before_deleted and after_deleted

`before_deleted` fires just before the DELETE query runs. `after_deleted` fires after the record has been removed. Model attributes are still accessible in both hooks.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected $table     = 'posts';
    protected $creatable = ['title', 'slug', 'body', 'status'];

    /**
     * Block deletion of published posts.
     */
    #[Hook('before_deleted')]
    public function preventDeleteIfPublished(): void
    {
        if ($this->status === 'published') {
            throw new \RuntimeException(
                "Post #{$this->id} is published and cannot be deleted. Unpublish it first."
            );
        }
    }

    /**
     * Block deletion if comments exist.
     */
    #[Hook('before_deleted')]
    public function preventDeleteIfHasComments(): void
    {
        $count = Comment::query()->where('post_id', $this->id)->count();

        if ($count > 0) {
            throw new \RuntimeException(
                "Post #{$this->id} has {$count} comment(s). Delete them before deleting the post."
            );
        }
    }

    /**
     * Clean up uploaded files and caches after deletion.
     */
    #[Hook('after_deleted')]
    public function cleanupRelatedAssets(): void
    {
        Cache::delete('post.' . $this->id);
        Cache::delete('posts.all');
        Cache::delete('posts.latest');
    }

    /**
     * Write an audit trail after deletion.
     */
    #[Hook('after_deleted')]
    public function logDeletion(): void
    {
        AuditLog::create([
            'event'    => 'post.deleted',
            'model_id' => $this->id,
            'title'    => $this->title,
            'user_id'  => auth()->id(),
        ]);
    }
}
```

Using the array format:

```php
protected array $hooks = [
    'before_deleted' => [self::class, 'preventDeleteIfPublished'],
    'after_deleted'  => [self::class, 'cleanupRelatedAssets'],
];

public static function preventDeleteIfPublished(Model $model): void
{
    if ($model->status === 'published') {
        throw new \RuntimeException('Published posts cannot be deleted.');
    }
}

public static function cleanupRelatedAssets(Model $model): void
{
    Cache::delete('post.' . $model->id);
}
```

::: warning
`before_deleted` and `after_deleted` are only triggered when deleting via the model instance's `delete()` method.

They are **not** triggered by `Post::query()->where('id', $id)->delete()`.
:::

## Accessing Model State Inside Hooks

Inside any hook you have full access to the model's current state, its original state before the operation began, and which attributes have changed.

### Reading Attributes

```php
// Current value via magic getter
$this->name
$this->email
$this->status

// All current attributes as array
$this->getAttributes()

// Single original attribute (value before the current operation)
$this->getOriginal('name')

// All original attributes as array
$this->getOriginalAttributes()
```

### Detecting Changes

```php
// true if the attribute differs from its original value
$this->isDirtyAttr('email')

// all changed attributes as key => new_value pairs
$this->getDirtyAttributes()
// returns: ['email' => 'new@example.com', 'role' => 'admin']
```

Combining these to build a precise per-field audit hook:

```php
#[Hook('after_updated')]
public function writeAuditLog(): void
{
    foreach ($this->getDirtyAttributes() as $field => $newValue) {
        AuditLog::create([
            'model'     => static::class,
            'model_id'  => $this->id,
            'field'     => $field,
            'old_value' => $this->getOriginal($field),
            'new_value' => $newValue,
            'user_id'   => auth()->id(),
            'ip'        => request()->ip(),
            'at'        => now(),
        ]);
    }
}
```

Detecting a specific state transition:

```php
#[Hook('after_updated', when: 'statusChangedToBanned')]
public function revokeUserSessions(): void
{
    (new NotifyUserBannedJob($this->id))->dispatch();
}

public function statusChangedToBanned(): bool
{
    return $this->isDirtyAttr('status')
        && $this->status === 'banned'
        && $this->getOriginal('status') !== 'banned';
}
```

### Mutating Attributes

In `before_created` and `before_updated` hooks, you can freely mutate `$this` attributes. The database snapshot is taken **after** the hook runs, so your changes are always persisted:

```php
#[Hook('before_created')]
public function prepareForInsert(): void
{
    $this->slug       = str()->slug($this->title);
    $this->price      = round((float) $this->price, 2);
    $this->email      = strtolower(trim($this->email));
    $this->created_by = auth()->id();
}
```
All of the above will be included in the INSERT.

::: warning
Attribute mutations in `after_created` and `after_updated` hooks are **not** automatically persisted. The INSERT or UPDATE has already executed. If you need to persist further changes after the fact, call `$this->save()` explicitly — but use `withoutHook()` to avoid triggering hooks recursively.
:::

::: warning
Inside `#[Hook]` attribute methods, use `$this` to access the model. Inside array-based inline callbacks (`[self::class, 'method']`) and class-based `handle()` methods, the model is passed as the `$model` parameter — `$this` is not the model.
:::

## Skipping Hooks

Sometimes you need to save, update, or delete a model without triggering any hooks — for example during data migrations, bulk imports, system writes, or seeding. Doppar provides `withoutHook()` for this.

#### Skip hooks on create

```php
// New instance
$user = new User();
$user->name  = 'Imported User';
$user->email = 'import@example.com';
$user->withoutHook()->save();

// Via static create
User::withoutHook()->create([
    'name'  => 'Imported User',
    'email' => 'import@example.com',
]);
```

#### Skip hooks on update via save()

```php
$user = User::find($id);
$user->name = 'Updated Name';
$user->withoutHook()->save();
```

#### Skip hooks on update via update()

```php
$user = User::find($id);
$user->withoutHook()->update(['name' => 'Updated Name']);
```

#### Skip hooks on updateOrCreate

```php
User::withoutHook()->updateOrCreate(
    ['email' => 'user@example.com'],
    ['name'  => 'Updated Name', 'role' => 'admin']
);
```

#### Skip hooks on delete

```php
$post = Post::find($id);
$post->withoutHook()->delete();

// Or in one line
User::withoutHook()->find($id)->delete();
```

::: warning
`withoutHook()` disables **all** hooks for that single operation chain. Subsequent operations on other model instances are not affected.
:::

## Execution Order

When a lifecycle event fires, hooks run in this fixed order:

1. **Array `$hooks` property handlers** — in the order they are defined in the array
2. **Attribute `#[Hook]` handlers** — in the order the annotated methods appear in the class file

Within each group, the same handler cannot be registered twice for the same event. Doppar automatically prevents duplicate registration and silently ignores any repeat.

```php
class Post extends Model
{
    // Executes first (array hook)
    protected array $hooks = [
        'before_created' => [self::class, 'stepOne'],
    ];

    // Executes second (first attribute hook in the file)
    #[Hook('before_created')]
    public function stepTwo(): void { ... }

    // Executes third (second attribute hook in the file)
    #[Hook('before_created')]
    public function stepThree(): void { ... }
}
```


## Important Constraints

**Hooks are triggered by model methods only**

Hooks fire when you use the model's own persistence methods. Direct entity builder calls bypass all hooks entirely.

```php
// Hooks fire
Post::create(['title' => 'Hello']);
$post->save();
$post->update(['title' => 'Updated']);
$post->delete();
User::withoutHook()->create(['name' => 'Import']);

// Hooks do NOT fire
Post::query()->insert(['title' => 'Hello']);
Post::query()->where('id', 1)->update(['title' => 'Updated']);
Post::query()->where('id', 1)->delete();
Post::query()->whereIn('id', [1, 2, 3])->delete();
```

**Attribute hooks use `$this`, array hooks receive `$model`**

```php
// Attribute hook — instance method, no parameter
#[Hook('before_created')]
public function generateSlug(): void
{
    $this->slug = str()->slug($this->title); // $this is the model
}

// Array hook — static method, model passed as parameter
protected array $hooks = [
    'before_created' => [self::class, 'generateSlug'],
];

public static function generateSlug(Model $model): void
{
    $model->slug = str()->slug($model->title); // use $model, not $this
}
```

**`when` on `#[Hook]` must be a method name string**

The `when` parameter accepts only a string — the name of another instance method on the same model. It cannot be a closure or a callable array. The named method must return exactly `bool`.

```php
// Correct
#[Hook('before_updated', when: 'isPublished')]
public function doSomething(): void { ... }

public function isPublished(): bool
{
    return $this->status === 'published';
}

// Wrong — closures cannot be passed as PHP attribute values
#[Hook('before_updated', when: fn() => true)]
public function doSomething(): void { ... }

// Wrong — when method must return bool, not string or int
public function isPublished(): string { return 'yes'; }
```

**`booting` and `booted` fire once per class, not once per instance**

These are class-level bootstrap events. They fire on the first instantiation of the class within a PHP process. All subsequent instantiations skip them entirely. Do not rely on them for per-record logic.

**`before_*` mutations are persisted, `after_*` mutations are not**

Attribute changes made in `before_created` and `before_updated` hooks are included in the INSERT or UPDATE because the snapshot is taken after hooks run. Attribute changes made in `after_created` and `after_updated` are **not** automatically persisted — the database operation has already completed.

**Duplicate hook prevention**

Doppar automatically prevents the same handler from being registered twice for the same event on the same model. This applies to array hooks, attribute hooks, and any combination of both. You do not need to guard against double registration.

**Reflection runs once per class**

For `#[Hook]` attribute hooks, Doppar uses PHP Reflection to scan model methods. This scan runs exactly once per class per process and the result is cached in a static property on the `Model` class. There is no performance cost for models hydrated in bulk or large collections.