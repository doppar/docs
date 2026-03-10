---
title: Authorization
description: Doppar Authorization
meta:
  - name: keywords
    content: Authorization
---

## Guard - Authorization

### Introduction

Guard is a robust and extensible authorization system designed for the Doppar framework. It provides a clean, expressive API for defining and evaluating user abilities across your application. With features like one-time temporary permissions, ability inheritance, grouped permissions, wildcard patterns, ability aliases, conditional runtime gates, and global lifecycle hooks, Guard gives you complete control over access logic without cluttering your codebase.

Guard is built to empower your Doppar applications with secure, scalable, and maintainable authorization logic. Whether you're building small tools or large systems, Guard adapts to your needs with clarity and power.

## Installation

You may install Doppar Guard via the composer require command:

```bash
composer require doppar/guard
```

Doppar does not support package auto-discovery, so you need to manually register the service provider to enable the `Guard` facade. In your `config/app.php`, add the following to the providers array:

```php
'providers' => [
    // Other service providers...
    \Doppar\Authorizer\GuardServiceProvider::class,
],
```

Once installed and registered, the `Guard` facade will be available for use, giving you access to a rich set of tools for managing authorization.

## Basic Ability Checks

Define abilities in your `App\Providers\AppServiceProvider` using `Guard::define`. Each ability receives a unique name and a closure containing the authorization logic. The closure receives the currently authenticated user as its first argument.

```php
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::define('edit-settings', function ($user) {
        return $user->isAdmin;
    });
}
```

After defining your abilities, evaluate them at runtime using `Guard::allows` and `Guard::denies`:

```php
if (Guard::allows('edit-settings')) {
    // User is allowed to edit settings
}

if (Guard::denies('edit-settings')) {
    // User is not allowed to edit settings
}
```

`Guard::allows` returns `true` if the current user has the given ability. `Guard::denies` is the exact inverse — it returns `true` when the user does not have the ability.

You can also check multiple abilities at once using `Guard::any`, which returns `true` if at least one ability passes:

```php
if (Guard::any(['edit-settings', 'admin-access'])) {
    // User has at least one of these abilities
}
```

To require all abilities to pass, use `Guard::all`:

```php
if (Guard::all(['edit-settings', 'manage-users'])) {
    // User has every ability in the list
}
```

You can also use `auth()->can()` anywhere in your application to check the currently authenticated user's abilities:

```php
if (auth()->can('edit-settings')) {
    // User is allowed to edit application settings
}
```

## Authorization with Odo Directives

Guard integrates directly with Odo, Doppar's templating engine, through the `#scope` and `#scopenot` directives. These allow you to conditionally render frontend elements based on user abilities, keeping your templates clean and expressive.

The `#scope` directive renders its contents only when the user has the given ability:

```html
#scope('edit-settings')
<a href="/settings/edit">Edit Settings</a>
#elsescope('store-settings')
<a href="/settings/store">Store Settings</a>
#else
<span>No settings access.</span>
#endscope
```

You can pass a model or any additional arguments to perform more precise checks:

```html
#scope('edit-settings', $user)
<a href="/settings/edit">Edit</a>
#endscope
```

The `#scopenot` directive is the inverse — it renders when the user does not have the given ability:

```html
#scopenot('edit-settings')
<span>Access restricted.</span>
#elsescopenot('store-settings')
<span>Store access restricted.</span>
#else
<span>You have full access.</span>
#endscopenot
```

## Defining Permissions

You can define fine-grained authorization rules using `Guard::define` with model arguments. The closure receives the authenticated user as the first argument, followed by any models or values passed during the check.

```php
use App\Models\Post;
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::define('update-post', function (User $user, Post $post) {
        return (int) $user->id === (int) $post->user_id;
    });
}
```

The ability compares the authenticated user's `id` against the post's `user_id`. Only the post's owner will be authorized.

