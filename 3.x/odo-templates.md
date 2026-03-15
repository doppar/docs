---
title: Odo templates
description: Doppar Odo templates page
meta:
  - name: keywords
    content: Odo templates
---

## Odo Templates

### Introduction

Doppar includes **Odo**, a modern, lightweight, and fully customizable templating engine designed exclusively for the Doppar Framework. Odo makes it easy to build dynamic HTML views using expressive directives and clean syntax — without scattering raw PHP throughout your markup.

Instead of writing PHP directly in your views, Odo gives you readable, expressive template directives like `#if`, `#foreach`, and `#include`, along with powerful echo tags like `[[ ]]`, `[[! !]]`, and `[[[ ]]]`.

Odo compiles your templates into efficient native PHP and caches them automatically, giving you:

- The simplicity of a templating language
- The full power and flexibility of PHP
- The performance of precompiled, cached views

Every part of Odo's syntax — directive prefix, echo tags, comment tags, raw output tags, and escaped output tags — is fully configurable via `config/odo.php`. This makes Odo not just a template engine, but a fully customizable expression layer built for the Doppar ecosystem.

## Template Files

Odo template files use the `.odo.php` extension and are stored in the `resources/views` directory. Subdirectories are referenced using dot notation when rendering or including views.
```
resources/
└── views/
    ├── layouts/
    │   └── app.odo.php
    ├── partials/
    │   └── header.odo.php
    ├── auth/
    │   ├── login.odo.php
    │   └── register.odo.php
    └── home.odo.php
```

A view named `auth.login` maps to `resources/views/auth/login.odo.php`.

## Configuring Odo Syntax

Odo's syntax is fully flexible. Every delimiter — directives, echo tags, raw output, escaped output, and comment markers — can be customized in `config/odo.php`. This is useful if you are integrating Odo into a project that already uses conflicting syntax, or if you simply prefer a different style.
```php
return [
    'directive_prefix'   => '#',     // Directive prefix: #if, #foreach, etc.
    'open_echo'          => '[[',    // Regular echo open tag
    'close_echo'         => ']]',    // Regular echo close tag
    'open_raw_echo'      => '[[!',   // Raw (unescaped) echo open tag
    'close_raw_echo'     => '!]]',   // Raw (unescaped) echo close tag
    'open_escaped_echo'  => '[[[',   // Escaped echo open tag
    'close_escaped_echo' => ']]]',   // Escaped echo close tag
    'open_comment'       => '[[--',  // Comment open tag
    'close_comment'      => '--]]',  // Comment close tag
];
```

Once changed, your templates must use the new delimiters. For example, if you change `directive_prefix` to `@`, your directives become `@if`, `@foreach`, `@auth`, and so on.

## Displaying Data

### Regular Echo

Use `[[ ]]` to output a variable or expression. Values are automatically HTML-escaped to prevent XSS attacks. This is the recommended way to output user-provided data.
```html
<h1>[[ $user->name ]]</h1>
<p>[[ $post->title ]]</p>
<span>[[ date('Y') ]]</span>
```

### Raw Echo

Use `[[! !]]` to output raw HTML without escaping. Use this only when you trust the content — for example, when rendering sanitized HTML stored in your database.
```html
<div class="post-body">
    [[! $post->body !]]
</div>
```

> **Warning:** Never use raw echo on user-provided input without sanitizing it first. Raw echo bypasses all HTML escaping and can expose your application to XSS attacks.

### Escaped Echo

Use `[[[ ]]]` to apply explicit HTML escaping. This behaves identically to `[[ ]]` but makes the escaping intent clear in contexts where you want to be explicit:
```html
<p>[[[ $user->bio ]]]</p>
```

### Default Values

Use the `or` operator to provide a fallback value when a variable is not set or is null:
```html
<h1>[[ $title or 'Untitled' ]]</h1>
<p>[[ $user->bio or 'No bio provided.' ]]</p>
<img src="[[ $user->avatar or '/images/default.png' ]]">
```

## Comments

Odo comments are stripped entirely from the compiled output and never sent to the browser. They are ideal for leaving notes in your templates without affecting performance or page source.
```html
[[-- This comment will not appear in the rendered HTML --]]

[[--
    Multi-line comments are fully supported.
    Use these to document complex template sections.
--]]
```

