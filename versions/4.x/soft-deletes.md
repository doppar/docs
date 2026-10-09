---
title: Soft Deletes
description: Doppar ORM soft deletes documentation
meta:
- name: keywords
  content: Soft Deletes, SoftDeletes, ORM, withTrashed, onlyTrashed, restore, forceDelete, deleted_at
---

## Soft Deletes

### Introduction

Sometimes a record should disappear from your application without actually being removed from the database. A user deletes a post but may want it back, an admin needs to review removed accounts, or a compliance rule requires that nothing is ever physically erased. Soft deletes solve this: instead of running a `DELETE`, Doppar stamps a timestamp column on the row, and from then on every query on that model quietly skips it.

Soft deletes are opt-in per model through the `#[SoftDeletes]` attribute. Once a model carries the attribute, `delete()` marks the row as deleted, every query excludes trashed rows by default, and you get a small API for including them again, restoring them, or removing them for good.

The constraint is applied consistently across the ORM. Regular queries, aggregates, pagination, eager loading, and relationship existence queries such as `present()` and `whereLinked()` all skip trashed rows unless you explicitly ask for them. Soft deletes also integrate with Doppar's [`Model Hooks`](/versions/4.x/hooks) and [`Temporal ORM`](/versions/4.x/temporal-orm), so deletions and restorations can be observed and recorded in the model's history.

Soft deletes are particularly useful when your application needs a recycle bin, an undo action, audit-friendly deletion, or a grace period before data is permanently removed.

No extra code is required in your controllers or services. Your existing `delete()` calls keep working, they simply stop removing rows.

## Marking a Model as Soft-Deletable

Add the `#[SoftDeletes]` attribute to the model class:

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Entity\Attributes\SoftDeletes;

#[SoftDeletes]
class Post extends Model
{
    protected $table = 'posts';

    protected $creatable = ['title', 'body', 'user_id'];

    protected $timeStamps = true;
}
```

By default Doppar uses a `deleted_at` column. To use a different column, pass its name:

```php
#[SoftDeletes(column: 'removed_at')]
class Note extends Model
{
    //
}
```

That single attribute is everything Doppar needs. No trait to pull in, no global scope to register — the framework handles the rest automatically.

> **Note:** The attribute is inherited, so putting it on a base model makes every child model soft-deletable as well.

## Preparing the Table

The soft delete column must be a nullable timestamp column. The migration `Blueprint` has a helper for it:

```php
public function up(): void
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('body');
        $table->timestamps();
        $table->softDeletes();
    });
}
```

If you need custom column name
```php
$table->softDeletes('removed_at');
```

timezone aware variant
```php
$table->softDeletesTz();
```

To add the column to an existing table, and drop it again on rollback:

```php
public function up(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->softDeletes();
    });
}

public function down(): void
{
    Schema::table('posts', function (Blueprint $table) {
        $table->dropSoftDeletes();
    });
}
```

A row whose soft delete column is `NULL` is live. A row with a timestamp in it is trashed.

## Soft Deleting Models

Call `delete()` exactly as you would on any other model:

```php
$post = Post::find(1);

$post->delete();
```

Instead of removing the row, Doppar sets `deleted_at` to the current time (and touches `updated_at` when the model uses timestamps). The in-memory model is updated with the same values, so there is no need to reload it:

```php
$post->trashed();    // true
$post->deleted_at;   // "2026-10-09 10:15:00"
```

Deleting through the query builder soft-deletes every matching row as well:

Soft-deletes every draft post
```php
Post::query()->where('status', 'draft')->delete();
```

Soft-deletes the posts with these primary keys
```php
Post::purge(1, 2, 3);
```

> **Note:** Bulk soft deletes only touch rows that are still live, so a row that is already trashed keeps its original `deleted_at` timestamp. Calling `delete()` on a single model that is already trashed stamps it again with the current time.

## Querying Soft-Deleted Models

Every query on a soft-deletable model excludes trashed rows automatically. This applies everywhere the model is queried: `get()`, `first()`, `find()`, `count()` and other aggregates, `paginate()`, cursor pagination, chunking, bulk `update()`, `increment()` and so on.

null when post 1 is trashed
```php
Post::find(1);
```

live posts only
```php
Post::where('user_id', 5)->count();
```

### Including Trashed Models

Use `withTrashed()` to include trashed rows in the results:

```php
$post = Post::withTrashed()->find(1);
```

### Retrieving Only Trashed Models

Use `onlyTrashed()` to retrieve trashed rows and nothing else. This is handy for a "recycle bin" screen:

```php
$trash = Post::onlyTrashed()
    ->where('user_id', auth()->id())
    ->orderBy('deleted_at', 'desc')
    ->get();