To check this ability in a controller, pass the model as the second argument to `Guard::allows`:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/update', methods: ['POST'])]
    public function update(Request $request): RedirectResponse
    {
        $post = Post::find(1);

        if (! Guard::allows('update-post', $post)) {
            abort(403);
        }

        // Update the post...

        return redirect('/posts');
    }
}
```

If the user does not own the post, `Guard::allows` returns `false` and the request is aborted with a 403 response.

### Odo View Integration

Wrap rows or UI elements in `#scope` to show them only to authorized users:

```html
#forelse ($posts as $item) #scope('update-post', $item)
<tr>
  <td>[[ $item->id ]]</td>
  <td>[[ $item->user_id ]]</td>
  <th scope="row">[[ $item->title ]]</th>
</tr>
#endscope #empty #endforelse
```

Only users who own each post will see its row in the table.

## Writing Authorizers

For applications with many model-level permissions, Guard supports dedicated Authorizer classes. This keeps your authorization logic modular, testable, and easy to maintain as your application scales.

Generate an Authorizer class using the `make:authorizer` command:

```bash
php pool make:authorizer PostAuthorizer --model=Post
```

This generates the following file:

```php
<?php

namespace App\Authorizers;

use App\Models\User;
use App\Models\Post;

class PostAuthorizer
{
    public function viewAny(User $user): bool
    {
        //
    }

    public function view(User $user, Post $post): bool
    {
        //
    }

    public function create(User $user): bool
    {
        //
    }

    public function update(User $user, Post $post): bool
    {
        //
    }

    public function delete(User $user, Post $post): bool
    {
        //
    }
}
```

Each method corresponds to a permission action. Implement the `update` method to check ownership:

```php
public function update(User $user, Post $post): bool
{
    return (int) $user->id === (int) $post->user_id;
}
```

Register the Authorizer class with Guard in your `AppServiceProvider`:

```php
use App\Authorizers\PostAuthorizer;
use App\Models\Post;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::authorize(Post::class, PostAuthorizer::class);
}
```

Guard now uses `PostAuthorizer` for all permission checks involving `Post` models. Check permissions in controllers using the method name as the ability:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/update', methods: ['POST'])]
    public function update(Request $request): RedirectResponse
    {
        $post = Post::find(1);

        if (! Guard::allows('update', $post)) {
            abort(403);
        }

        // Update the post...

        return redirect('/posts');
    }
}
```

The ability name `'update'` maps directly to the `update` method on `PostAuthorizer`. Odo templates work the same way:

```html
#scope('update', $post)
<a href="/posts/[[ $post->id ]]/edit">Edit Post</a>
#endscope
```

## Temporary Abilities

Temporary abilities are valid for a single evaluation only. After the first check they are automatically removed, making them ideal for sensitive one-time actions like email confirmation, terms acceptance, or onboarding steps.

```php
Guard::temporary('one-time-access', function ($user) {
    return true;
});
```

The first call to `Guard::allows` consumes the ability and returns `true`. Every subsequent call returns `false`:

```php
if (Guard::allows('one-time-access')) {
    // Passes — ability is consumed here
}

if (Guard::denies('one-time-access')) {
    // Passes — ability no longer exists
}
```

## Ability Inheritance

Guard supports ability inheritance, allowing a parent ability to pass if any of its registered child abilities pass. This is useful for modelling roles where access is determined by a combination of specific permissions.

Define the child abilities first, then register the parent with `Guard::inherit`:

```php
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::define('manage-users',    fn($user) => $user->isAdmin);
    Guard::define('manage-settings', fn($user) => $user->isAdmin);

    Guard::inherit('admin', ['manage-users', 'manage-settings']);
}
```

Checking the parent ability returns `true` if any child ability passes:

```php
if (Guard::allows('admin')) {
    // Passes if the user can manage users or manage settings
}
```

You can also check child abilities directly:

```php
if (Guard::allows('manage-users')) {
    // Passes independently
}
```

Retrieve the registered children of any parent ability using `getChildren`:

```php
$children = Guard::getChildren('admin');
// Returns: ['manage-users', 'manage-settings']
```

## Ability Grouping

Ability groups let you logically organize related abilities under a named group for reporting, permission dashboards, or conditional UI logic. Groups do not affect how abilities are evaluated — they are a metadata layer only.

```php
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::group('content-management', [
        'create-post',
        'edit-post',
        'delete-post',
    ]);
}
```

Check whether a given ability belongs to a group using `Guard::inGroup`:

```php
if (Guard::inGroup('content-management', 'edit-post')) {
    // Returns true — 'edit-post' is part of 'content-management'
}