Unlike HTML comments (`<!-- -->`), Odo comments are invisible even in the page source.

## Conditionals

### #if

The `#if` directive conditionally renders content based on any PHP expression:
```html
#if ($user->isAdmin())
    <a href="/admin">Admin Panel</a>
#endif
```

### #elseif

Chain additional conditions using `#elseif`:
```html
#if ($user->isAdmin())
    <a href="/admin">Admin Panel</a>
#elseif ($user->isEditor())
    <a href="/editor">Editor Panel</a>
#elseif ($user->isModerator())
    <a href="/moderation">Moderation Queue</a>
#endif
```

### #else

Provide a fallback using `#else`:
```html
#if ($user->isAdmin())
    <a href="/admin">Admin Panel</a>
#else
    <a href="/dashboard">Dashboard</a>
#endif
```

### #unless

The `#unless` directive is the inverse of `#if`. It renders its block only when the condition evaluates to `false`. It improves readability when checking for the absence of something:
```html
#unless ($user->isVerified())
    <div class="alert alert-warning">
        Your email address is not verified.
        <a href="/verify">Verify Now</a>
    </div>
#endunless
```

### #isset

Safely renders a block only when a variable exists and is not null:
```html
#isset ($user->phone)
    <p>Phone: [[ $user->phone ]]</p>
#endisset
```

This is equivalent to `#if (isset($user->phone))` but more expressive.

### #unset

Removes a variable from the template context at that point in rendering:
```html
#unset ($sensitiveData)

[[-- $sensitiveData is no longer accessible below this line --]]
```

## Loops

### #for

Runs a block a fixed number of times using a traditional for loop:
```html
#for ($i = 1; $i <= 5; $i++)
    <p>Step [[ $i ]]</p>
#endfor
```

### #foreach

Iterates over every item in an array or collection:
```html
#foreach ($posts as $post)
    <article>
        <h2>[[ $post->title ]]</h2>
        <p>[[ $post->excerpt ]]</p>
    </article>
#endforeach
```

#### The $loop Variable

Odo automatically provides a `$loop` variable inside every `#foreach` block. It gives you rich metadata about the current state of the loop without writing extra PHP:

| Variable | Type | Description |
|---|---|---|
| `$loop->iteration` | `int` | Current iteration number, starting from 1 |
| `$loop->index` | `int` | Current index, starting from 0 |
| `$loop->remaining` | `int` | Number of items remaining after the current one |
| `$loop->count` | `int` | Total number of items in the array |
| `$loop->first` | `bool` | `true` only on the first iteration |
| `$loop->last` | `bool` | `true` only on the last iteration |
| `$loop->depth` | `int` | Nesting depth — `1` for the outermost loop |
| `$loop->parent` | `object\|null` | The parent loop's `$loop` object when nested |
```html
#foreach ($posts as $post)
    #if ($loop->first)
        <h2>Latest Posts</h2>
    #endif

    <div class="post">
        <span class="counter">
            [[ $loop->iteration ]] of [[ $loop->count ]]
        </span>
        <h3>[[ $post->title ]]</h3>
        <p>[[ $post->excerpt ]]</p>
    </div>

    #if ($loop->last)
        <p class="end-note">You have reached the end.</p>
    #endif
#endforeach
```

#### Nested Loops

When nesting `#foreach` loops, Odo tracks depth automatically and exposes the outer loop via `$loop->parent`:
```html
#foreach ($categories as $category)
    <h2>[[ $category->name ]]</h2>

    #foreach ($category->posts as $post)
        <p>
            Category [[ $loop->parent->iteration ]],
            Post [[ $loop->iteration ]]:
            [[ $post->title ]]
            (Depth: [[ $loop->depth ]])
        </p>
    #endforeach
#endforeach
```

### #forelse

Works exactly like `#foreach` but renders an `#empty` fallback block when the array is empty or null. This eliminates the need for a separate `#if` check:
```html
#forelse ($posts as $post)
    <div class="card">
        <h3>[[ $post->title ]]</h3>
        <p>[[ $post->excerpt ]]</p>
    </div>
#empty
    <div class="alert alert-info">
        No posts found. <a href="/posts/create">Create one now.</a>
    </div>
#endforelse
```

