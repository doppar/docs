---
title: Model Hooks
description: Lifecycle hooks for Doppar ORM models
meta:
- name: keywords
  content: Model Hooks, Lifecycle, before_created, after_created, before_updated, after_updated, before_deleted, after_deleted, before_restored, after_restored, booting, booted, withoutHook, withHooks
---

## Hooks
### Introduction
Model hooks let you run code at specific moments in the life of a model: just before it is inserted, right after it is updated, when it is deleted, when it is restored, and so on. You attach small pieces of logic to those moments, and Doppar runs them for you every time the model is saved through its own methods.

Hooks are the right place for rules that belong close to the model but should not be repeated in every controller or service. Typical examples are building a slug from a title, setting a default status, clearing a cache after a delete, or writing an audit record.

Doppar gives you two ways to define hooks, and both can be used on the same model:

| **Style**         | **How it looks**                              | **Best for**                                    |
| ----------------- | --------------------------------------------- | ----------------------------------------------- |
| **Attribute**     | `#[Hook('before_created')]` above a method    | Short logic that lives next to the model        |
| **Array property**| `protected $hooks = [...]` on the model       | Reusable handler classes and static callbacks   |

A first example, using the attribute style:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    protected $creatable = ['title', 'slug', 'status'];

    #[Hook('before_created')]
    public function generateSlug(): void
    {
        $this->slug = str()->slug($this->title);
    }
}
```

Now every post created through the model gets its slug automatically:

```php
Post::create(['title' => 'Hello World']);
// saved with slug = 'hello-world'
```

## Supported Events
Every hook is tied to one of the following ten lifecycle events:

| **Event**         | **When it fires**                                                                              | **Changes are saved?** |
| ----------------- | ---------------------------------------------------------------------------------------------- | ---------------------- |
| `booting`         | Before the model class is initialised. Once per class per process, not once per record.       | No                     |
| `booted`          | After the model class is initialised. Once per class per process.                             | No                     |
| `before_created`  | Just before a new record is inserted.                                                          | Yes                    |
| `after_created`   | Right after the insert. The primary key is available.                                         | No                     |
| `before_updated`  | Just before an existing record is updated.                                                     | Yes                    |
| `after_updated`   | Right after the update.                                                                        | No                     |
| `before_deleted`  | Just before the record is deleted.                                                             | -                      |
| `after_deleted`   | Right after the record is removed, or soft-deleted on a `#[SoftDeletes]` model.               | -                      |
| `before_restored` | Just before a soft-deleted record is restored. `#[SoftDeletes]` models only.                  | -                      |
| `after_restored`  | Right after the record is restored. `#[SoftDeletes]` models only.                              | -                      |

"Changes are saved" means that attribute changes you make inside the hook are written to the database. That is only true for `before_created` and `before_updated`, because Doppar takes its snapshot after those hooks have run.

## When Hooks Run
Hooks belong to the model's own methods. The table shows which call fires which events:

| **Call**                                            | **Events fired**                                  |
| --------------------------------------------------- | ------------------------------------------------- |
| `Post::create([...])`                               | `before_created`, `after_created`                 |
| `$post->save()` on a new model                      | `before_created`, `after_created`                 |
| `$post->save()` on an existing model                | `before_updated`, `after_updated`                 |
| `$post->update([...])`                              | `before_updated`, `after_updated`                 |
| `$post->delete()`                                   | `before_deleted`, `after_deleted`                 |
| `$post->forceDelete()`                              | `before_deleted`, `after_deleted`                 |
| `$post->restore()`                                  | `before_restored`, `after_restored`               |
| `Post::updateOrCreate([...], [...])`                | The insert or the update events, whichever runs   |
| `Post::firstOrCreate([...], [...])`                 | The insert events, only when it creates a record  |

Query builder writes are single SQL statements and do not involve a model, so they skip hooks by default:

| **Call**                                            | **Hooks run?**                                    |
| --------------------------------------------------- | ------------------------------------------------- |
| `Post::query()->insert([...])`                      | No                                                |
| `Post::query()->where(...)->update([...])`          | No                                                |
| `Post::query()->where(...)->delete()`               | No                                                |
| `Post::where(...)->withHooks()->update([...])`      | Yes, for every matching record                    |

You can opt in with [`withHooks()`](#running-hooks-on-bulk-queries-with-withhooks) when you need hooks on a bulk query.

## Defining Hooks

### Attribute Based Hooks
The `#[Hook]` attribute declares a model method as a hook handler. Import it at the top of your model:

```php
use Phaseolies\Database\Entity\Attributes\Hook;
```

The method runs on the live model, so use `$this` to read and change attributes. It can be `public` or `protected`, and it receives no parameters.

#### Basic usage
```php
#[Hook('before_created')]
public function setDefaultStatus(): void
{
    if (empty($this->status)) {
        $this->status = 'draft';
    }
}
```

#### Several hooks on one model
A model can have as many hook methods as it needs. Give each method one focused job:

```php
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

#[Hook('after_deleted')]
public function clearPostCache(): void
{
    Cache::delete('post.' . $this->id);
    Cache::delete('posts.all');
}
```

#### One method for several events
The attribute is repeatable. Stack it to register a single method for more than one event:

```php
#[Hook('before_created')]
#[Hook('before_updated')]
public function normaliseEmail(): void
{
    $this->email = strtolower(trim($this->email));
}
```

#### Conditional hooks with `when`
Add `when:` to run a hook only if a condition is true. It takes the **name** of another method on the same model, as a string. That method must return `bool`. When it returns `false`, the hook is skipped.

```php
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
```

The `when` method runs on the same model instance, so it can use `$this`, `isDirtyAttr()`, `getOriginal()` and relationships.

> `when` cannot be a closure, because PHP attributes cannot hold one. If the method does not exist or does not return `bool`, Doppar throws a `RuntimeException` instead of guessing.

### Array Property Hooks
The other style is a `$hooks` property that maps each event to a handler. Handlers in the array are not tied to `$this`. They receive the model as a parameter.

> Declare the property **without a type**. `protected $hooks = [...]` works. `protected array $hooks = [...]` causes a fatal error, because the base `Model` class declares `$hooks` without a type and PHP does not allow a child class to add one.

#### Static method on the model
Point an event at a static method with the `[ClassName::class, 'method']` format. The method receives the model as its only argument:

```php
protected $hooks = [
    'before_created' => [self::class, 'setDefaults'],
];

public static function setDefaults(Model $model): void
{
    if (empty($model->status)) {
        $model->status = 'draft';
    }
}
```

#### Hook class
For reusable or larger logic, put the handler in its own class. Doppar can scaffold one for you:

```bash
php pool make:hook PostCreatedHook
```

This creates the following class inside `App\Hooks`:

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

Register it on the model with the class name:

```php
use App\Hooks\PostCreatedHook;

protected $hooks = [
    'after_created' => PostCreatedHook::class,
];
```

Doppar resolves the class from the container, so you can inject dependencies through its constructor, and then calls `handle(Model $model)`.

#### Conditional handler
Use the `handler` and `when` keys to run a handler only when a condition holds. `when` points to a static method on the model that receives the model and returns `bool`:

```php
protected $hooks = [
    'after_updated' => [
        'handler' => AuditUserHook::class,
        'when'    => [self::class, 'shouldAudit'],
    ],
];

public static function shouldAudit(Model $model): bool
{
    return $model->isDirtyAttr('email')
        || $model->isDirtyAttr('role');
}
```

The `handler` can also be a static method instead of a class:

```php
protected $hooks = [
    'before_updated' => [
        'handler' => [self::class, 'stampPaidAt'],
        'when'    => [self::class, 'isBeingPaid'],
    ],
];
```

### Mixing Both Styles
Both styles can live on the same model. A common split is to use the array for class-based handlers and attributes for small inline logic:

```php
<?php

namespace App\Models;

use App\Hooks\UserAuditHook;
use Phaseolies\Database\Entity\Attributes\Hook;
use Phaseolies\Database\Entity\Model;

class User extends Model
{
    protected $creatable = ['name', 'email', 'role'];

    protected $hooks = [
        'after_updated' => UserAuditHook::class,
    ];

    #[Hook('before_created')]
    #[Hook('before_updated')]
    public function normaliseEmail(): void
    {
        $this->email = strtolower(trim($this->email));
    }
}
```

### Execution Order
When an event fires, hooks run in a fixed order:

| **Order** | **Handlers**                                                          |
| --------- | --------------------------------------------------------------------- |
| 1         | Array `$hooks` handlers, in the order they appear in the array        |
| 2         | `#[Hook]` methods, in the order they appear in the class              |

The same handler cannot be registered twice for the same event. Doppar silently ignores a repeat, so you do not need to guard against it.

```php
class Post extends Model
{
    protected $hooks = [
        'before_created' => [self::class, 'stepOne'],   // runs first
    ];

    #[Hook('before_created')]
    public function stepTwo(): void {}                  // runs second

    #[Hook('before_created')]
    public function stepThree(): void {}                // runs third
}
```

## Events One by One
Each event below has one small example. All of them work the same way in the array style, only the handler signature changes.

### booting
Fires before the model class is initialised, once per class per process. Use it for class-level setup, not for per-record work.

```php
#[Hook('booting')]
public function onBooting(): void
{
    info('Setting model is booting.');
}
```

### booted
Fires after the model class is initialised, once per class per process.

```php
#[Hook('booted')]
public function onBooted(): void
{
    info('Setting model has booted.');
}
```

> `booting` and `booted` fire on the first use of the class, then never again in the same process. Use `before_created` and friends for logic that must run for every record.

### before_created
Fires just before the INSERT. Changes you make here are saved, which makes it the right place for defaults and generated values.

```php
#[Hook('before_created')]
public function prepareForInsert(): void
{
    $this->slug  = str()->slug($this->title);
    $this->email = strtolower(trim($this->email));
}
```

### after_created
Fires right after the INSERT. The primary key is now available.

```php
#[Hook('after_created')]
public function createDefaultProfile(): void
{
    Profile::create([
        'user_id' => $this->id,
        'avatar'  => 'default-avatar.png',
    ]);
}
```

### before_updated
Fires just before the UPDATE. Changes you make here are saved. Use `isDirtyAttr()` to react only to specific changes.

```php
#[Hook('before_updated', when: 'titleChanged')]
public function regenerateSlug(): void
{
    $this->slug = str()->slug($this->title);
}

public function titleChanged(): bool
{
    return $this->isDirtyAttr('title');
}
```

### after_updated
Fires right after the UPDATE. The original values are still available through `getOriginal()`, which makes it a good place for audit records and cache clearing.

```php
#[Hook('after_updated')]
public function invalidateCaches(): void
{
    Cache::delete('post.' . $this->id);
    Cache::delete('posts.latest');
}
```

### before_deleted
Fires just before the delete. Throw an exception to stop it.

```php
#[Hook('before_deleted')]
public function preventDeleteIfPublished(): void
{
    if ($this->status === 'published') {
        throw new \RuntimeException(
            "Post #{$this->id} is published and cannot be deleted. Unpublish it first."
        );
    }
}
```

### after_deleted
Fires right after the record is removed. Attributes are still readable.

```php
#[Hook('after_deleted')]
public function logDeletion(): void
{
    AuditLog::create([
        'event'    => 'post.deleted',
        'model_id' => $this->id,
        'user_id'  => auth()->id(),
    ]);
}
```

On a model marked `#[SoftDeletes]`, `delete()` soft-deletes the record and both delete events still fire. They also fire for `forceDelete()`. Call `$this->isForceDeleting()` inside the hook to tell a permanent delete from a soft one:

```php
#[Hook('after_deleted')]
public function removeFiles(): void
{
    if ($this->isForceDeleting()) {
        @unlink($this->cover_path);
    }
}
```

### before_restored
Fires just before a soft-deleted record is restored. Throw an exception to stop the restore. `#[SoftDeletes]` models only.

```php
#[Hook('before_restored')]
public function preventRestoreIfArchived(): void
{
    if ($this->status === 'archived') {
        throw new \RuntimeException("Post #{$this->id} is archived and cannot be restored.");
    }
}
```

### after_restored
Fires once the soft delete column has been cleared. `#[SoftDeletes]` models only.

```php
#[Hook('after_restored')]
public function refreshCache(): void
{
    Cache::delete('posts.all');
    Cache::delete('posts.latest');
}
```

See [`Soft Deletes`](/versions/4.x/soft-deletes) for the full soft delete API.

## Accessing Model State Inside Hooks
Inside any hook you can read the model's current state, its original state before the operation, and what changed.

| **Method**                       | **Returns**                                                        |
| -------------------------------- | ------------------------------------------------------------------ |
| `$this->name`                    | The current value of an attribute                                  |
| `$this->getAttributes()`         | All current attributes as an array                                 |
| `$this->getOriginal('name')`     | The value of one attribute before the current operation            |
| `$this->getOriginalAttributes()` | All original attributes as an array                                |
| `$this->isDirtyAttr('email')`    | `true` if the attribute differs from its original value            |
| `$this->getDirtyAttributes()`    | The changed attributes as `key => new_value` pairs                 |

Writing an audit entry for each changed field:

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
        ]);
    }
}
```

Reacting to a transition, not just a change:

```php
#[Hook('after_updated', when: 'wasJustApproved')]
public function notifyAuthor(): void
{
    (new ArticleApprovedNotification($this->id, $this->author_id))->dispatch();
}