if (Guard::inGroup('content-management', 'manage-users')) {
    // Returns false — 'manage-users' is not in this group
}
```

## Global Before and After Callbacks

### Before Callbacks

The `before` callback runs before any ability is evaluated. If it returns a boolean, Guard short-circuits immediately and skips all further checks — including defined abilities, policies, and hierarchy. If it returns `null`, normal evaluation continues.

This is the standard pattern for super-admin access:

```php
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::before(function ($user, $ability) {
        if ($user->isSuperAdmin) {
            return true; // Super admins bypass everything
        }

        return null; // Everyone else goes through normal checks
    });
}
```

### After Callbacks

The `after` callback runs after every authorization check, regardless of the result and regardless of whether a `before` callback short-circuited the process. It receives the user, the ability name, and the final boolean result. It cannot change the result — it is intended for logging, metrics, and auditing.

```php
Guard::after(function ($user, $ability, $result) {
    logAccessAttempt($user, $ability, $result);
});
```

## Ability Aliases

Ability aliases let you register alternative names that resolve to existing abilities. When an alias is checked, Guard follows the alias to the real ability name and evaluates it normally — including all arguments, hooks, policies, and hierarchy. The alias is completely transparent to every part of the system.

This is useful when you want to provide a shorthand name, preserve backwards compatibility after renaming an ability, or map multiple verb styles to one canonical definition.

Register an alias with `Guard::alias` in your `AppServiceProvider`:

```php
use App\Models\Post;
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::define('update-post', function (User $user, Post $post) {
        return (int) $user->id === (int) $post->user_id;
    });

    Guard::alias('edit', 'update-post');
}
```

Use the alias anywhere you would use the real ability name:

```php
if (Guard::allows('edit', $post)) {
    // Resolved to 'update-post' internally
}

if (auth()->can('edit', $post)) {
    // Works identically with auth()->can()
}
```

In a controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/update', methods: ['POST'])]
    public function update(Request $request): RedirectResponse
    {
        $post = Post::find(1);

        if (! Guard::allows('edit', $post)) {
            abort(403);
        }

        // Update the post...

        return redirect('/posts');
    }
}
```

In an Odo template:

```html
#scope('edit', $post)
<a href="/posts/[[ $post->id ]]/edit">Edit Post</a>
#endscope #scopenot('edit', $post)
<span>You cannot edit this post.</span>
#endscopenot
```

### Chained Aliases

Aliases can point to other aliases. Guard follows the entire chain until it reaches a real ability name:

```php
Guard::define('update-post', fn(User $user) => $user->isEditor);
Guard::alias('edit', 'update-post');
Guard::alias('modify', 'edit'); // modify -> edit -> update-post

// Evaluates 'update-post' after following the full chain
Guard::allows('modify');
```

Circular chains are detected automatically and return `false` without looping.

### Aliases with Authorizer Classes

Aliases resolve before policy lookups, so they work with Authorizer classes too:

```php
public function boot(): void
{
    Guard::authorize(Post::class, PostAuthorizer::class);

    // 'save' resolves to PostAuthorizer::update()
    Guard::alias('save', 'update');
}
```

```php
if (Guard::allows('save', $post)) {
    // PostAuthorizer::update() was called
}
```

### Inspecting Registered Aliases

```php
$aliases = Guard::aliases();
```

Output:

```php
[
    'edit' => 'update-post',
    'modify' => 'edit',
    'save' => 'update'
]
```

## Wildcard Abilities

Wildcard abilities let you define one authorization rule that covers an entire family of related ability names. The `*` character acts as a single dot-segment placeholder. Guard matches concrete ability names against registered wildcard patterns automatically when no exact match is found.

This eliminates the need to register `post.create`, `post.edit`, `post.delete`, and `post.publish` separately when the logic is identical.

Register a wildcard ability with `Guard::wildcard` in your `AppServiceProvider`:

```php
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::wildcard('post.*', function (User $user) {
        return $user->role === 'editor';
    });
}
```

### Pattern Reference

| Pattern     | Matches                                       | Does Not Match           |
| ----------- | --------------------------------------------- | ------------------------ |
| `post.*`    | `post.create`, `post.edit`, `post.delete`     | `comment.create`, `post` |
| `*.create`  | `post.create`, `comment.create`, `tag.create` | `post.edit`, `create`    |
| `admin.*.*` | `admin.users.delete`, `admin.settings.edit`   | `admin.single`, `admin`  |
| `*`         | Every ability name                            | —                        |

Each `*` matches one or more non-dot characters within a single segment. It does not span across dots.

Check wildcard-covered abilities using the concrete ability name — no changes are needed at the call site:

```php
Guard::allows('post.create');   // Matched by 'post.*'
Guard::allows('post.edit');     // Matched by 'post.*'
Guard::allows('post.delete');   // Matched by 'post.*'
Guard::allows('post.publish');  // Matched by 'post.*'
```

In a controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/store', methods: ['POST'])]
    public function store(Request $request): RedirectResponse
    {
        if (! Guard::allows('post.create')) {
            abort(403);
        }

        // Create the post...

        return redirect('/posts');
    }

    #[Route(uri: 'post/delete', methods: ['POST'])]
    public function destroy(Request $request): RedirectResponse
    {
        $post = Post::find($request->id);

        if (! Guard::allows('post.delete', $post)) {
            abort(403);
        }

        // Delete the post...

        return redirect('/posts');
    }
}
```

In Odo templates:

```html
#scope('post.create')
<a href="/posts/create">New Post</a>
#endscope #forelse ($posts as $post) #scope('post.edit')
<tr>
  <td>[[ $post->id ]]</td>
  <td>[[ $post->title ]]</td>
  <td><a href="/posts/[[ $post->id ]]/edit">Edit</a></td>
</tr>
#endscope #empty #endforelse
```

### Exact Abilities Take Precedence

When both an exact ability and a wildcard pattern match a given name, the exact ability is always evaluated first. The wildcard is only reached as a fallback. This lets you carve out exceptions inside a broad rule:

```php
Guard::define('post.delete', fn(User $user) => false);  // Exact: always deny
Guard::wildcard('post.*',    fn(User $user) => true);   // Wildcard: always allow

Guard::allows('post.delete'); // false — exact match wins
Guard::allows('post.edit');   // true  — no exact match, wildcard applies
```

### Wildcard Abilities with Model Arguments

Wildcard callbacks receive the same arguments as any `Guard::define` callback, allowing ownership checks across an entire family of abilities:

```php
use App\Models\Post;
use App\Models\User;

Guard::wildcard('post.*', function (User $user, Post $post) {
    return (int) $user->id === (int) $post->user_id;
});
```

```php
Guard::allows('post.edit',   $post); // Ownership check
Guard::allows('post.delete', $post); // Ownership check
```

### Suffix Wildcard

A suffix wildcard is useful when you want to gate a specific action type across all resources:

```php
Guard::wildcard('*.delete', function (User $user) {
    return $user->isAdmin;
});