### #while

Repeatedly executes a block as long as the condition is true. Useful when the number of iterations is not known in advance:
```html
#while ($queue->isNotEmpty())
    <p>Processing: [[ $queue->pop() ]]</p>
#endwhile
```

> **Warning:** Always ensure your `#while` condition will eventually become false to avoid infinite loops.

### #break

Exits the current loop immediately when a condition is met:
```html
#foreach ($users as $user)
    #if ($user->isBanned())
        #break
    #endif
    <p>[[ $user->name ]]</p>
#endforeach
```

You can break out of multiple nested loops at once by passing a depth integer:
```html
[[-- Breaks out of 2 levels of nested loops --]]
#break(2)
```

You can also break conditionally:
```html
#break($loop->iteration === 5)
```

### #continue

Skips the rest of the current iteration and moves to the next one:
```html
#foreach ($users as $user)
    #if ($user->isBanned())
        #continue
    #endif
    <p>[[ $user->name ]]</p>
#endforeach
```

Like `#break`, you can skip multiple nesting levels:
```html
#continue(2)
```

## Switch Statements

Use `#switch` when comparing a single variable against multiple possible values. This is cleaner than chaining multiple `#elseif` blocks:
```html
#switch ($user->role)
    #case ('admin')
        <p>Welcome, Administrator.</p>
        #break

    #case ('editor')
        <p>Welcome, Editor.</p>
        #break

    #case ('moderator')
        <p>Welcome, Moderator.</p>
        #break

    #default
        <p>Welcome.</p>
#endswitch
```

## Layout Inheritance

Odo's layout system lets you define a master page structure once and reuse it across all your views. Child views extend a layout and inject their content into named placeholders called sections.

### Defining a Layout

Create a base layout in `resources/views/layouts/app.odo.php`:
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>#yield('title') — [[ config('app.name') ]]</title>
    <link rel="stylesheet" href="/css/app.css">
    #yield('styles')
</head>
<body>
    <nav>
        #include('partials.navigation')
    </nav>

    <main class="container">
        #yield('content')
    </main>

    <footer>
        #include('partials.footer')
    </footer>

    <script src="/js/app.js"></script>
    #yield('scripts')
</body>
</html>
```

### Extending a Layout

In a child view, use `#extends` to inherit the layout and `#section` to fill each placeholder:
```html
#extends('layouts.app')

#section('title')
    Dashboard
#endsection

#section('styles')
    <link rel="stylesheet" href="/css/dashboard.css">
#endsection

#section('content')
    <h1>Welcome, [[ $user->name ]]</h1>

    #if ($user->isAdmin())
        <a href="/admin">Go to Admin Panel</a>
    #endif
#endsection

#section('scripts')
    <script src="/js/dashboard.js"></script>
#endsection
```

### #yield

Defines a named placeholder in a layout where child content will be injected. If the child does not define that section, nothing is output:
```html
#yield('content')
```

You can also provide a default value that renders when the child does not define the section:
```html
#yield('sidebar')
```

### #section / #endsection

Defines a named block of content in a child template that maps to a `#yield` in the parent layout:
```html
#section('content')
    <p>This content is injected into the layout's content placeholder.</p>
#endsection
```

### #stop

An alias for `#endsection`. Both work identically — use whichever reads better in context:
```html
#section('content')
    <p>Page content here.</p>
#stop
```

### #show

Ends a section and immediately outputs its content at that point. Useful for defining and rendering a section inline in a layout itself:
```html
#section('sidebar')
    <p>Default sidebar content.</p>
#show
```

### #append

Appends additional content to an existing section without replacing it. Useful for adding extra scripts or styles from nested includes:
```html
#section('scripts')
    <script src="/js/charts.js"></script>
#append
```

### #overwrite

Completely replaces the content of a section defined in a parent layout, discarding whatever was there before:
```html
#section('content')
    <p>This entirely replaces the parent section content.</p>
#overwrite
```

## Including Partials

### #include

