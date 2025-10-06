---
title: Blade templates
description: Doppar blade-templates page
meta:
  - name: keywords
    content: blade-templates
---

## Blade Templates
### Introduction
Doppar includes a powerful and intuitive templating engine called Blade, which is designed to simplify the process of building dynamic HTML views. Blade allows developers to embed PHP logic directly into their templates using a clean and concise syntax, without sacrificing performance or readability.

Unlike traditional PHP, Blade provides a set of expressive directives such as `@if`, `@foreach`, and `@include` that are compiled into plain PHP code and cached until they are modified. This means you get the flexibility of PHP with the elegance of a templating language.

## JIT Template Compilation
Doppar brings advanced `Just-in-Time (JIT)` compilation to the Blade template engine — a performance-focused feature that transforms how views are compiled and rendered at runtime.

### What is JIT Optimization?
In Doppar, JIT optimization refers to the process of dynamically analyzing and transforming compiled Blade templates at the moment they're needed — just before rendering. Instead of relying solely on precompiled static views, JIT inspects templates during request execution and applies smart, runtime optimizations to reduce rendering cost and speed up response times.

### Why JIT Makes Doppar Exceptional
JIT is enabled by default `(BLADE_JIT_ENABLED=true)` and you can configure it from `.env` and comes with an adjustable optimization level:
| Level | Description                       | Features Included                                                                   |
| ----- | --------------------------------- | ----------------------------------------------------------------------------------- |
| 0     | Disabled                          | No runtime optimization, standard Blade behavior                                    |
| 1     | Basic Optimization                | Whitespace cleanup, faster control structures, echo consolidation                   |
| 2     | Aggressive Optimization (default) | Inlines small templates, simplifies nested loops, lazy-loads components dynamically |

### What JIT Does Under the Hood
When a view is compiled, Doppar’s JIT engine intelligently refines the resulting PHP code using techniques like:

<ul class="doppar-features">
  <li><strong>Whitespace Reduction</strong>: Compresses unnecessary spacing to reduce file size and improve parsing speed.</li>
  <li><strong>Optimized Control Structures</strong>: Transforms verbose `@if`, `@foreach`, and `@else` blocks into efficient native PHP syntax, leading to cleaner and faster execution.</li>
  <li><strong>Echo Optimization</strong>: Combines adjacent `{{ }}` outputs into single expressions, significantly reducing PHP overhead during rendering.</li>
  <li><strong>Blade Loop Simplification</strong>: Detects and optimizes common loop constructs (like `@foreach`) for faster iteration and cleaner compiled output, improving rendering efficiency.</li>
  <li><strong>Inline Small Templates</strong>: Automatically inlines `@include()` directives for small views (under 500 bytes). This eliminates disk I/O for these components, speeding up overall load times.</li>
  <li><strong>Lazy Component Loading</strong>: Converts Blade components into shared, in-memory objects that render only once per request. This intelligent caching mechanism drastically reduces duplication and improves performance for frequently used components..</li>
</ul>

### Why It Matters
Doppar’s JIT system is designed to improve the real-world performance of your Blade templates, especially:

- On high-traffic sites where rendering efficiency matters
- When using reusable components across pages
- For developers who want to keep using Blade’s elegance without sacrificing speed

Doppar doesn’t just compile Blade — it understands it, improves it, and runs it faster than ever.

Blade templates in Doppar support:
- Control structures (conditions, loops)
- Template inheritance (layouts and sections)
- Components and includes (reusable partials)
- Data binding (safe, escaped output with {{ }})

Whether you're building simple pages or complex UIs, Blade makes it easy to keep your markup clean and maintainable.
## Conditionals and Loops

### @if
The `@if` directive is used to conditionally render content based on a variable or expression. If the condition evaluates to true, the content inside the @if block will be rendered. Otherwise, it will be skipped.
```blade
@if ($condition)
    // Code to execute if the condition is true
@endif
```
### @elseif
The `@elseif` directive is used to check an additional condition if the previous `@if` or any prior `@elseif` condition was not met (i.e., evaluated as false).
#### Syntax:
```blade
@if ($condition1)
    // Code for condition1
@elseif ($condition2)
    // Code for condition2
@endif
```
### @else
The `@else` directive is used to define a fallback block of content. It will run only if none of the previous @if or @elseif conditions are true.
#### Syntax:
```blade
@if ($condition)
    // Code for condition
@else
    // Code if condition is false
@endif
```
### @unless
The `@unless` directive is the inverse of `@if`. It checks if a condition is false, and only then executes the block of code inside. This makes it especially useful for readability when you're testing for the absence of a condition.
#### Syntax:
```blade
@unless ($condition)
    // Code to execute if the condition is false
@endunless
```

### @isset
The `@isset` directive checks if a variable exists and is not null before attempting to use it. This prevents errors and allows you to safely access variables that may or may not be present in the current context.

#### Syntax:
```blade
@isset ($variable)
    // Code to execute if the variable is set
@endisset
```

### @unset
The `@unset` directive is used to remove a variable from memory so it is no longer available in the template context after that point. This can help prevent accidental reuse or override of sensitive or temporary data.

