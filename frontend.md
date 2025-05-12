## Blade Templates
### Introduction
Doppar includes a powerful and intuitive templating engine called Blade, which is designed to simplify the process of building dynamic HTML views. Blade allows developers to embed PHP logic directly into their templates using a clean and concise syntax, without sacrificing performance or readability.

Unlike traditional PHP, Blade provides a set of expressive directives such as `@if`, `@foreach`, and `@include` that are compiled into plain PHP code and cached until they are modified. This means you get the flexibility of PHP with the elegance of a templating language.

Blade templates in Doppar support:
- Control structures (conditions, loops)
- Template inheritance (layouts and sections)
- Components and includes (reusable partials)
- Data binding (safe, escaped output with {{ }})

Whether you're building simple pages or complex UIs, Blade makes it easy to keep your markup clean and maintainable.
### Conditionals and Loops

### @if
Checks if a condition is true.
```blade
@if ($condition)
    // Code to execute if the condition is true
@endif
```
### @elseif
Checks an additional condition if the previous `@if` or `@elseif` condition is false.
#### Syntax:
```blade
@if ($condition1)
    // Code for condition1
@elseif ($condition2)
    // Code for condition2
@endif
```
### @else
Executes code if all previous `@if` and `@elseif` conditions are false.

#### Syntax:
```blade
@if ($condition)
    // Code for condition
@else
    // Code if condition is false
@endif
```
### @unless
Executes code if the condition is false.
#### Syntax:
@unless ($condition)
    // Code to execute if the condition is false
@endunless

### @isset
Checks if a variable is set and not null.

#### Syntax:
```blade
@isset ($variable)
    // Code to execute if the variable is set
@endisset
```

### @unset
Unsets a variable.

#### Syntax:
```blade
@unset ($variable)
```

### @for
Executes a loop for a specified number of iterations.

#### Syntax:
```blade
@for ($i = 0; $i < 10; $i++)
    // Code to execute in each iteration
@endfor
```
### @foreach
Iterates over an array or collection.

#### Syntax:
```blade
@foreach ($items as $item)
    // Code to execute for each item
@endforeach
```

### @forelse
Iterates over an array or collection, with a fallback if the array is empty.

#### Syntax:
```blade
@forelse ($items as $item)
    // Code to execute for each item
@empty
    // Code to execute if the array is empty
@endforelse
```

### @while
Executes a loop while a condition is true.

#### Syntax:
```blade
@while ($condition)
    // Code to execute while the condition is true
@endwhile
```

### @switch
Executes a switch-case block.

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
Breaks out of a loop or switch-case block.

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
Skips the current iteration of a loop.

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
Executes raw PHP code.

#### Syntax:
```blade
@php
    // PHP code
@endphp
```

### @json
Encodes data as JSON.

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
Generates a CSRF token for forms.

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
Usage
```blade
@foreach ($users as $user)
    @if ($user->isAdmin())
        @exit
    @endif
@endforeach
```
### @empty
Usage
```blade
@empty ($users)
    //  User is empty
@endempty
```
## Section Directives
### @extends
```blade
@extends('layouts.app')
```
Extends a Blade layout.

### @include
```blade
@include('partials.header')
```
Includes another Blade view
### @yield
```blade
@yield('content')
```
Outputs the content of a section.

### @section
Example:
```blade
@section('content')
    <p>This is the content section</p>
@endsection
```
Defines a section to be yielded later

### @stop
```blade
@stop
```
Stops the current section output.

### @overwrite
```blade
@overwrite
```
Overwrites the current section content.

### Authentication Directives

### @auth
Checks if a user is authenticated.

#### Syntax:
```blade
@auth
    // Code to execute if the user is authenticated
@endauth
```
### @guest
Checks if a user is not authenticated (guest).

#### Syntax:
```blade
@guest
    // Code to execute if the user is not authenticated
@endguest
```
### Flash Message Directives
### @errors
Checks if there are any errors messages.

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
            @foreach (session()->get('errors') as $field => $messages)
                @foreach ($messages as $message)
                    <li>{{ $message }}</li>
                @endforeach
            @endforeach
        </ul>
    </div>
@enderrors
```

### @error
Checks if a specific error message exists in the flash messages.

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