Guard::allows('post.delete');    // Checks isAdmin
Guard::allows('comment.delete'); // Checks isAdmin
Guard::allows('user.delete');    // Checks isAdmin
```

## Conditional Abilities

A conditional ability attaches a runtime gate to any existing ability. The condition is a zero-argument callable that returns a boolean. If it returns `false`, access is denied immediately — the ability callback, policy, and hierarchy are never reached, regardless of who the user is.

This is the right tool when access depends on application-level state that has nothing to do with the user or the model directly — things like feature flags, time windows, environment restrictions, or billing state.

Register a condition with `Guard::condition` alongside the ability in your `AppServiceProvider`. The two are registered independently — you can add a condition to any existing ability without modifying its callback:

```php
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::condition('ai-assistant', function () {
        return config('features.ai_enabled');
    });

    Guard::define('ai-assistant', function (User $user) {
        return $user->plan === 'pro';
    });
}
```

Even if the user is on the pro plan, `Guard::allows('ai-assistant')` returns `false` when the feature flag is off.

### How the Condition Is Evaluated

When `Guard::allows('ai-assistant')` is called:

1. Global `before` callbacks run first and can still override everything
2. The condition callable is invoked with no arguments
3. If the condition returns `false`, access is denied immediately and `after` callbacks fire with `$result = false`
4. If the condition returns `true`, normal evaluation continues

### Feature Flag Gate

```php
public function boot(): void
{
    Guard::condition('beta-dashboard', fn() => config('features.beta_dashboard'));
    Guard::define('beta-dashboard', fn(User $user) => true);
}
```

Toggling `features.beta_dashboard` in your configuration immediately affects every `Guard::allows`, `auth()->can`, and `#scope` check throughout the application. No code changes required.

In a controller:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class DashboardController extends Controller
{
    #[Route(uri: 'dashboard/beta', methods: ['GET'])]
    public function beta(Request $request): RedirectResponse
    {
        if (! Guard::allows('beta-dashboard')) {
            abort(403);
        }

        // Render the beta dashboard...

        return redirect('/dashboard/beta');
    }
}
```

### Business Hours Restriction

```php
Guard::condition('schedule-report', function () {
    $hour = (int) date('G');
    return $hour >= 9 && $hour < 17;
});

Guard::define('schedule-report', fn(User $user) => $user->canExport);
```

Reports can only be scheduled during business hours, regardless of the user's permissions outside that window.

### Environment Guard

```php
Guard::condition('debug-panel', fn() => app()->environment('local', 'staging'));
Guard::define('debug-panel', fn(User $user) => $user->isDeveloper);
```

In production, `debug-panel` is always denied regardless of the user's role.

### Maintenance Window

```php
Guard::condition('bulk-import', function () {
    $hour = (int) gmdate('G');
    return !($hour >= 2 && $hour < 4);
});

Guard::define('bulk-import', fn(User $user) => $user->canImport);
```

Bulk imports are blocked during a nightly maintenance window for all users.

### Odo View Integration

Conditions are invisible to Odo templates. The `#scope` directive evaluates `Guard::allows` internally, so the condition is enforced automatically:

```html
#scope('beta-dashboard')
<a href="/dashboard/beta">Beta Dashboard</a>
#endscope #scope('schedule-report')
<button>Export Report</button>
#endscope
```

### Conditions Work with Aliases

Conditions are applied to the resolved ability name after alias resolution. If an alias points to a conditional ability, the condition is enforced through the alias as well:

```php
Guard::condition('export-data', fn() => false);
Guard::define('export-data',    fn(User $user) => true);
Guard::alias('export', 'export-data');

// false — alias resolves to 'export-data', condition blocks it
Guard::allows('export');
```

### Super-Admin Bypass

Global `before` callbacks run before condition checks. A super-admin `before` callback therefore bypasses all conditions:

```php
Guard::before(function (User $user, string $ability) {
    if ($user->isSuperAdmin) {
        return true;
    }
    return null;
});

Guard::condition('locked-feature', fn() => false);
Guard::define('locked-feature',    fn() => true);

// true for super-admins, false for everyone else
Guard::allows('locked-feature');
```

### Inspecting Registered Conditions

```php
$conditions = Guard::conditions();
```