#### Syntax:
```blade
@unset ($variable)
```

### @for
The `@for` directive is used to create a loop that runs a specific number of times. It’s helpful when you want to repeat a block of content — such as generating rows, repeating messages, or testing layouts with placeholder items.

#### Syntax:
```blade
@for ($i = 0; $i < 10; $i++)
    // Code to execute in each iteration
@endfor
```
### @foreach
The `@foreach` directive is used to loop through each item in an array or collection. For each iteration, it assigns the current item to a temporary variable you define, which you can use inside the loop.

#### Syntax:
```blade
@foreach ($items as $item)
    // Code to execute for each item
@endforeach
```
Blade provides a `$loop` variable during `@foreach` iterations, which includes useful metadata about the current loop state. In your `@foreach` loop, you can access several useful loop variables through the `$loop` object in your `@foreach` blocks. Here's what information is available and how to use it:

#### Available Loop Variables

| Variable           | Description                             |
|--------------------|-----------------------------------------|
| `$loop->iteration` | Current iteration (1-based)             |
| `$loop->index`     | Current index (0-based)                 |
| `$loop->remaining` | Number of items remaining               |
| `$loop->count`     | Total number of items                   |
| `$loop->first`     | `true` if this is the first iteration   |
| `$loop->last`      | `true` if this is the last iteration    |
| `$loop->depth`     | Nesting level (`1` for the first loop)  |
| `$loop->parent`    | Parent loop object if nested            |

#### Example Usage:
```blade
@foreach (range(1, rand(1, 10)) as $item)
    <div>
        @if ($loop->first)
            <strong>START OF LOOP</strong><br>
        @endif
        Iteration: {{ $loop->iteration }}<br>
        Index: {{ $loop->index }}<br>
        Value: {{ $item }}<br>
        Remaining: {{ $loop->remaining }}<br>
        Count: {{ $loop->count }}<br>
        Depth: {{ $loop->depth }}<br>
        @if ($loop->last)
            <strong>END OF LOOP</strong>
        @endif
        <hr>
    </div>
@endforeach
```

#### Nested Loop Example:
When using nested loops, Blade automatically tracks the depth and provides access to the parent loop via `$loop->parent`.
This can be especially helpful when rendering complex structures like tables, trees, or multi-level menus.
```blade
@foreach (['A', 'B'] as $outer)
    @foreach (range(1, 3) as $inner)
        Outer: {{ $outer }}<br>
        Inner: {{ $inner }}<br>
        Depth: {{ $loop->depth }}<br>
        Parent: {{ $loop->parent ? $loop->parent->index : 'none' }}<br>
        <hr>
    @endforeach
@endforeach
```

### @forelse
The `@forelse` directive works like @foreach to iterate over each item in an array or collection. But it also provides an @empty block that runs if the array is empty or null. This lets you handle both cases in one clean construct.

#### Syntax:
```blade
@forelse ($items as $item)
    // Code to execute for each item
@empty
    // Code to execute if the array is empty
@endforelse
```

### @while
The `@while` directive repeatedly executes a block of code as long as the given condition evaluates to true. It’s ideal for situations where you don’t know in advance how many times the loop will run, but you want to keep repeating until a condition changes.

#### Syntax:
```blade
@while ($condition)
    // Code to execute while the condition is true
@endwhile
```

### @switch
The `@switch` directive evaluates a variable and runs the code block that matches one of the defined cases. If no case matches, the `@default` block runs. This is useful for replacing multiple `@if/@elseif` checks when comparing the same variable.

#### Syntax:
```blade
@switch ($variable)
    @case ($value1)
        // Code for value1
        @break

    @case ($value2)
        // Code for value2
        @break

    @default
        // Default code
@endswitch
```
### @break
The `@break` directive exits the nearest enclosing loop or switch-case block immediately. It’s useful when you want to stop processing as soon as a certain condition is met.

#### Syntax:
```blade
@break
```
Example
```blade
@foreach ($users as $user)
    @if ($user->isBanned)
        @break
    @endif
    <p>{{ $user->name }}</p>
@endforeach
```
### @continue
The `@continue` directive skips the rest of the current loop iteration and immediately starts the next iteration. It’s useful when you want to ignore certain items based on a condition but keep looping.

#### Syntax:
```blade
@continue
```
Example:
```blade
@foreach ($users as $user)
    @if ($user->isBanned)
        @continue
    @endif
    <p>{{ $user->name }}</p>
@endforeach
```
### @php
The `@php` directive lets you write raw PHP code blocks inside your template. This can be useful for complex logic, variable manipulation, or calculations that aren’t easily expressed with the template’s built-in directives.

#### Syntax:
```blade
@php
    // PHP code
@endphp
```

### @json
The `@json` directive encodes a PHP variable or data structure into JSON format so you can safely include it in your HTML or scripts. It automatically converts arrays or objects to a JSON string.

#### Syntax:
```blade
@json ($data)
```
Example
```blade
<script>
    var users = @json($users);
</script>
```

### @csrf
The `@csrf` directive inserts a hidden input field containing a CSRF (Cross-Site Request Forgery) token inside an HTML form. This token helps protect your application from CSRF attacks by verifying that the form submission came from your site.