public function wasJustApproved(): bool
{
    return $this->isDirtyAttr('status')
        && $this->status === 'approved'
        && $this->getOriginal('status') !== 'approved';
}
```

### Before and after are not the same
Changes made in `before_created` and `before_updated` are saved, because the snapshot is taken after your hook runs. Changes made in `after_created` and `after_updated` are **not** saved, because the query has already run.

If you need to save more after the fact, call `save()` again with hooks switched off for that save. Otherwise you would trigger the hooks again:

```php
#[Hook('after_created')]
public function stampCode(): void
{
    $this->code = 'CODE-' . $this->id;

    $this->withoutHook()->save();
}
```

## Cancelling an Operation
A `before_*` hook can stop the operation by throwing an exception. The exception reaches the code that called `save()`, `update()`, `delete()` or `restore()`, and nothing is written.

```php
try {
    $post->delete();
} catch (\RuntimeException $e) {
    return back()->with('error', $e->getMessage());
}
```

Returning `false` from a hook does not cancel anything. Throw instead.

## Skipping Hooks with withoutHook()
Some writes should not run your hooks: data migrations, bulk imports, seeders, system fixes, or a hook that needs to save its own model again. `withoutHook()` switches hooks off for **one write**.

It works in two places, and the difference matters:

| **Called on**                  | **Returns**          | **Effect**                                                                        |
| ------------------------------ | -------------------- | --------------------------------------------------------------------------------- |
| A model instance               | The same instance    | Skips hooks for the next write on that instance                                   |
| The model class (`User::...`)  | A query builder      | Models the builder creates or loads skip hooks for their next write              |

Two things hold in both cases:

- **It lasts for one write.** After `save()`, `update()`, `delete()` or `restore()` finishes, hooks are back on for that model.
- **It never leaks.** The switch lives on the model or the builder you called it on. Other models and other queries are not affected, even if the call is followed by a plain read.

### On a model instance
Because the instance is returned, you can chain the write straight onto it. Each call below touches only that one record.

#### Save
```php
$user = User::find($id);
$user->name = 'Updated Name';
$user->withoutHook()->save();
```

#### Update
```php
$user = User::find($id);
$user->withoutHook()->update(['name' => 'Updated Name']);
```

#### Delete
```php
$post = Post::find($id);
$post->withoutHook()->delete();
```

#### Create from a new instance
```php
$user = new User();
$user->name  = 'Imported User';
$user->email = 'import@example.com';
$user->withoutHook()->save();
```

### On the model class
The class form is the right choice when you create or look up the record in the same expression.

#### Create
```php
User::withoutHook()->create([
    'name'  => 'Imported User',
    'email' => 'import@example.com',
]);
```

#### Find and delete
```php
User::withoutHook()->find($id)->delete();
```

#### Update or create
```php
User::withoutHook()->updateOrCreate(
    ['email' => 'user@example.com'],
    ['name'  => 'Updated Name', 'role' => 'admin']
);
```

#### First or create
```php
User::withoutHook()->firstOrCreate(
    ['email' => 'user@example.com'],
    ['name'  => 'New User']
);
```

#### Restore
```php
Post::withoutHook()->withTrashed()->find($id)->restore();
```

On a `#[SoftDeletes]` model, `withoutHook()` still soft-deletes. It only skips the hooks:

```php
Post::withoutHook()->find($id)->delete();   // soft-deleted, no hooks
```

> A model loaded through `User::withoutHook()->find($id)` carries the switch with it. If you change that model and call `save()` later, that first save also skips hooks.

### Use cases
| **Situation**                                   | **Why `withoutHook()` helps**                                          |
| ----------------------------------------------- | ---------------------------------------------------------------------- |
| Importing records from another system           | Skip slug generation, notifications and cache work for every row       |
| Seeders and test data                           | Avoid sending emails or dispatching jobs while seeding                 |
| Saving again from inside an `after_*` hook      | Prevents the hook from triggering itself in a loop                     |
| Housekeeping writes, such as a remember token   | Avoid audit records for changes nobody made                            |
| Repairing bad data                              | Fix values without running business rules a second time                |

## Running Hooks on Bulk Queries with withHooks()
A query builder write such as `User::query()->where(...)->update([...])` is one SQL statement. No model is involved, so no hooks run. That is the fast path, and it is the right default.

When you need hooks, watches and temporal snapshots to run for every affected record, add `withHooks()` to the query:

```php
User::where('status', 'pending')->withHooks()->update(['status' => 'active']);
```

Doppar loads the matching records in chunks of 500, ordered by primary key, and calls each model's own `update()`, `delete()`, `restore()` or `forceDelete()`. Your hooks see a real model, exactly as they do for a single record.