```

### Going Back to the Default

`withoutTrashed()` switches a query back to the default behaviour of excluding trashed rows. It is useful when a query object is built up in steps:

```php
$query = Post::onlyTrashed();

if ($request->has('live')) {
    $query->withoutTrashed();
}

$posts = $query->get();
```

### OR Conditions Are Safe

The soft delete constraint is added after your own conditions are wrapped in parentheses, so an `orWhere()` can never bring trashed rows back by accident:

```php
Post::where('status', 'draft')->orWhere('status', 'review')->toSql();

// SELECT * FROM posts WHERE (status = ? OR status = ?) AND posts.deleted_at IS NULL
```

Nested `where` groups do not repeat the constraint. It is applied once, on the outer query. Table aliases are respected too, so `Post::query()->from('posts as p')` qualifies the column as `p.deleted_at`.

## Restoring Soft-Deleted Models

Call `restore()` on a trashed model to bring it back. Doppar sets the soft delete column back to `NULL` and, when the model uses timestamps, touches `updated_at`:

```php
$post = Post::withTrashed()->find(1);

$post->restore();

$post->trashed(); // false
```

To restore many rows at once, call `restore()` on a query. A restore query targets trashed rows only, so you don't need to add `onlyTrashed()` yourself:

Restore every trashed post of a user
```php
Post::where('user_id', 5)->restore();
```

Live rows that happen to match the query are left untouched, so their `updated_at` does not change.

## Permanently Deleting Models

When a row really has to go, use `forceDelete()`. It runs a real `DELETE`, whether the model is live or already trashed:

```php
$post = Post::withTrashed()->find(1);

$post->forceDelete();
```

`forceDelete()` is available on queries too:

```php
Post::onlyTrashed()->forceDelete();
```

> **Note:** A query's `forceDelete()` only removes the rows the query matches, and a query skips trashed rows by default. `Post::where('user_id', 5)->forceDelete()` therefore leaves that user's trashed posts in place. Add `withTrashed()` to remove live and trashed rows together, or `onlyTrashed()` to remove trashed rows only.

Permanently remove old trashed posts
```php
Post::onlyTrashed()
    ->where('deleted_at', '<', now()->subDays(30))
    ->forceDelete();
```

## Relationships

Trashed rows are hidden from relationships in the same way they are hidden from regular queries.

### Loading Relations

Lazy loading and eager loading with `embed()` skip trashed related models, including nested relations and many-to-many relations through a pivot table:

live posts only
```php
$user = User::find(1);

$user->posts;

$users = User::query()->embed('posts.comments')->get();
```

### Relationship Existence Queries

`present()`, `absent()`, `ifExists()` and `whereLinked()` ignore trashed related rows, so a user whose only post is trashed counts as having no posts:

Users with at least one live post
```php
User::query()->present('posts')->get();
```

Users with no live posts
```php
User::query()->absent('posts')->get();
```

Users with a live post titled "Hello"
```php
User::query()->whereLinked('posts', 'title', 'Hello')->get();
```

This also works through nested relations (`present('posts.comments')`) and many-to-many relations, where a trashed row anywhere in the chain is ignored.

To change this, call `withTrashed()` or `onlyTrashed()` inside the callback:

Users with any post, live or trashed
```php
User::query()->present('posts', fn($query) => $query->withTrashed())->get();
```

Users who have at least one trashed post
```php
User::query()->present('posts', fn($query) => $query->onlyTrashed())->get();
```

## Hooks

A soft delete fires the usual `before_deleted` and `after_deleted` hooks. A permanent delete through `forceDelete()` fires them as well. Inside the hook, `isForceDeleting()` tells you which one is happening:

```php
use Phaseolies\Database\Entity\Attributes\Hook;

