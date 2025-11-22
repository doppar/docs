---
title: Authorization
description: Doppar Authorization
meta:
  - name: keywords
    content: Authorization
---

## Guard - Authorization
### Introduction
Guard is a robust and extensible authorization system designed for the Doppar framework. It provides a clean, expressive API for defining and evaluating user abilities across your application. With features like one-time temporary permissions, ability inheritance, grouped permissions, contextual policy checks, and global lifecycle hooks, Guard gives you complete control over access logic without cluttering your codebase.

Key features include:
- Define abilities using simple callbacks or complex logic
- Temporary (one-time) abilities for limited access
- Ability inheritance to model hierarchical permissions
- Grouped abilities for organized, modular access control
<!-- - 📚 Context-aware policy support for flexible access evaluation -->
- Dynamic ability registration from configuration or runtime
- Global before/after hooks for centralized control
- Bulk checks using any() and all() methods
Guard is built to empower your Doppar applications with secure, scalable, and maintainable authorization logic. Whether you're building small tools or large systems, Guard adapts to your needs with clarity and power.

## Installation
To get started with the Doppar Guard, simply install it via Composer:
```bash
composer require doppar/guard
```
Doppar does not support package auto-discovery, you need to manually register the service provider to enable the `Guard` facade.

In your `config/app.php` (or equivalent configuration file), add the following to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Authorizer\GuardServiceProvider::class,
],
```

Once installed and registered, the Guard facade will be available for use, giving you access to a rich set of tools for managing authorization.

## Basic Ability Checks
To define and check user abilities, you may use the `define`, `allows`, `denies`, and bulk-check methods provided by the Guard facade. The define method registers a new ability using a callback that determines access, while the allows and denies methods let you evaluate permissions at runtime.

You can define this guard in your `App\Providers\AppServiceProvider`. Let's look at how to register an ability and perform basic synchronous checks:
```php
use Doppar\Authorizer\Support\Facades\Guard;

Guard::define('edit-settings', function ($user) {
    return $user->isAdmin();
});
```

After you’ve defined your abilities using `Guard::define`, you can evaluate them at runtime using allows and denies methods:
```php

if (Guard::allows('edit-settings')) {
    // User is allowed to edit settings
}
```
- Use `Guard::allows(ability, arguments...)` to check if the current user has a specific ability.
- Use `Guard::denies(ability, arguments...)` to check if the user does not have a specific ability.

You can also check multiple abilities using the any method, which passes if at least one ability returns true:
```php
use Doppar\Authorizer\Support\Facades\Guard;

if (Guard::any(['edit-settings', 'admin-access'])) {
    // User has at least one of these abilities
}
```

In Doppar, you can use the `auth()->can()` method to check if the currently authenticated user has permission to perform a specific action or "ability." This is a quick and expressive way to handle authorization at any point in your code.

For example, suppose you want to allow users to edit application settings only if they have the edit-settings ability. You can write:
```php
if (auth()->can("edit-settings")) {
    dd("User is allowed to edit application settings.");
}
```

## Authorization with Odo Directives
Guard provides intuitive Odo directives to conditionally render frontend elements based on user abilities. This allows your views to remain clean and expressive, while staying in sync with your backend access logic.

The `@scope` directive checks if the currently authenticated user has the given ability:
```Odo
@scope('edit-settings')
    // You have edit-settings access
@elsescope('store-settings')
    // You have store-settings access
@else
    // You have default access
@endscope
```

You can pass additional arguments (like a model or context object) to check abilities with more precision:
```Odo
@scope('edit-settings', $user)
   //
@endscope

@scopenot('edit-settings', $user)
   //
@endscopenot
```

The `@scopenot` directive checks if the user does not have the given ability:
```Odo
@scopenot('edit-settings')
 //
@elsescopenot('store-settings')
 //
@else
 //
@endscopenot
```

## Defining Permissions
You can define authorization rules globally using the `Guard::define` method. Each permission is given a unique name (like 'update-post') and a closure that contains the authorization logic. The closure typically receives the currently authenticated user and any relevant model instance.

In this example, we'll define a gate to determine if a user can update a given `App\Models\Post` model. The gate will accomplish this by comparing the user's `id` against the `user_id` of the user that created the post:
```php
use App\Models\Post;
use App\Models\User;
use Doppar\Authorizer\Support\Facades\Guard;

/**
 * Bootstrap any application services.
 */
public function boot(): void
{
    Guard::define('update-post', function (User $user, Post $post) {
        return (int) $user->id === (int) $post->user_id;
    });
}
```

To check if the current user is authorized to perform an action, use `Guard::allows`. If the check passes, proceed. Otherwise, return a forbidden response or abort.
```php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use Phaseolies\Http\Response\RedirectResponse;
use Phaseolies\Http\Request;
use Doppar\Authorizer\Support\Facades\Guard;