#### Syntax:
```blade
@csrf
```
Example
```blade
<form method="POST">
    @csrf
    <button type="submit">Submit</button>
</form>
```
### @exit
The `@exit` directive halts the execution of the template completely at the point where it is used. It stops processing any further code or output.
```blade
@foreach ($users as $user)
    @if ($user->isAdmin())
        @exit
    @endif
@endforeach
```
### @empty
The `@empty` directive checks if a given variable is empty (meaning it’s either null, an empty array, an empty string, or evaluates to false). If it is empty, the enclosed block of code runs.

```blade
@empty ($users)
    //  User is empty
@endempty
```
## Section Directives
In Doppar, section directives play a crucial role in managing how content is organized and rendered across multiple pages. They allow you to define named sections within your templates that can be filled or overridden by child templates. This makes building complex layouts and reusable components much easier.

By using section directives along with layout inheritance (via @extends), you can create a master layout with common elements like headers, footers, and navigation, while customizing the content area on each individual page. This approach promotes cleaner, more maintainable code by separating structure from content.

### @extends
The `@extends` directive lets your template inherit from a base layout. This means you can define a master layout file (like a main page structure), and individual templates can extend it — injecting their content into predefined sections.

```blade
@extends('layouts.app')
```
Extends a Blade layout.

### @include
The `@include` directive allows you to insert another template (partial) inside your current template. This is useful for reusable components like headers, footers, or widgets.

```blade
@include('partials.header')
```
This pulls in the content of the `partials.header` template right where you place the directive.

### Passing Data to Included Views:
You can pass data to the included view by passing an array as the second argument:
```blade
@include('partials.version', ['version' => 'v2.0.4'])
```
The data passed to an included view is scoped only to that view. It does not automatically inherit all variables from the parent view. You must explicitly pass any data you want available in the included template.

### @yield
The `@yield` directive defines a placeholder in a layout where content from child templates will be injected. It’s used in a parent layout file to mark spots that child views can fill using @section.

```blade
@yield('content')
```
Outputs the content of a section.
The string 'content' is the name of the section. When a child template defines a section named content, its content replaces this placeholder during rendering. If the child doesn’t provide the section, nothing is output (or a default can be specified)
```html
<html>
<head>
  <title>My Site</title>
</head>
<body>
  <header>Site Header</header>

  <main>
    @yield('content')
  </main>

  <footer>Site Footer</footer>
</body>
</html>
```

### @section
The `@section` directive defines a named block of content that can be injected into a layout’s corresponding @yield placeholder. It’s how child templates specify the content for different sections of a parent layout.

Example:
```blade
@section('content')
    <p>This is the content section</p>
@endsection
```
This defines a section named content. Whatever is inside this block will replace `@yield('content')` in the parent layout when rendered.


### @stop
The `@stop` directive ends the current section, just like @endsection. It tells the template engine where the content for a section finishes.

```blade
@section('content')
    <p>This is the content section</p>
@stop
```
`@stop` marks the end of the content section. It works exactly like @endsection—you can use either one to close your sections.

### @overwrite
The `@overwrite` directive replaces the content of a section defined in a parent layout. Instead of appending or prepending, it completely overrides the section’s existing content.

```blade
@section('content')
    <p>This content will overwrite the parent layout's content.</p>
@overwrite
```
Use `@overwrite` at the end of a section to replace the parent section’s content entirely. This is useful when you want to provide a completely different version of a section without merging.

## Authentication Directives
These directives help you control content visibility based on user authentication status.

### @auth
The `@auth` directive checks if a user is authenticated (logged in). If the user is authenticated, the enclosed code will be executed.

#### Syntax:
```blade
@auth
    // Code to execute if the user is authenticated
@endauth
```
### @guest
The `@guest` directive checks if a user is NOT authenticated (i.e., a guest). It executes the enclosed code only when the user is not logged in.

#### Syntax:
```blade
@guest
    // Code to execute if the user is not authenticated
@endguest
```
## Flash Message Directives
These directives help you show feedback messages like errors, success alerts, or notifications to users.

### @errors
The `@errors` directive checks if there are any error messages available (usually from form validation). If errors exist, it runs the enclosed code block, letting you display them.

#### Syntax:
```blade
@errors
    // Code to execute if errors messages exist
@enderrors
```
Example
```blade
@errors
    <div class="alert alert-danger">
        <ul>
            @foreach (session()->pull('errors') as $messages)
                @foreach ($messages as $message)
                    <li>{{ $message }}</li>
                @endforeach
            @endforeach
        </ul>
    </div>
@enderrors
```
This block only renders if there are errors. Each error message is listed inside an unordered list.

### @error
The `@error` directive checks if a specific error message exists for a given input or key (like a form field name). If that key has an error, it will execute the enclosed code.

#### Syntax:
```blade
@error('key')
    // Code to execute if the error message exists
@enderror
```
Example
```blade
@error('email')
    <p class="error">{{ $message }}</p>
@enderror
```
If there's a validation error for the email field, it shows the error message. `$message` automatically contains the error text for that field.