Output:

```php
[
    'ai-assistant' => Closure,
    'beta-dashboard' => Closure,
    ...
]
```

## Using Aliases, Wildcards, and Conditions Together

The three features compose naturally. Here is a complete example where a `post.create` ability is gated by a feature flag, covered by a wildcard rule, and accessible via an alias:

```php
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::wildcard('post.*', function (User $user) {
        return $user->role === 'editor';
    });

    Guard::condition('post.create', fn() => config('features.post_creation_enabled'));

    Guard::alias('write', 'post.create');
}
```

When `Guard::allows('write')` is called, Guard processes it in this sequence:

1. `'write'` is resolved through the alias chain to `'post.create'`
2. Global `before` callbacks run
3. The condition for `'post.create'` is evaluated — if the feature flag is off, deny immediately
4. No exact ability named `'post.create'` is registered — wildcard matching begins
5. `'post.*'` matches `'post.create'` — the wildcard callback checks the user's role
6. Global `after` callbacks run with the final result

In a controller:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/store', methods: ['POST'])]
    public function store(Request $request): RedirectResponse
    {
        if (! Guard::allows('write')) {
            abort(403);
        }

        // Create the post...

        return redirect('/posts');
    }
}
```

In an Odo template:

```html
#scope('write')
<a href="/posts/create">Write New Post</a>
#endscope
```

## Role Inference

Role inference lets you define a map of roles to abilities in one place. Guard reads a property on the authenticated user object, looks up that role in the map, and automatically grants every ability listed under it. No individual `Guard::define` calls are needed for abilities that are fully covered by a role.

Wildcard patterns work inside role ability lists too — `post.*` inside a role grants every `post.*` ability for users of that role, just as `Guard::wildcard()` would.

This is the right tool when your application has stable, well-known roles (`admin`, `editor`, `viewer`) and you want to declare their permissions in one readable map rather than scattering `Guard::define` calls across your codebase.

### Registering Roles

Define the role map in your `App\Providers\AppServiceProvider` using `Guard::roles`. The keys are role names and the values are lists of ability names:

```php
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::roles([
        'admin'  => ['manage-users', 'manage-settings', 'post.*'],
        'editor' => ['post.create', 'post.edit', 'post.view'],
        'viewer' => ['post.view'],
    ]);
}
```

Guard reads the `role` property on the authenticated user object by default. A user with `$user->role === 'editor'` is automatically granted `post.create`, `post.edit`, and `post.view` without any further configuration.

### Checking Role-Inferred Abilities

At the call site, nothing changes. Use `Guard::allows`, `auth()->can`, or `#scope` exactly as you would for any other ability:

```php
if (Guard::allows('post.create')) {
    // Granted automatically because user->role === 'editor'
}

if (auth()->can('manage-users')) {
    // Granted automatically because user->role === 'admin'
}
```

In a controller:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/store', methods: ['POST'])]
    public function store(Request $request): RedirectResponse
    {
        if (! Guard::allows('post.create')) {
            abort(403);
        }

        // Create the post...

        return redirect('/posts');
    }
}
```

In Odo templates:

```html
#scope('post.create')
    <a href="/posts/create">New Post</a>
#endscope

#scope('manage-users')
    <a href="/admin/users">Manage Users</a>
#endscope
```

### Wildcard Patterns Inside Roles

A `*` in a role's ability list follows the same dot-segment rules as `Guard::wildcard()`. Assigning `post.*` to a role grants every ability whose name begins with `post.`:

```php
Guard::roles([
    'admin'  => ['*'],  // grants every ability
    'editor' => ['post.*'],  // grants post.create, post.edit, post.delete, …
    'viewer' => ['post.view'],  // exact match only
]);
```

### Custom Role Property

If your user model stores the role under a different property name, pass it as the second argument:

```php
Guard::roles([
    'superuser' => ['manage-everything'],
    'operator'  => ['run-reports'],
], property: 'access_level');
```

Guard will now read `$user->access_level` instead of `$user->role`.

### Priority: Defined Abilities Win

Role inference is checked after directly defined abilities and wildcard patterns. If you register an explicit ability for an action, it always takes precedence over what the role map would grant:

```php
// Explicit deny — overrides whatever the role map says
Guard::define('post.delete', fn(User $user) => false);

