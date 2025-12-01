---
title: Odo templates
description: Doppar html-templates page
meta:
  - name: keywords
    content: html-templates
---

## Odo Templates
### Introduction
Doppar includes ODO, a modern, lightweight, and fully customizable templating engine designed exclusively for the Doppar Framework.

ODO makes it easy to build dynamic HTML views using expressive directives and clean syntax, without the complexity of traditional PHP templating. Instead of mixing raw PHP throughout your markup, ODO provides clear, readable template expressions like `#if`, `#foreach`, and `#include`, along with powerful echo tags such as `[[ ]]` and `[[! !]]`.

The ODO engine compiles templates into efficient native PHP and caches them automatically. This gives you:

- the simplicity of a templating language,
- the flexibility of PHP,
- and the performance of precompiled views.

Every part of ODO's syntax — directive prefix, echo tags, comment tags, raw output tags, and more — is fully configurable. This makes ODO not just a templating engine, but a customizable expression layer tailored to the Doppar ecosystem.

ODO is built from the ground up for clarity, speed, and developer happiness — making template development in Doppar intuitive, elegant, and fast.

## Configure ODO Syntax
ODO is designed to be fully flexible. Every part of its syntax—directives, echo tags, raw output, escaped output, and comment markers—can be customized to match your preferred style or project requirements.

The syntax configuration lives in the `config/odo.php` file. This file allows you to define how ODO interprets template expressions and which symbols or delimiters it should use during compilation.

## Conditionals and Loops

### #if
The `#if` directive is used to conditionally render content based on a variable or expression. If the condition evaluates to true, the content inside the #if block will be rendered. Otherwise, it will be skipped.
```html
#if ($condition)
    // Code to execute if the condition is true
#endif
```
### #elseif
The `#elseif` directive is used to check an additional condition if the previous `#if` or any prior `#elseif` condition was not met (i.e., evaluated as false).
#### Syntax:
```html
#if ($condition1)
    // Code for condition1
#elseif ($condition2)
    // Code for condition2
#endif
```
### #else
The `#else` directive is used to define a fallback block of content. It will run only if none of the previous #if or #elseif conditions are true.
#### Syntax:
```html
#if ($condition)
    // Code for condition
#else
    // Code if condition is false
#endif
```
### #unless
The `#unless` directive is the inverse of `#if`. It checks if a condition is false, and only then executes the block of code inside. This makes it especially useful for readability when you're testing for the absence of a condition.
#### Syntax:
```html
#unless ($condition)
    // Code to execute if the condition is false
#endunless
```

### #isset
The `#isset` directive checks if a variable exists and is not null before attempting to use it. This prevents errors and allows you to safely access variables that may or may not be present in the current context.

#### Syntax:
```html
#isset ($variable)
    // Code to execute if the variable is set
#endisset
```

### #unset
The `#unset` directive is used to remove a variable from memory so it is no longer available in the template context after that point. This can help prevent accidental reuse or override of sensitive or temporary data.

#### Syntax:
```html
#unset ($variable)
```

### #for
The `#for` directive is used to create a loop that runs a specific number of times. It’s helpful when you want to repeat a block of content — such as generating rows, repeating messages, or testing layouts with placeholder items.

#### Syntax:
```html
#for ($i = 0; $i < 10; $i++)
    // Code to execute in each iteration
#endfor
```
### #foreach
The `#foreach` directive is used to loop through each item in an array or collection. For each iteration, it assigns the current item to a temporary variable you define, which you can use inside the loop.

#### Syntax:
```html
#foreach ($items as $item)
    // Code to execute for each item
#endforeach
```
Odo provides a `$loop` variable during `#foreach` iterations, which includes useful metadata about the current loop state. In your `#foreach` loop, you can access several useful loop variables through the `$loop` object in your `#foreach` blocks. Here's what information is available and how to use it:

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
```html
#foreach (range(1, rand(1, 10)) as $item)
    <div>
        #if ($loop->first)
            <strong>START OF LOOP</strong><br>
        #endif
        Iteration: [[ $loop->iteration ]]<br>
        Index: [[ $loop->index ]]<br>
        Value: [[ $item ]]<br>
        Remaining: [[ $loop->remaining ]]<br>
        Count: [[ $loop->count ]]<br>
        Depth: [[ $loop->depth ]]<br>
        #if ($loop->last)
            <strong>END OF LOOP</strong>
        #endif
        <hr>
    </div>
#endforeach
```