Inserts another template file at the current position in the template. This is ideal for reusable components like navigation bars, footers, alerts, and cards:
```html
#include('partials.header')
#include('partials.footer')
#include('partials.navigation')
```

### Passing Data to Included Views

Pass an array as the second argument to make specific variables available inside the included view. Data passed this way is scoped only to that view and does not automatically inherit the parent's variables:
```html
#include('partials.alert', ['type' => 'success', 'message' => 'Your profile was updated.'])

#include('partials.user-card', ['user' => $user])

#include('partials.pagination', ['paginator' => $posts->paginator()])
```

## Raw PHP

### Inline PHP Statement

Use `#php()` for a single inline PHP expression when you need to assign a variable or run a quick calculation:
```html
#php($subtotal = $price * $quantity)
#php($tax = $subtotal * 0.15)

<p>Subtotal: [[ $subtotal ]]</p>
<p>Tax: [[ $tax ]]</p>
<p>Total: [[ $subtotal + $tax ]]</p>
```

### PHP Blocks

Use `#php / #endphp` for multi-line PHP logic that is too complex for a single expression:
```html
#php
    $grouped = collect($orders)->groupBy('status');
    $pending = $grouped->get('pending', []);
    $completed = $grouped->get('completed', []);
#endphp

<p>Pending: [[ count($pending) ]]</p>
<p>Completed: [[ count($completed) ]]</p>
```

## JSON Output

Use `#json` to encode a PHP variable as a JSON string for use in JavaScript. It automatically applies safe HTML encoding for use inside `<script>` tags:
```html
<script>
    const users = #json($users);
    const settings = #json($settings);
    const config = #json($config, JSON_PRETTY_PRINT);
</script>
```

## Variable Assignment

Use `#set` to assign or reassign a variable directly inside the template without a full `#php` block:
```html
#set('greeting', 'Good morning')
#set('year', date('Y'))

<h1>[[ $greeting ]], [[ $user->name ]]</h1>
<footer>© [[ $year ]]</footer>
```

## HTTP Method Spoofing

HTML forms only support `GET` and `POST`. Use `#method` to spoof `PUT`, `PATCH`, or `DELETE` requests so your routes can handle them correctly:
```html
[[-- Update a resource --]]
<form method="POST" action="/posts/[[ $post->id ]]">
    #csrf
    #method('PUT')
    <input type="text" name="title" value="[[ $post->title ]]">
    <button type="submit">Update</button>
</form>

[[-- Delete a resource --]]
<form method="POST" action="/posts/[[ $post->id ]]">
    #csrf
    #method('DELETE')
    <button type="submit">Delete</button>
</form>
```

## CSRF Protection

Always include `#csrf` inside any form that submits data via `POST`, `PUT`, `PATCH`, or `DELETE`. It inserts a hidden input field containing a CSRF token that Doppar validates on every non-GET request:
```html
<form method="POST" action="/contact">
    #csrf
    <input type="text" name="name" placeholder="Your name">
    <input type="email" name="email" placeholder="Your email">
    <textarea name="message"></textarea>
    <button type="submit">Send Message</button>
</form>
```

## Authentication Directives

### #auth

Renders its content only when a user is authenticated and logged in:
```html
#auth
    <p>Welcome back, [[ Auth::user()->name ]]</p>
    <a href="/profile">My Profile</a>
    <a href="/logout">Logout</a>
#endauth
```

### #guest

Renders its content only when no user is authenticated — i.e., the visitor is a guest:
```html
#guest
    <a href="/login">Login</a>
    <a href="/register">Create Account</a>
#endguest
```

Both directives can be combined in the same template:
```html
#auth
    <a href="/dashboard">Dashboard</a>
#endauth

#guest
    <a href="/login">Login</a>
#endguest
```

## Authorization Directives

### #scope

Renders content only when the authenticated user has a specific ability or permission. This integrates directly with Doppar's authorization system:
```html
#scope('edit-posts')
    <a href="/posts/[[ $post->id ]]/edit">Edit Post</a>
#endscope

#scope('delete-posts')
    <button class="btn btn-danger">Delete Post</button>
#endscope
```

### #elsescope