Guard::roles(['admin' => ['post.*']]);

// Admin user is still denied post.delete because the explicit define runs first
Guard::allows('post.delete'); // false
```

This lets you grant broad access via roles while carving out specific exceptions without touching the role map.

### Inspecting the Role Map

```php
$map = Guard::roleMap();
// ['admin' => ['manage-users', 'post.*'], 'editor' => ['post.create', ...]]

$property = Guard::roleProperty();
// 'role'
```

## Lazy Abilities

A lazy ability defers evaluation of its callback until the first time the ability is actually checked in a request. If the ability is never checked, the callback is never called and its cost is zero.

This matters when an ability's authorization logic is expensive — for example, a callback that calls an external compliance service, reads a remote configuration, builds a complex object, or joins multiple database tables. In a typical request only a handful of abilities are actually evaluated. With `Guard::lazy`, all others cost nothing.

Once a lazy ability is checked for the first time, its callback is promoted into the standard abilities map and behaves identically to `Guard::define` for the rest of the request.

### Registering a Lazy Ability

Use `Guard::lazy` in your `AppServiceProvider`:

```php
use App\Models\User;
use App\Services\ComplianceService;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::lazy('compliance-check', function (User $user) {
        return ComplianceService::verify($user->id);
    });
}
```

The `ComplianceService::verify` call does not happen at boot, at service provider registration, or at any point before `Guard::allows('compliance-check')` is called. If this request never reaches that check, the service is never contacted.

### Checking a Lazy Ability

At the call site, lazy abilities are completely transparent. Use them exactly like any other ability:

```php
if (Guard::allows('compliance-check')) {
    // ComplianceService::verify was called here for the first time
}

if (auth()->can('compliance-check')) {
    // Works identically with auth()->can()
}
```

In a controller:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class ExportController extends Controller
{
    #[Route(uri: 'export/run', methods: ['POST'])]
    public function run(Request $request): RedirectResponse
    {
        if (! Guard::allows('compliance-check')) {
            abort(403);
        }

        // Run the export...

        return redirect('/exports');
    }
}
```

In Odo templates:

```html
#scope('compliance-check')
    <a href="/export/run">Run Export</a>
#endscope
```

### Promotion on First Check

When `Guard::allows('compliance-check')` is called:

1. Guard finds the ability in the lazy registry
2. The callback is moved into the standard abilities map
3. The lazy registry entry is removed
4. The callback is evaluated normally

Every subsequent call to `Guard::allows('compliance-check')` in the same request hits the standard abilities map directly — the callback runs once per request, never more.

### Lazy Abilities with Arguments

Lazy callbacks receive the same arguments as any `Guard::define` callback:

```php
Guard::lazy('expensive-ownership-check', function (User $user, Post $post) {
    return ExternalOwnershipService::verify($user->id, $post->id);
});
```

```php
Guard::allows('expensive-ownership-check', $post);
```

### Inspecting Lazy Abilities

```php
$pending = Guard::lazyAbilities();
// ['compliance-check' => Closure, ...]
// Only abilities not yet promoted appear here
```

## Ability Voting

Ability voting lets you attach multiple independent voter callbacks to a single ability. Each voter examines the user and any provided arguments from its own perspective and returns a vote. Guard tallies the votes according to a configurable strategy and returns the final boolean result.

This is the right tool for sensitive actions that require sign-off from multiple independent conditions — for example, publishing content requires the user to be an editor, the content to have been reviewed, and the content not to be banned. Writing this as a single callback with multiple `&&` conditions works, but voting makes each condition independent, individually testable, and easy to extend.