| **Call after `withHooks()`** | **Runs for each record**              | **Notes**                                             |
| ---------------------------- | ------------------------------------- | ----------------------------------------------------- |
| `update([...])`              | `before_updated`, `after_updated`     | Follows the model's `$creatable` list                 |
| `delete()`                   | `before_deleted`, `after_deleted`     | Soft-deletes on a `#[SoftDeletes]` model              |
| `restore()`                  | `before_restored`, `after_restored`   | `#[SoftDeletes]` models only                          |
| `forceDelete()`              | `before_deleted`, `after_deleted`     | `#[SoftDeletes]` models only                          |

### Ways to use it
Each example below changes only the call after `withHooks()`.

#### Update many records
```php
User::where('status', 'pending')
    ->withHooks()
    ->update(['status' => 'active']);
```

#### Update a list of ids
```php
Order::whereIn('id', [10, 11, 12])
    ->withHooks()
    ->update(['status' => 'shipped']);
```

#### Delete many records
```php
Post::where('author_id', $authorId)
    ->withHooks()
    ->delete();
```

#### Restore trashed records
```php
Post::onlyTrashed()
    ->where('author_id', $authorId)
    ->withHooks()
    ->restore();
```

#### Permanently delete
```php
Post::where('archived', true)
    ->withHooks()
    ->forceDelete();
```

> `restore()` and `forceDelete()` are available on models marked `#[SoftDeletes]`. On any other model they throw a `LogicException`, the same as without `withHooks()`.