#### Nested Loop Example:
When using nested loops, Odo automatically tracks the depth and provides access to the parent loop via `$loop->parent`.
This can be especially helpful when rendering complex structures like tables, trees, or multi-level menus.
```html
#foreach (['A', 'B'] as $outer)
    #foreach (range(1, 3) as $inner)
        Outer: [[ $outer ]]<br>
        Inner: [[ $inner ]]<br>
        Depth: [[ $loop->depth ]]<br>
        Parent: [[ $loop->parent ? $loop->parent->index : 'none' ]]<br>
        <hr>
    #endforeach
#endforeach
```

### #forelse
The `#forelse` directive works like #foreach to iterate over each item in an array or collection. But it also provides an #empty block that runs if the array is empty or null. This lets you handle both cases in one clean construct.

#### Syntax:
```html
#forelse ($items as $item)
    // Code to execute for each item
#empty
    // Code to execute if the array is empty
#endforelse
```

### #while
The `#while` directive repeatedly executes a block of code as long as the given condition evaluates to true. It’s ideal for situations where you don’t know in advance how many times the loop will run, but you want to keep repeating until a condition changes.

#### Syntax:
```html
#while ($condition)
    // Code to execute while the condition is true
#endwhile
```

### #switch
The `#switch` directive evaluates a variable and runs the code block that matches one of the defined cases. If no case matches, the `#default` block runs. This is useful for replacing multiple `#if/#elseif` checks when comparing the same variable.

#### Syntax:
```html
#switch ($variable)
    #case ($value1)
        // Code for value1
        #break

    #case ($value2)
        // Code for value2
        #break

    #default
        // Default code
#endswitch
```
### #break
The `#break` directive exits the nearest enclosing loop or switch-case block immediately. It’s useful when you want to stop processing as soon as a certain condition is met.

#### Syntax:
```html
#break
```
Example
```html
#foreach ($users as $user)
    #if ($user->isBanned)
        #break
    #endif
    <p>[[ $user->name ]]</p>
#endforeach
```
### #continue
The `#continue` directive skips the rest of the current loop iteration and immediately starts the next iteration. It’s useful when you want to ignore certain items based on a condition but keep looping.

#### Syntax:
```html
#continue
```
Example:
```html
#foreach ($users as $user)
    #if ($user->isBanned)
        #continue
    #endif
    <p>[[ $user->name ]]</p>
#endforeach
```
### #php
The `#php` directive lets you write raw PHP code blocks inside your template. This can be useful for complex logic, variable manipulation, or calculations that aren’t easily expressed with the template’s built-in directives.

#### Syntax:
```html
#php
    // PHP code
#endphp
```

### #json
The `#json` directive encodes a PHP variable or data structure into JSON format so you can safely include it in your HTML or scripts. It automatically converts arrays or objects to a JSON string.

#### Syntax:
```html
#json ($data)
```
Example
```html
<script>
    var users = #json($users);
</script>
```

### #csrf
The `#csrf` directive inserts a hidden input field containing a CSRF (Cross-Site Request Forgery) token inside an HTML form. This token helps protect your application from CSRF attacks by verifying that the form submission came from your site.

#### Syntax:
```html
#csrf
```
Example
```html
<form method="POST">
    #csrf
    <button type="submit">Submit</button>
</form>
```
### #exit
The `#exit` directive halts the execution of the template completely at the point where it is used. It stops processing any further code or output.
```html
#foreach ($users as $user)
    #if ($user->isAdmin())
        #exit
    #endif
#endforeach
```
### #empty
The `#empty` directive checks if a given variable is empty (meaning it’s either null, an empty array, an empty string, or evaluates to false). If it is empty, the enclosed block of code runs.

```html
#empty ($users)
    //  User is empty
#endempty
```
## Section Directives
In Doppar, section directives play a crucial role in managing how content is organized and rendered across multiple pages. They allow you to define named sections within your templates that can be filled or overridden by child templates. This makes building complex layouts and reusable components much easier.