class PostController extends Controller
{
    /**
     * Update the given post.
     */
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

## Odo View Integration with `#scope`
Doppar also introduces the `#scope` directive for Odo, enabling you to wrap parts of your view and conditionally render them only if the user has the required permission.
```html
#forelse ($posts as $item)
    #scope('update-post', $item)
        <tr>
            <td>[[ $item->id ]]</td>
            <td>[[ $item->user_id ]]</td>
            <th scope="row">[[ $item->title ]]</th>
        </tr>
    #endscope
#empty
#endforelse
```
This ensures that only users authorized to "update-post" a post will see the row, adding a secure and expressive layer to your templates.

## Writing Authorizers
Doppar provides a structured way to define permissions by leveraging dedicated Authorizer classes. This method keeps your authorization logic modular, testable, and easy to maintain—especially as your application scales.

Doppar provides `make:authorizer` command to create a authorizer class.
```bash
php pool make:authorizer PostAuthorizer --model=Post
```
This will generate a file like:
```php
<?php

namespace App\Authorizers;

use App\Models\User;
use App\Models\Post;

class PostAuthorizer
{
    /**
     * Determine whether the user can view any models.
     *
     * @param \App\Models\User $user
     * @return bool
     */
    public function viewAny(User $user): bool
    {
        //
    }

    /**
     * Determine whether the user can view the model.
     *
     * @param \App\Models\User $user
     * @param App\Models\Post $post
     * @return bool
     */
    public function view(User $user, Post $post): bool
    {
        //
    }

    /**
     * Determine whether the user can create models.
     *
     * @param \App\Models\User $user
     * @return bool
     */
    public function create(User $user): bool
    {
        //
    }

    /**
     * Determine whether the user can update the model.
     *
     * @param \App\Models\User $user
     * @param App\Models\Post $post
     * @return bool
     */
    public function update(User $user, Post $post): bool
    {
        //
    }

    /**
     * Determine whether the user can delete the model.
     *
     * @param \App\Models\User $user
     * @param App\Models\Post $post
     * @return bool
     */
    public function delete(User $user, Post $post): bool
    {
        //
    }
}
```
Each method represents a permission action. Now assume you want to do same thing you did before except authorizer class. So now update your `PostAuthorizer` class update method like this.
```php
/**
 * Determine whether the user can update the model.
 *
 * @param \App\Models\User $user
 * @param App\Models\Post $post
 * @return bool
 */
public function update(User $user, Post $post): bool
{
    return (int) $user->id === (int) $post->user_id;
}
```

Once your authorizer is defined, register it with the Guard:
```php
/**
 * Bootstrap any application services.
 *
 * @return void
 */
public function boot(): void
{
    Guard::authorize(Post::class, PostAuthorizer::class);
}
```
This tells the Guard service to use PostAuthorizer when handling permission checks related to Post. Now you can check permissions in your controllers or services using a clean and readable syntax:
```php
/**
 * Update the given post.
 */
public function update(Request $request): RedirectResponse
{
    $post = Post::find(1);

    if (! Guard::allows('update', $post)) {
        abort(403);
    }

    // Update the post...

    return redirect('/posts');
}
```

You can also conditionally render Odo components using the `@scope` directive, which works seamlessly with registered authorizers:
```Odo
@scope('update', $post)
 //
@endscope
```


## Temporary Abilities (One-Time Checks)
Guard allows you to define temporary abilities that are valid for a single check only. These are useful for sensitive, time-limited, or one-time actions such as confirming email, accepting terms, or completing onboarding.

Use the `Guard::temporary()` method to define a one-time permission:
```php
Guard::temporary('one-time-access', function($user) {
    return true; // Grant access for one check only
});
```

Once defined, the ability will pass only on the first check:
```php
if (Guard::allows('one-time-access')) {
    // ✅ This will execute (first check passes)
}

if (Guard::denies('one-time-access')) {
    // ❌ This will now execute (subsequent checks fail)
}
```

## Ability Inheritance (Hierarchy)
Guard supports ability inheritance, allowing you to define a parent ability that inherits permissions from multiple child abilities. This is useful for roles like admin, where access may depend on a combination of other specific abilities.

### Define Inheritance
Use `Guard::inherit()` to link a parent ability with its child abilities:
```php
Guard::define('manage-users', fn($user) => $user->isAdmin());
Guard::define('manage-settings', fn($user) => $user->isAdmin());

// Define parent and its inherited abilities
Guard::inherit('admin', ['manage-users', 'manage-settings']);
```
###  Check Inherited Ability
Now you can check the parent ability, and it will return true if any of its children return true:
```php
if (Guard::allows('admin')) {
    // This passes if the user can either manage users or settings
}
```

You can retrieve the list of child abilities using `getChildren():`
```php
$children = Guard::getChildren('admin');
// Returns: ['manage-users', 'manage-settings']
```

## Ability Grouping
Sometimes, you may want to logically organize related abilities under a named group. This can be useful when you're dealing with features like role management, admin interfaces, or permission reporting.

You can use the `Guard::group()` method to create a group of abilities and later use `Guard::inGroup()` to check if a particular ability belongs to that group.
```php
Guard::group('content-management', [
    'create-post',
    'edit-post',
    'delete-post'
]);
```

In the example above, we've grouped three content-related abilities under the `content-management` group. This allows you to refer to the group as a whole in your logic, rather than dealing with each ability individually.

You can then check if an ability is part of a specific group using the inGroup method:
```php
if (Guard::inGroup('content-management', 'edit-post')) {
    // This will return true, since 'edit-post' is part of 'content-management'
}
```

## Global before and after Callbacks
In some cases, you might want to intercept every authorization check globally—either to override logic (like for super admins), or for logging, metrics, or debugging. That’s where global before and after callbacks come into play.

The `before()` method registers a closure that will be executed before any ability is evaluated. This is ideal for:

```php
Guard::before(function($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true; // Super admins can do everything
    }

    return null; // Return null to continue with normal checks
});
```
If the callback returns a boolean (true or false), it short-circuits the check and skips all other logic, including abilities, policies, or hierarchies.

The `after()` method allows you to hook into the end of every authorization check, regardless of the result. This is useful for:
```php
Guard::after(function($user, $ability, $result) {
    logAccessAttempt($user, $ability, $result);
});
```
This always runs, even if a before callback short-circuits the process.