#### All or nothing, inside a transaction
Records are written one by one, and there is no transaction around them. If a hook throws, records handled before it stay changed. Wrap the call in a transaction when you need all or nothing:

```php
DB::transaction(function () {
    User::where('status', 'pending')
        ->withHooks()
        ->update(['status' => 'active']);
});
```

#### Combined with withoutHook()
You can combine both. The records are still written one by one, but without any hooks. This is rarely what you want, so pick one unless you have a reason:

```php
User::withoutHook()
    ->where('status', 'pending')
    ->withHooks()
    ->update(['status' => 'active']);
```

### What to know before using it
| **Point**                       | **Detail**                                                                                              |
| ------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Speed                           | Every record gets its own query. Use it when you need the hooks, not by default.                        |
| `$creatable`                    | `update()` ignores columns that are not in the model's `$creatable` list, like `$user->update([...])`.   |
| No implicit transaction         | A throwing hook stops the run. Earlier records stay changed and later records are not touched.          |
| Rows that stop matching         | Not skipped. The query walks the primary key, so changing the column used in `where` is safe.           |
| `orderBy()` and `limit()`       | Replaced by the chunking while the query runs.                                                          |

## Choosing Between the Three
| **You want to...**                                         | **Use**                                              |
| ---------------------------------------------------------- | ---------------------------------------------------- |
| Write one record and run its hooks                         | `$post->save()`, `$post->update([...])`              |
| Write one record and skip its hooks                        | `$post->withoutHook()->save()`                       |
| Create or find a record and skip hooks                     | `Post::withoutHook()->create([...])`                 |
| Change many rows as fast as possible, no hooks             | `Post::where(...)->update([...])`                    |
| Change many rows and run hooks for each                    | `Post::where(...)->withHooks()->update([...])`       |

## Hooks, Watches and Temporal Snapshots
Hooks are the base for two other features, and `withoutHook()` and `withHooks()` affect them in the same way:

| **Feature**                                       | **Runs with hooks?** | **Skipped by `withoutHook()`?** |
| ------------------------------------------------- | -------------------- | ------------------------------- |
| Property watches (`#[Watches]`)                   | Yes                  | Yes                             |
| Temporal snapshots (history records)              | Yes                  | Yes                             |

Use `withoutHook()` only when you intentionally want to skip them as well. See [`Reactive Model`](/versions/4.x/reactive-model) and [`Temporal ORM`](/versions/4.x/temporal-orm) for details.

## Important Constraints

#### Attribute hooks use `$this`, array hooks receive `$model`
Inside a `#[Hook]` method, `$this` is the model. Inside a static callback or a `handle()` method, the model is the `$model` parameter and `$this` is not the model.

```php
#[Hook('before_created')]
public function generateSlug(): void
{
    $this->slug = str()->slug($this->title);
}
```

Here `$this` is the model.

```php
protected $hooks = [
    'before_created' => [self::class, 'generateSlug'],
];

public static function generateSlug(Model $model): void
{
    $model->slug = str()->slug($model->title);
}
```
Use `$model`, not `$this` this case.

#### `when` on `#[Hook]` must be a method name
It accepts only a string, the name of another method on the same model, and that method must return `bool`.

Correct one.
```php
#[Hook('before_updated', when: 'isPublished')]
public function doSomething(): void {}

public function isPublished(): bool
{
    return $this->status === 'published';
}
```

Wrong, you can't use closures as attribute values:
```php
#[Hook('before_updated', when: fn() => true)]
public function doSomething(): void {}
```

#### `$hooks` must not have a type
Write `protected $hooks = [...]`. Adding `array` is a fatal error.

#### `booting` and `booted` run once per class
They fire on the first use of the class in a process. Do not rely on them for per-record logic.

#### Duplicate registration is ignored
The same handler is never registered twice for the same event. This applies to array hooks, attribute hooks and any mix of both.

#### Reflection runs once per class
For `#[Hook]` attributes, Doppar scans the model's methods with Reflection once per class per process and caches the result. Loading large collections of models does not repeat the scan.