By using section directives along with layout inheritance (via #extends), you can create a master layout with common elements like headers, footers, and navigation, while customizing the content area on each individual page. This approach promotes cleaner, more maintainable code by separating structure from content.

### #extends
The `#extends` directive lets your template inherit from a base layout. This means you can define a master layout file (like a main page structure), and individual templates can extend it — injecting their content into predefined sections.

```html
#extends('layouts.app')
```
Extends a Odo layout.

### #include
The `#include` directive allows you to insert another template (partial) inside your current template. This is useful for reusable components like headers, footers, or widgets.

```html
#include('partials.header')
```
This pulls in the content of the `partials.header` template right where you place the directive.

### Passing Data to Included Views:
You can pass data to the included view by passing an array as the second argument:
```html
#include('partials.version', ['version' => 'v2.0.4'])
```
The data passed to an included view is scoped only to that view. It does not automatically inherit all variables from the parent view. You must explicitly pass any data you want available in the included template.

### #yield
The `#yield` directive defines a placeholder in a layout where content from child templates will be injected. It’s used in a parent layout file to mark spots that child views can fill using #section.

```html
#yield('content')
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
    #yield('content')
  </main>

  <footer>Site Footer</footer>
</body>
</html>
```

### #section
The `#section` directive defines a named block of content that can be injected into a layout’s corresponding #yield placeholder. It’s how child templates specify the content for different sections of a parent layout.

Example:
```html
#section('content')
    <p>This is the content section</p>
#endsection
```
This defines a section named content. Whatever is inside this block will replace `#yield('content')` in the parent layout when rendered.


### #stop
The `#stop` directive ends the current section, just like #endsection. It tells the template engine where the content for a section finishes.

```html
#section('content')
    <p>This is the content section</p>
#stop
```
`#stop` marks the end of the content section. It works exactly like #endsection—you can use either one to close your sections.

### #overwrite
The `#overwrite` directive replaces the content of a section defined in a parent layout. Instead of appending or prepending, it completely overrides the section’s existing content.

```html
#section('content')
    <p>This content will overwrite the parent layout's content.</p>
#overwrite
```
Use `#overwrite` at the end of a section to replace the parent section’s content entirely. This is useful when you want to provide a completely different version of a section without merging.

## Authentication Directives
These directives help you control content visibility based on user authentication status.

### #auth
The `#auth` directive checks if a user is authenticated (logged in). If the user is authenticated, the enclosed code will be executed.

#### Syntax:
```html
#auth
    // Code to execute if the user is authenticated
#endauth
```
### #guest
The `#guest` directive checks if a user is NOT authenticated (i.e., a guest). It executes the enclosed code only when the user is not logged in.

#### Syntax:
```html
#guest
    // Code to execute if the user is not authenticated
#endguest
```
## Flash Message Directives
These directives help you show feedback messages like errors, success alerts, or notifications to users.

#### Displaying Flash Messages

```html
#if (session()->has('success'))
    <div class="alert alert-success">
        [[ session()->pull('success') ]]
    </div>
#endif
```

### #errors
The `#errors` directive checks if there are any error messages available (usually from form validation). If errors exist, it runs the enclosed code block, letting you display them.

#### Syntax:
```html
#errors
    // Code to execute if errors messages exist
#enderrors
```
Example
```html
#errors
    <div class="alert alert-danger">
        <ul>
            #foreach (session()->pull('errors') as $messages)
                #foreach ($messages as $message)
                    <li>[[ $message ]]</li>
                #endforeach
            #endforeach
        </ul>
    </div>
#enderrors
```
This block only renders if there are errors. Each error message is listed inside an unordered list.

### #error
The `#error` directive checks if a specific error message exists for a given input or key (like a form field name). If that key has an error, it will execute the enclosed code.

#### Syntax:
```html
#error('key')
    // Code to execute if the error message exists
#enderror
```
Example
```html
#error('email')
    <p class="error">[[ $message ]]</p>
#enderror
```
If there's a validation error for the email field, it shows the error message. `$message` automatically contains the error text for that field.