Provides an alternative block for a different ability, similar to `#elseif` for conditionals:
```html
#scope('admin')
    <a href="/admin">Admin Panel</a>
#elsescope('editor')
    <a href="/editor">Editor Panel</a>
#endscope
```

### #scopenot

The inverse of `#scope` — renders content only when the user does NOT have the specified ability:
```html
#scopenot('admin')
    <p>You do not have administrative access.</p>
#endscopenot
```

### #elsescopenot

Provides an alternative block inside a `#scopenot` directive:
```html
#scopenot('admin')
    <p>Not an admin.</p>
#elsescopenot('editor')
    <p>Not an editor either.</p>
#endscopenot
```

## #blank
The `#blank` directive renders its content block when a variable is empty — meaning it is null, an empty string `""`, an `empty` array `[]`, or `0`. This is a cleaner and more expressive alternative to writing `#if (empty($var))` every time you need to check for the absence of data.

```html
#blank($posts)
    <div class="alert alert-info">
        No posts found. <a href="/posts/create">Create your first post.</a>
    </div>
#endblank
```

## #notblank
The `#notblank` directive is the inverse of `#blank`. It renders its block only when the variable has a value — meaning it is not null, not an empty string, and not an empty array.

```html
#notblank($user->bio)
    <p>[[ $user->bio ]]</p>
#endnotblank

#notblank($posts)
    <p>[[ count($posts) ]] posts available.</p>
#endnotblank
```

## #solo
The `#solo` directive renders its content block exactly once per request, regardless of how many times the surrounding template or partial is included or iterated. With `#solo`, the asset tag renders exactly once no matter how many iterations the loop runs:
```html
#foreach ($posts as $post)
    #solo
        <script src="/js/editor.js"></script>
        <link rel="stylesheet" href="/css/editor.css">
    #endsolo

    <div class="post">
        <h3>[[ $post->title ]]</h3>
        <p>[[ $post->excerpt ]]</p>
    </div>
#endforeach
```

`#solo` is also useful when a partial is included from multiple places in the same request. Even if the partial is included ten times across different views, anything inside `#solo` renders only on the first encounter:

## Inject and Slot Directives
Odo provides a named content stack system through the `#inject` and #slot directives. This system allows any view or partial — regardless of where it sits in the template hierarchy — to push content into a named location defined in the layout. This is the recommended way to manage page-specific scripts, stylesheets, and any other content that belongs in a specific part of the layout but originates from a child view or partial.

### #slot
The `#slot` directive defines a named output point in a layout. It outputs all content that has been pushed into that slot by any view or partial during the current render. Define your slots in your layout file where you want the collected content to appear:
```html
[[-- resources/views/layouts/app.odo.php --]]
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>#yield('title') — [[ config('app.name') ]]</title>
    <link rel="stylesheet" href="/css/app.css">

    [[-- All stylesheets injected from child views appear here --]]
    #slot('styles')
</head>
<body>
    #yield('content')

    <script src="/js/app.js"></script>

    [[-- All scripts injected from child views appear here --]]
    #slot('scripts')
</body>
</html>
```
You can define as many named slots as you need. Common slot names include styles, scripts, head, and modals, but you can use any name that suits your layout.

### #inject
The `#inject` directive pushes a block of content into a named slot. Any view, partial, or component can inject content into any slot defined in the layout. Multiple `#inject` calls targeting the same slot are all collected and rendered together in the order they were pushed.

Injecting page-specific styles and scripts from a child view:
```html
[[-- resources/views/dashboard.odo.php --]]
#extends('layouts.app')

#inject('styles')
    <link rel="stylesheet" href="/css/dashboard.css">
    <link rel="stylesheet" href="/css/charts.css">
#endinject

#section('content')
    <h1>Dashboard</h1>
    #include('partials.revenue-chart')
    #include('partials.user-table')
#endsection

#inject('scripts')
    <script src="/js/dashboard.js"></script>
#endinject
```
## Flash Message Directives

### #errors