Each voter returns one of three values:
- `true` — affirmative vote (GRANT)
- `false` — negative vote (DENY)
- `null` — abstain (ignored in the tally entirely)

### Strategies

**`majority`** (default) — access is granted when more voters return `true` than `false`. Abstentions are not counted. Ties deny access.

**`unanimous`** — every non-abstaining voter must return `true`. A single `false` vote denies access regardless of all other votes. If all voters abstain, access is denied.

### Registering a Vote

Use `Guard::vote` in your `AppServiceProvider`:

```php
use App\Models\User;
use App\Models\Post;
use Doppar\Authorizer\Support\Facades\Guard;

public function boot(): void
{
    Guard::vote('publish-post', [
        fn(User $user, Post $post) => $user->isEditor,
        fn(User $user, Post $post) => $post->isReviewed,
        fn(User $user, Post $post) => !$post->isBanned,
    ]);
}
```

With no strategy specified, `majority` is used. Two of the three voters must return `true` for the ability to be granted.

### Registering a Unanimous Vote

```php
Guard::vote('deploy-production', [
    fn(User $user) => $user->hasRole('lead'),
    fn(User $user) => $user->hasMfa,
    fn(User $user) => !$user->isOnLeave,
], strategy: 'unanimous');
```

Every voter must agree. A user who is a lead with MFA enabled but currently on leave is denied.

### Checking a Voting Ability

At the call site, voting abilities are transparent. Use them like any other ability:

```php
if (Guard::allows('publish-post', $post)) {
    // All voters agreed (or majority did, depending on strategy)
}
```

In a controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    #[Route(uri: 'post/publish', methods: ['POST'])]
    public function publish(Request $request): RedirectResponse
    {
        $post = Post::find($request->post_id);

        if (! Guard::allows('publish-post', $post)) {
            abort(403);
        }

        // Publish the post...

        return redirect('/posts');
    }
}
```

In Odo templates:

```html
#forelse ($posts as $post)
    #scope('publish-post', $post)
        <tr>
            <td>[[ $post->title ]]</td>
            <td>
                <form method="POST" action="/post/publish">
                    <input type="hidden" name="post_id" value="[[ $post->id ]]">
                    <button type="submit">Publish</button>
                </form>
            </td>
        </tr>
    #endscope
#empty
#endforelse
```

### Abstentions

A voter returns `null` to abstain. Abstentions are excluded from the tally entirely — they do not count as a grant or a deny:

```php
Guard::vote('feature-access', [
    fn(User $user) => $user->isPremium, // votes true or false
    fn(User $user) => $user->isInBeta ? true : null, // abstains if not in beta
]);
```

The second voter only participates if the user is in the beta program. Otherwise it has no opinion and does not affect the outcome.

### Tally Reference

| Strategy | Grants | Denies | Abstains | Result |
|---|---|---|---|---|
| majority | 2 | 1 | 0 | granted |
| majority | 1 | 2 | 0 | denied |
| majority | 1 | 1 | 1 | denied (tie) |
| majority | 1 | 0 | 2 | granted |
| majority | 0 | 0 | 3 | denied |
| unanimous | 3 | 0 | 0 | granted |
| unanimous | 2 | 1 | 0 | denied |
| unanimous | 2 | 0 | 1 | granted |
| unanimous | 0 | 0 | 3 | denied (no grants) |

### Inspecting Voting Abilities

```php
$voting = Guard::votingAbilities();
```

Output:
```php
[
    'publish-post' => [
        'voters' => [...],
        'strategy' => 'majority',
    ],
    'deploy-production' => [
        'voters' => [...],
        'strategy' => 'unanimous',
    ],
]
```

Authorization in Doppar Guard keeps your application secure by providing a powerful, expressive way to define and evaluate user abilities. From simple permission checks to advanced features like roles, wildcards, conditions, and voting, Guard ensures access logic remains clean and maintainable.

Build scalable applications with confidence, knowing every action is protected by a flexible and reliable authorization system