#[SoftDeletes]
class Post extends Model
{
    #[Hook('after_deleted')]
    public function cleanUp(): void
    {
        if ($this->isForceDeleting()) {
            // The row is gone for good: remove uploaded files
            Storage::delete($this->cover_path);
        }
    }
}
```

Restoring a model fires two new events, `before_restored` and `after_restored`. Throwing an exception from a `before_restored` hook stops the restore:

```php
#[Hook('before_restored')]
public function preventRestoreIfArchived(): void
{
    if ($this->status === 'archived') {
        throw new \RuntimeException("Post #{$this->id} is archived and cannot be restored.");
    }
}

#[Hook('after_restored')]
public function clearCache(): void
{
    Cache::delete('posts.all');
}
```

> `before_deleted`, `after_deleted`, `before_restored` and `after_restored` are only triggered by the model instance's `delete()`, `forceDelete()` and `restore()` methods. They are not triggered by bulk queries such as `Post::query()->where(...)->delete()` or `Post::query()->where(...)->restore()`.

`withoutHook()` still soft-deletes, it only skips the hooks:

soft-deleted, no hooks
```php
Post::withoutHook()->find($id)->delete();
```

restored, no hooks
```php
Post::withoutHook()->withTrashed()->find($id)->restore();
```

See [`Model Hooks`](/versions/4.x/hooks) for more about hooks.

## Temporal ORM Integration

Soft deletes work together with the [`Temporal ORM`](/versions/4.x/temporal-orm). When a model carries both `#[SoftDeletes]` and `#[Temporal]`:

- A soft delete is recorded as a `deleted` snapshot, and a restore is recorded as a new `restored` snapshot.
- Time-travel queries such as `::at()` treat the record as absent while it was trashed and present again after it was restored.
- `restoreTo()` carries the soft delete state of the snapshot. Restoring a trashed record to a moment before it was deleted brings it back as a live record.

```php
#[Temporal]
#[SoftDeletes]
class Contract extends Model
{
    //
}

$contract = Contract::withTrashed()->find(42);

// Roll back to the state before it was deleted; the contract is live again
$contract->restoreTo('2026-01-01 12:00:00');
```

## Utility Methods

### `trashed()`

Returns `true` if the model has been soft-deleted.

```php
$post = Post::withTrashed()->find(1);

if ($post->trashed()) {
    echo 'This post is in the recycle bin.';
}
```

### `isForceDeleting()`

Returns `true` while `forceDelete()` is running. Use it inside `before_deleted` and `after_deleted` hooks to tell a permanent delete from a soft one.

```php
#[Hook('after_deleted')]
public function cleanUp(): void
{
    if ($this->isForceDeleting()) {
        Storage::delete($this->cover_path);
    }
}
```

### `usesSoftDeletes()`

Returns `true` if the model class is marked with `#[SoftDeletes]`.

```php
$post = new Post();

if ($post->usesSoftDeletes()) {
    echo 'This model uses soft deletes.';
}
```

### `getSoftDeleteColumn()`

Returns the name of the soft delete column, or `null` if the model is not marked with `#[SoftDeletes]`.

```php
echo (new Post())->getSoftDeleteColumn(); // 'deleted_at'
echo (new Note())->getSoftDeleteColumn(); // 'removed_at'
```

## Driver Support

Soft deletes work with all three database drivers supported by Doppar: MySQL, PostgreSQL, and SQLite. The constraint is plain `IS NULL` / `IS NOT NULL` SQL, so no driver-specific configuration is needed.

## Important Notes

### Soft delete methods require the attribute

Calling `withTrashed()`, `onlyTrashed()`, `withoutTrashed()`, `restore()` or `forceDelete()` on a model without `#[SoftDeletes]` throws a `LogicException`. Models without the attribute are not affected by soft deletes at all, and their `delete()` removes the row as before.

### A trashed model can still be saved

Writes that target a single model by its primary key (`save()`, `update()`, `increment()`, `decrement()`, `delete()`) ignore the soft delete constraint. You can load a trashed model with `withTrashed()`, change it and save it. It stays trashed:

```php
$post = Post::withTrashed()->find(1);
$post->title = 'Archived: ' . $post->title;
$post->save();   // saved, still trashed
```

### Raw SQL is not scoped

The constraint is added by the model's query builder. Raw queries run through `DB::sql()` do not know about soft deletes, so add `deleted_at IS NULL` yourself when you need it.