Renders its block only when validation errors exist in the session. Use this to wrap your error display so it only appears after a failed form submission:
```html
#errors
    <div class="alert alert-danger">
        <strong>Please fix the following errors:</strong>
        <ul>
            #foreach (session()->pull('errors') as $field => $messages)
                #foreach ($messages as $message)
                    <li>[[ $message ]]</li>
                #endforeach
            #endforeach
        </ul>
    </div>
#enderrors
```

### #error

Checks whether a specific form field has a validation error. The `$message` variable is automatically available inside the block and contains the error text for that field:
```html
<div class="form-group">
    <label>Email Address</label>
    <input type="email" name="email" value="[[ old('email') ]]">
    #error('email')
        <p class="text-danger small">[[ $message ]]</p>
    #enderror
</div>

<div class="form-group">
    <label>Password</label>
    <input type="password" name="password">
    #error('password')
        <p class="text-danger small">[[ $message ]]</p>
    #enderror
</div>
```

### Other Flash Messages

Use standard `#if` checks with the session helper to display other flash messages like success or warning alerts:
```html
#if (session()->has('success'))
    <div class="alert alert-success">
        [[ session()->pull('success') ]]
    </div>
#endif

#if (session()->has('warning'))
    <div class="alert alert-warning">
        [[ session()->pull('warning') ]]
    </div>
#endif

#if (session()->has('info'))
    <div class="alert alert-info">
        [[ session()->pull('info') ]]
    </div>
#endif
```

## Halting Execution

### #exit

Stops template execution at the point it is placed. Nothing after `#exit` in the template will be rendered. This can be useful for early returns based on conditions:
```html
#if (! $user->isActive())
    <p>Your account has been deactivated.</p>
    #exit
#endif

[[-- This content only renders for active users --]]
<h1>Welcome, [[ $user->name ]]</h1>
```

## Escaping Directives

To output a directive literally without it being processed by Odo, prefix it with an extra `#`. The double prefix is stripped and the directive is output as plain text:
```html
##if       [[-- outputs: #if --]]
##foreach  [[-- outputs: #foreach --]]
##auth     [[-- outputs: #auth --]]
##csrf     [[-- outputs: #csrf --]]
```

This is useful when writing documentation or code examples inside Odo templates.

## Custom Directives

Odo allows you to register your own custom template directives using `Odo::stamp()`. This lets you encapsulate any PHP logic into a clean, reusable template directive that can be used across all your views. Register custom directives inside the `boot()` method of any service provider.

### Registering Directives
```php
<?php

namespace App\Providers;

use Phaseolies\Support\ServiceProvider;
use Phaseolies\Support\Odo\Odo;

class AppServiceProvider extends ServiceProvider
{
    public function boot(): void
    {
        Odo::stamp('datetime', function ($expression) {
            return "<?php echo date('Y-m-d H:i:s', strtotime({$expression})); ?>";
        });
    }
}
```

### Inline Directives

Inline directives take a single expression and return a transformed output value:
```php
Odo::stamp('uppercase', function ($expression) {
    return "<?php echo strtoupper({$expression}); ?>";
});

Odo::stamp('lowercase', function ($expression) {
    return "<?php echo strtolower({$expression}); ?>";
});

Odo::stamp('datetime', function ($expression) {
    return "<?php echo date('Y-m-d H:i:s', strtotime({$expression})); ?>";
});

Odo::stamp('money', function ($expression) {
    return "<?php echo number_format({$expression}, 2); ?>";
});

Odo::stamp('asset', function ($expression) {
    return "<?php echo asset({$expression}); ?>";
});
```

Use them in your templates just like any built-in directive:
```html
<h1>#uppercase($user->name)</h1>
<p>#lowercase($user->email)</p>
<span>#datetime($post->created_at)</span>
<td>#money($product->price)</td>
<img src="#asset('images/logo.png')" alt="Logo">
```

### Block Directives

Block directives wrap sections of HTML with an opening and closing tag. Always register them as a pair — an opening directive that returns a PHP opening block, and a closing directive that returns the appropriate closing statement:
```php
Odo::stamp('role', function ($expression) {
    return "<?php if(auth()->user()->hasRole({$expression})): ?>";
});

Odo::stamp('endrole', function () {
    return "<?php endif; ?>";
});
```
```html
#role('admin')
    <a href="/admin">Dashboard</a>
    <a href="/users">Manage Users</a>
#endrole

#role('editor')
    <a href="/posts">Manage Posts</a>
#endrole
```

### Directive Utility Methods
```php
Odo::hasDirective('role');
```
Check whether a directive has been registered

```php
Odo::getDirectives();
```
Retrieve all registered custom directives

```php
Odo::forgetDirective('role');
```

### Naming Rules

Directive names must contain only alphanumeric characters and underscores. Hyphens and special characters are not allowed. Invalid names throw an `InvalidArgumentException` immediately on registration:
```php
Odo::stamp('my-directive', ...);   // InvalidArgumentException — hyphens not allowed
Odo::stamp('my directive', ...);   // InvalidArgumentException — spaces not allowed
Odo::stamp('my_directive', ...);   // Registered successfully
Odo::stamp('myDirective', ...);    // Registered successfully
Odo::stamp('roleCheck', ...);      // Registered successfully
```

## Quick Reference

### Echo Tags

| Syntax | Escaped | Description |
|---|---|---|
| `[[ $var ]]` | Yes | Regular echo — HTML escaped output |
| `[[! $var !]]` | No | Raw echo — unescaped HTML output |
| `[[[ $var ]]]` | Yes | Explicit escaped echo |
| `[[-- comment --]]` | — | Template comment — never rendered |

### Built-in Directives

| Directive | Closing | Description |
|---|---|---|
| `#if ($expr)` | `#endif` | Conditional rendering |
| `#elseif ($expr)` | — | Additional condition branch |
| `#else` | — | Fallback condition branch |
| `#unless ($expr)` | `#endunless` | Renders when condition is false |
| `#isset ($var)` | `#endisset` | Renders when variable is set |
| `#unset ($var)` | — | Removes a variable |
| `#for (...)` | `#endfor` | Fixed count loop |
| `#foreach ($arr as $item)` | `#endforeach` | Array iteration with `$loop` |
| `#forelse ($arr as $item)` | `#endforelse` | Loop with `#empty` fallback |
| `#while ($expr)` | `#endwhile` | Condition-based loop |
| `#break` | — | Exit current loop or switch |
| `#break(n)` | — | Exit n levels of nesting |
| `#continue` | — | Skip current iteration |
| `#continue(n)` | — | Skip n levels of iteration |
| `#switch ($var)` | `#endswitch` | Switch statement |
| `#case ($val)` | — | Switch case branch |
| `#default` | — | Switch default branch |
| `#php` | `#endphp` | Raw PHP block |
| `#php($expr)` | — | Inline PHP expression |
| `#json($data)` | — | JSON encode and output |
| `#set('var', value)` | — | Assign a template variable |
| `#csrf` | — | Hidden CSRF token field |
| `#method('verb')` | — | HTTP method spoofing field |
| `#exit` | — | Halt template execution |
| `#extends('layout')` | — | Inherit a parent layout |
| `#section('name')` | `#endsection` | Define a named content section |
| `#stop` | — | Alias for `#endsection` |
| `#show` | — | End section and output immediately |
| `#append` | — | Append content to a section |
| `#overwrite` | — | Replace parent section entirely |
| `#yield('name')` | — | Layout placeholder for a section |
| `#include('view')` | — | Include a partial template |
| `#auth` | `#endauth` | Renders for authenticated users |
| `#guest` | `#endguest` | Renders for unauthenticated users |
| `#scope('ability')` | `#endscope` | Renders when user has ability |
| `#elsescope('ability')` | — | Else branch for scope check |
| `#scopenot('ability')` | `#endscopenot` | Renders when user lacks ability |
| `#elsescopenot('ability')` | — | Else branch for scopenot |
| `#errors` | `#enderrors` | Renders when any errors exist |
| `#error('field')` | `#enderror` | Renders when field has an error |

### Custom Directive API

| Method | Description |
|---|---|
| `Odo::stamp($name, $callback)` | Register a custom directive |
| `Odo::hasDirective($name)` | Check if a directive is registered |
| `Odo::getDirectives()` | Get all registered custom directives |
| `Odo::forgetDirective($name)` | Remove a registered directive |