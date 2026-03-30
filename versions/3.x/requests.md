---
title: Requests
description: Doppar Requests page
meta:
  - name: keywords
    content: Requests
---

## HTTP Requests

### Introduction

Doppar’s `Phaseolies\Http\Request` class provides a powerful, object-oriented interface for working with the current HTTP request. It goes far beyond simple input retrieval—offering an expressive API designed for modern, data-driven applications.

With the Request class, you can seamlessly:

- Access input fields, query parameters, cookies, headers, uploaded files, and more.
- Apply inline transformations to clean or normalize user input on the fly.
- Enforce conditions and constraints directly at the request layer.
- Extract structured data into arrays or DTOs with minimal boilerplate.
- Perform context-aware processing, ensuring data adapts to your application’s flow.
- Interact with routes, sessions, and validation results without additional glue code.
- Tap into advanced features like conditional transformations, nested DTO binding, blank input normalization, and fluent data manipulation.

In short, the Doppar Request class doesn’t just give you access to raw request data—it provides a comprehensive toolkit for transforming, validating, and binding input into the exact shape your application needs, all while keeping your controllers clean and expressive.

## Accessing the Request

To access request data, Doppar has some default built in method. Doppar provides a powerful Request class to handle incoming HTTP requests. This class offers a wide range of methods to access and manipulate request data, making it easy to build robust web applications.

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use Phaseolies\Utilities\Attributes\Route;

class UserController extends Controller
{
    #[Route(uri: 'user', method: 'POST')]
    public function store(Request $request)
    {
        $name = $request->name;
    }
}
```

As mentioned, you may also type-hint the `Phaseolies\Http\Request` class on a route closure. The service container will automatically inject the incoming request into the closure when it is executed:

```php
use Phaseolies\Http\Request;

Route::get('/', fn(Request $request) => $request->name);
```

## Global Request helper

The `request()` helper function in Doppar provides a convenient and globally accessible way to retrieve data from the current HTTP request. It’s an alias for accessing the `Phaseolies\Http\Request` instance without needing to inject or typehint. It supports multiple usage patterns. Let’s take a look:

See the basic usages of global http request helper

```php
$name = request('name', 'default');
$name = request()->name ?? 'default';
```

These are functionally equivalent to:

```php
$request->name ?? 'default';
```

What ever you are doing using `Phaseolies\Http\Request` instance, you can do the same job using `request()` helper any where without injecting `Phaseolies\Http\Request` as method params. The `request()` helper is available globally, so you can use it anywhere within your Doppar application

## Input Transformation

The `pipe()` method allows you to retrieve a request input value, transform it using a given callback, and return the result — all in a clean, functional style.

It's a convenient way to filter, transform, or format input data directly when fetching it.

Usage example of `pipe()` function

```php

$title = $request->pipe('title', fn($v) => strtoupper($v));
// Example input: "hello world" => Output: "HELLO WORLD"
```

You can also use built in php function like this way:
```php
$title = $request->pipe('title', 'strtoupper', 'Untitled');
// If 'title' is missing => 'Untitled' => Output: 'UNTITLED'
```

With default value:
```php
$slug = $request->pipe('slug', 'strtolower', 'default-slug');
// If 'slug' is missing => 'default-slug' => Output: 'default-slug'
```

If the input key doesn't exist, the $default value is used and passed to the callback. This ensures transformations don’t break your app due to missing input. Use `pipe()` when you want to transform request data inline and keep your controller or form logic clean and declarative.

### Multiple Input Transformation

The `pipeInputs()` method allows you to apply a set of transformations to multiple request input fields at once. Each input key is passed through its corresponding callback, and the transformed results are merged back into the request.

This is a powerful utility for sanitizing, formatting, or normalizing request data inline — especially when working with form inputs.

Basic usage example:

```php
$processed = $request
    ->pipeInputs([
        'name' => 'trim',
        'email' => 'strtolower',
    ])
    ->pipe('username', fn($v) => strtolower($v));

return $processed->all();
```

The `pipeInputs()` method provides a clean and declarative syntax, making it easy to transform request inputs in a readable and maintainable way. By allowing batch processing of multiple fields, it eliminates repetitive code and simplifies input normalization. This approach helps to keep your controller or form logic concise, focusing only on business logic rather than input handling. Additionally, since the method is fully chainable, it fits naturally into fluent request processing workflows, enabling more elegant and expressive code.

### Sanitize and Normalize Input

The `cleanse()` method provides a declarative way to sanitize and normalize request input data before it's used or stored. It allows you to define transformation rules for specific fields, supporting multiple rules per field using pipe (|) syntax, and works seamlessly with nested data structures using dot notation.

This is especially useful when you need to `strip` out `unwanted HTML`, `trim whitespace`, `enforce casing`, or convert types — all without cluttering your controller logic.

Supported Rules

- `trim:` Removes whitespace from the beginning and end of the string.
- `strip_tags:` Removes all HTML and PHP tags.
- `int:` Casts the value to an integer.
- `lowercase:` Converts the string to lowercase.
- `uppercase:` Converts the string to uppercase.

Each rule is applied in the order it’s defined.

See the example of aleaning and preparing input to store Post.

```php
$data = $request
    ->pipeInputs([
        'title' => fn($v) => ucfirst(trim($v)),
        'slug'  => fn($v) => Str::slug($v),
    ])
    ->cleanse([
        'excerpt' => 'strip_tags|trim',
        'body'    => 'strip_tags|trim',
    ]);

Post::create($data);
```

Using `cleanse()` keeps your input handling concise, readable, and centralized. It avoids scattered string operations or manual sanitization throughout your code. Combined with `pipeInputs()`, it forms a robust preprocessing layer that can be reused across forms, APIs, and admin panels.

### Transform Input Data

The `transform()` method offers a straightforward way to process specific input fields by applying a set of transformation callbacks. It’s ideal when you want to extract and reformat a subset of input values without altering the original request data.

Unlike `pipeInputs()`, which mutates the request object, `transform()` returns a new array containing only the transformed fields.

Here are the usage example of transform() function

```php
$data = $request->transform([
    'title' => fn($v) => ucfirst(trim($v)),
    'slug'  => fn($v) => Str::slug($v),
]);

return $data;
```

Example Input:

```json
{
  "title": "  a simple post ",
  "slug": "This is the slug"
}
```

Output:

```json
{
  "title": "A simple post",
  "slug": "this-is-the-slug"
}
```

Only the fields you specify in the `transform()` call are returned — making this method perfect for selectively extracting and formatting data for models, APIs, or view rendering.

### Enforce Input Conditions

The `ensure()` method allows you to apply inline, custom validation logic to individual request inputs. It ensures that a specific field meets a condition defined by your callback, and throws an exception if it doesn't.

This is especially useful for lightweight, rule-based checks outside of full-blown validation — ideal for enforcing structural or formatting constraints right where the data is being handled.

See the example of combining `ensure()` with `pipeInputs()` or `transform()` for a clean, declarative input processing flow:

```php
$data = $request
    ->pipeInputs([
        'title' => 'trim',
        'slug' => fn($v) => Str::slug($v),
    ])
    ->ensure('title', fn($v) => strlen($v) >= 10)
    ->ensure('slug', fn($v) => !empty($v))
    ->all();

Post::create($data);
```

If either condition fails, an exception is thrown immediately, halting the chain and indicating exactly which key failed.

### Context-Aware Input Processing

The `contextual()` method allows you to perform custom, data-aware transformations by giving you full access to the request’s current input state. You provide a callback that receives the entire input array and returns an array of modifications, which are then merged back into the request.

This method is ideal when the transformation logic depends on multiple fields or needs to be aware of the full request context.

Example: Generate Slug from Title and trim Title

```php
$request->contextual(function ($data) {
    return [
        'title' => trim($data['title']),
        'slug'  => Str::slug($data['title'] ?? '')
    ];
});

return $request->all();
```

The slug is dynamically generated based on the (trimmed) title, using logic that would be cumbersome with single-field transformations

`contextual()` plays well with other transformation methods. You can chain it with `pipeInputs()`, `cleanse()`, or `ensure()` for a full declarative preprocessing flow:

```php
$data = $request
    ->pipeInputs([
        'title' => fn($v) => ucfirst(trim($v)),
        'tags' => fn($v) => is_string($v) ? explode(',', $v) : $v
    ])
    ->contextual(fn($data) => [
        'slug' => Str::slug($data['title'])
    ])
    ->ensure('slug', fn($slug) => strlen($slug) > 0)
    ->only('title','slug');

Post::create($data);
```

### Extract Structured Data

The `extract()` method allows you to build custom payloads or derive structured data from the request in a clean, declarative way. It provides access to the entire request instance and returns the result of a user-defined callback — perfect for extracting only what you need, in the format you want.

Example: Custom Payload for Storage or Response

```php
$payload = $request->extract(function ($req) {
    return [
        'data' => $req->except(['token', '_method']),
        'timestamp' => now(),
    ];
});

return $payload;
```

Example Input:

```json
{
  "title": "Sample Post",
  "body": "Some content here.",
  "token": "abc123",
  "_method": "POST"
}
```

Output:

```json
{
  "data": {
    "title": "Sample Post",
    "body": "Some content here."
  },
  "timestamp": "2025-07-21T13:55:00"
}
```

You can return anything from `extract()` — arrays, objects, scalar values, or even JSON-ready payloads. This makes it a flexible tool for composing dynamic responses or request-based logic:

### Conditional Transformation

The `mapIf()` method provides a clean and expressive way to conditionally transform request data. It applies a given callback to the full input payload only if a specific condition is true — otherwise, it returns the original data unchanged.

This is useful when you want to mutate input based on the current environment, user roles, debug flags, or any runtime condition.

Transform Input in Non-Production Environments

```php
$data = $request->mapIf(!app()->isProduction(), function ($data) {
    return array_map('strtoupper', $data);
});

return $data;
```

`mapIf()` returns a plain array. If you want to continue chaining other request methods (pipeInputs(), cleanse(), etc.), apply transformations directly on the request before this, or rewrap the result as needed.

### Normalize Blank Inputs

The `nullifyBlanks()` method helps standardize empty input values by converting them to null. This ensures consistent nullability across your application and simplifies database inserts, validation, and model behavior — especially when dealing with optional fields.

Basic usage

```php
$cleanData = $request->nullifyBlanks()->all();
```

Input:

```json
{
  "name": "",
  "description": "   ",
  "tags": []
}
```

Output:

```json
{
  "name": null,
  "description": null,
  "tags": []
}
```

By default, empty arrays remain unchanged. Set `includeArrays` to true if you want to nullify them.

You can fluently combine `nullifyBlanks()` with other methods like `cleanse()`:

```php
$data = $request->nullifyBlanks()
    ->cleanse([
        'title' => 'trim',
        'count' => 'int',
    ]);
```

This first removes `empty/blank` values, then cleans specific fields — ideal for prepping data before validation or model operations.

Use this method early in your request pipeline to ensure all downstream logic works with clean, normalized data. Combine with `cleanse()`, `pipeInputs()`, and `ensure()` for full input hygiene.

## Seamless DTO Binding

The `bindTo()` method enables direct, structured mapping of request input data into strongly typed Data Transfer Objects (DTOs). It supports both flat and nested data binding, and can operate in strict or relaxed modes depending on your validation needs.

Basic Usage

```php
$userDto = $request->bindTo(new \App\DTO\UserDTO());
```

Given the following input:

```json
{
  "name": "Hello",
  "email": "h@h.com",
  "address": {
    "street": "123 Main St",
    "city": "New York"
  }
}
```

Your `$userDto` is now a fully populated object:

```json
\App\DTO\UserDTO {
    name: "Hello",
    email: "h@h.com",
    username: null,
    address: \App\DTO\AddressDTO {
        street: "123 Main St",
        city: "New York",
        zipCode: null
    }
}
```

#### Strict vs. Non-Strict Mode

Strict mode ensures that only known and declared properties are set:

```php
$request->bindTo(new UserDTO(), strict: true); // default
```

Relaxed mode lets you bind to undeclared properties — useful for dynamic DTOs:

```php
$request->bindTo(new \stdClass(), strict: false);
```

## Nested DTO Handling

DTO nesting is supported out of the box. If your input contains a sub-object, and your DTO defines a corresponding typed property, it will be resolved recursively:

```php
class UserDTO {
    public ?Address $address = null;
}
```

If no explicit type is declared, the system will attempt to guess the class name based on the property:

```php
// e.g. 'address' → Address, AddressDTO, etc.
```

You can override this behavior by explicitly setting types in your DTOs.

If your DTOs define a `toArray()` method, you can easily serialize them for output or further transformation:

```php
return $request->bindTo(new UserDTO())->toArray();
```

#### 📌 When to Use

- When working with structured JSON or form payloads.
- When isolating validation/data transfer logic into typed DTOs.
- When handling nested form structures in APIs, admin panels, or form builders.
- When needing clean, type-safe data layers beneath controllers or service classes.

### Attribute-based DTO binding in Controllers

You can bind the request body directly into controller parameters using the `#[BindPayload]` attribute. The router will instantiate the parameter type and hydrate it via `Request::bindTo()` before your method runs.

```php
use Phaseolies\Utilities\Attributes\BindPayload;
use App\Models\User;

#[Route('api/user', methods: ['POST'])]
public function store(
    #[BindPayload(strict: false)]
    User $user
) {
    return User::createFromModel($user);
}
```

## Tap into Input with `tapInput`

The `tapInput()` method offers a convenient way to inspect, log, or act on a specific input value without modifying it.

Basic Usage

```php
$request->tapInput('views', function ($views) {
    Redis::increment("views:{$views}");
});
```

In this example, you access the views input and increment a Redis counter using its value — all without interrupting your form/request flow.

## Conditional Execution with `ifFilled`

The `ifFilled()` method allows you to conditionally act on input values only when they are present and non-empty, making it ideal for partial updates or optional fields in forms and APIs.

```php
$request->ifFilled('title', function ($title) use ($post) {
    $post->title = $title;
});

$request->ifFilled('slug', function ($slug) use ($post) {
    $post->slug = Str::slug($slug);
});
```

In this example, only the inputs that are provided (and not empty) are applied to the $post model. This avoids overwriting existing data with null or empty values.

## Convert Input to Array

The `asArray()` method retrieves an input value by key and guarantees it as an array, whether it’s already an array or a comma-separated string.

Basic usage example

```php
$tags = $request->asArray('tags');
// Input: 'php, doppar, clean code'
// Output: ['php', 'doppar', 'clean code']
```

If the input is a comma-separated string, it’s split, trimmed, and filtered into an array. If the input is already an array, it's returned as-is. If it's another type (like null or scalar), it’s safely cast to an array.

## Retrieving the Current Route

In Doppar, the `route()` method on the request object allows you to access or compare the current route name. This is useful for conditional logic based on routing, such as determining active navigation states or applying middleware behavior.

```php
$request->route();
// Returns the name of the current route as a string.

$request->route('dashboard');
// Returns true if the current route name is 'dashboard', otherwise false.
```

## Accessing `route()` in Odo Templates

In Odo views, you can directly access the current route using Doppar’s global Request alias — no need to import a namespace.

```html
#if(Request::route() === 'dashboard') [[-- Current route is "dashboard" --]]
#else [[-- Not on the "dashboard" route --]] #endif
```

### Alternative: Using `request()` Helper

The `request()` global helper works the same way:

```html
#if(request()->route() === 'dashboard') [[-- Current route is "dashboard" --]]
#else [[-- Not on the "dashboard" route --]] #endif
```

## Interacting with Form Request data

Doppar makes working with form data intuitive and expressive via the `Phaseolies\Http\Request` class. Whether you're retrieving all inputs, checking for field presence, or filtering specific values, Doppar provides a rich API to simplify handling request data from HTML forms.

Retrieve the complete set of request input:

```php
$request->all();
```

### Accessing Specific Field Values

Doppar provides various ways to interact with specific fields name. You can use any of them

```php
$request->name;
// Accesses the value of the 'name' field directly.

$request->input('name', 'default');
// Equivalent to the above, using the input() method.

$request->get('name', 'default');
// Same result, alternative syntax.
```

### Retrieving Data with Exclusions

Doppar provides `except` method to interact with fields name. You can use like that to exclude some fields from the current requested data.

```php
$request->except('name');
// Returns all data except the 'name' field.

$request->except('name', 'age');
// Excludes multiple fields from the result.
```

### Retrieving Specific Fields Only

Doppar provides `only` method to interact with fields name. You can use like that to get only those fields from the current requested data.

```php
$request->only('name');
// Returns only the 'name' field.

$request->only(['name', 'age']);
// Retrieves just the 'name' and 'age' fields.
```

### Checking Field Existence

When handling form submissions or query parameters in Doppar, it's often important to check whether specific fields are present or if the request contains any data at all. The Request class provides intuitive methods for these checks:

Use the has() method to check if a particular field is present in the incoming request data.

```php
$request->has('name');
// Returns true if the 'name' field exists.
```

This is useful for conditional logic or validating optional fields.

The `isEmpty()` method allows you to determine whether the entire request payload is empty.

```php
$request->isEmpty();
// Returns true if no request data is available.
```

This is especially helpful for detecting invalid or empty submissions before processing.

You can also use the `hasAny` method to check multiple key existance. The `hasAny` method checks whether at least one of the given input keys exists in the request. This is especially useful when you want to conditionally act if any one of multiple inputs is present.

```php
if ($request->hasAny('title', 'username')) {
    // At least one of the keys exists in the request
    return $request->title;
}
```

## Matching Request Paths

Need to determine if the current request URL matches a specific pattern? The `is()` method allows you to compare the request path against a pattern with wildcard support. This is especially useful for applying conditional logic based on the current route.

```php
$request->is('/user/*');
// Matches routes like /user/profile, /user/settings, etc.
```

## Accessing Validation Results

After validating form input, you might want to separate valid and invalid data. Doppar provides two handy methods for this:

- `passed():` Returns only the fields that passed validation.
- `failed():` Returns the fields that failed validation, typically used for error feedback.

```php
$request->passed();
// Returns only the data that passed validation.

$request->failed();
// Returns the data that failed validation checks.
```

## Accessing Session Data

Doppar's Request class provides seamless access to session data, allowing you to retrieve, store, and manage temporary user-specific data easily. This is particularly useful for flash messages, authentication info, and CSRF protection.

### Retrieve CSRF Token

When building forms or making authenticated requests, it's important to include a CSRF (Cross-Site Request Forgery) token to protect your application from malicious requests. Doppar automatically generates and stores a CSRF token in the session for each user. You can retrieve this token whenever needed—for example, to include it in a custom form or API request header.

Use the following method to access the current session's CSRF token:

```php
$request->session()->token();
// Retrieves the CSRF token stored in the session.
```

Use this token to secure form submissions and protect against cross-site request forgery.

### Get Session Value by Key

Doppar provides a convenient way to interact with session data. To retrieve a specific value stored in the session, you can use the `get` method by passing the key associated with the data. This is useful for accessing things like flash messages, user preferences, or any data saved during a previous request.

Here’s how you can get a session value by its key:

```php
$request->session()->get('key');
// Returns the value stored in the session under 'key'.
```

### Set Session Data

To store data in the session, Doppar allows you to use the `put` method on the session instance. This is useful for saving user-specific data like authentication status, flash messages, preferences, or temporary variables that need to persist between requests.

Here’s how to set a value in the session:

```php
$request->session()->put('key', 'value');
// Stores 'value' in the session under the key 'key'.
```

You can use this to save custom data in the session for later retrieval.

### Clear All Session Data

If you need to remove all data from the session—such as during logout or when resetting a user's state—you can use the flush method. This method clears the entire session storage for the current request lifecycle.

```php
$request->session()->flush();
// Deletes all session data.
```

### Request Path, Host, and Method

The `Phaseolies\Http\Request` instance provides a variety of methods for examining the incoming HTTP request and usage some method from the `Symfony\Component\HttpFoundation\Request` class. We will discuss a few of the most important methods below.

### Retrieving the Request Path

The path method returns the request's path information. So, if the incoming request is targeted at http://example.com/foo/bar, the path method will return `foo/bar`:

```php
$uri = $request->getPath();
```

### Retrieving the Request URL

To retrieve the full URL for the incoming request you may use the url or fullUrl methods. The url method will return the URL without the query string, while the fullUrl method includes the query string:

```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

### Retrieving the Request Host

You may retrieve the "host" of the incoming request via the host, scheme methods:

```php
$request->host();
$request->scheme();
```

## Retrieving the Request Method

The method method will return the HTTP verb for the request. You may use the isMethod method to verify that the HTTP verb matches a given string:

```php
$method = $request->method(); // return 'get'
$method = $request->getMethod(); // return 'GET'

if ($request->isGet()) {
    //
 }
```

You will get the method like `isPost()`, `isDelete()`, `isPatch()`, `isPut()` etc to check incoming HTTP request type.

## Request Headers

You can retrieve a request header using the header method on the` Phaseolies\Http\Request` instance in Doppar. If the specified header does not exist in the request, it will return null by default. However, you may also provide a second argument to return a default value if the header is missing.

```php
$agent = $request->header('User-Agent');
$agent = $request->headers->get('User-Agent')
// Returns the value of the 'User-Agent' header
```

You can set headers for your current request like

```php
$request->headers->set('key','value'); // Set the value to the header
```

The `hasHeader` method may be used to determine if the request contains a given header:

```php
if ($request->hasHeader('User-Agent')) {
    // ...
}
```

For convenience, the `bearerToken` method may be used to retrieve a bearer token from the Authorization header.

```php
$token = $request->bearerToken();
```

## Request IP Address

The `ip` method may be used to retrieve the IP address of the client that made the request to your application

```php
$ipAddress = $request->ip();
```

To retrieve an array of IP addresses — including all client IPs forwarded by proxies — you can use the `ips()` method on the `Phaseolies\Http\Request` instance. This method returns all available IPs in order, with the original client IP listed at the end of the array.

```php
$ipAddresses = $request->ips();
```

This is especially useful when your application is behind a load balancer or proxy and you want to trace the full route of the request

## Content Negotiation

Doppar allows you to inspect the content types a client can accept through the Accept header of the request. Using the `getAcceptableContentTypes()` method from the `Phaseolies\Http\Request` class, you can retrieve an array of all content types the client is willing to receive.

```php
$acceptedTypes = $request->getAcceptableContentTypes();
```

The `accepts` method `accepts` an array of content types and returns true if any of the content types are accepted by the request. Otherwise, false will be returned:

```php
if ($request->accepts(['text/html', 'application/json'])) {
    // ...
}
```

You may use the `prefers` method to determine which content type out of a given array of content types is most preferred by the request. If none of the provided content types are accepted by the request, null will be returned:

```php
$preferred = $request->prefers(['text/html', 'application/json']);
```

Since many applications only serve HTML or JSON, you may use the `expectsJson` method to quickly determine if the incoming request expects a JSON response:

```php
if ($request->expectsJson()) {
    // ...
}
```

There are some methods you can use like

```php
$request->wantsJson();
$request->isAjax();
$request->isSecure();
```

To check all the available methods, you can check `Phaseolies\Http\Request` class.

## Merging Additional Input

Sometimes you may need to manually `merge` additional input into the request's existing input data. To accomplish this, you may use the `merge` method. If a given input key already exists on the request, it will be overwritten by the data provided to the `merge` method:

```php
$request->merge(['country' => 'bd']);
```

You can also use `mergeIfMissing` for the default. The `mergeIfMissing` method allows you to merge default values into the request input only if the keys are missing. It’s a simple way to enforce fallback data without overwriting existing input.

```php
$request->mergeIfMissing([
    'language' => 'en',
    'theme'    => 'light',
]);
```

If `language` or `theme` were not included in the request, this method injects them with default values.

### Retrieving Old Input

Doppar provides a global `old` helper. If you are displaying old input within a Odo template, it is more convenient to use the old helper to repopulate the form. If no old input exists for the given field, null will be returned:

```html
<input type="text" name="username" value="[[ old('username') ]]" />
```

## Cookies

In the Doppar framework, you can interact with HTTP cookies through the `Phaseolies\Http\Request` object. Cookies are a convenient way to store small pieces of data on the client side, such as user preferences or session identifiers.

To retrieve cookie values, Doppar provides a clean and simple API. Here's how you can access a specific cookie from an incoming HTTP request:

### Retrieving Cookies From Requests

To retrieve a cookie value from the request, use the cookie method on an `Phaseolies\Http\Request` instance:

```php
$value = $request->cookies->get('name');
```

You can check request cookie existance using `hasCookie` method like that

```php
$request->hasCookie('key');
```

## Servers

In the Doppar framework, you can retrieve server-related variables (like headers, host info, request method, etc.) using the `Phaseolies\Http\Request` object. There are two main ways to access this data:

### Retrieve All Server Variables

To get all server variables as an associative array:

```php
$serverData = $request->server();
```

### Retrieve a Specific Server Variable

To get a single server variable by its key (e.g., 'HTTP_HOST', 'REQUEST_METHOD'):

```php
$host = $request->server->get('HTTP_HOST');
```

This allows you to target individual values without loading the entire array.

## Query String

When handling GET requests in Doopar, you often need to access the query parameters (data appended to the URL after a ?). Doopar provides several methods via the Request object to easily work with these.

```php
$queryString = $request->query();
```

Retrieve all query parameters from the current request as an associative array.

Retrieve the value of a specific query parameter using its key. assume example url is `http://example.com/search?name=mahedi&school=academy
`

```php
$name = $request->query('name'); // returns 'mahedi'
```

### With Default Value (Optional):

```php
$city = $request->query('city', 'unknown');
```

returns 'unknown' if 'city' is not present.

Returns the entire query string as a raw string, similar to $\_SERVER['QUERY_STRING'].

```php
$queryString = $request->getQueryString();
```

This will give you the raw query string `'name=mahedi&school=academy'`.

## Get Authenticated User

When a user is authenticated (typically via session, token, or doppar flarion), Doppar provides simple methods to retrieve their information through the request object.

```php
$request->auth(); // Retrieves the authenticated user data.
$request->user(); // Retrieves the authenticated user data.
```

## Files

Handling file uploads is straightforward in the Doppar framework. Uploaded files can be accessed directly from the `Phaseolies\Http\Request` instance using the `file()` method. This gives you access to a file object that provides various methods to inspect and move the uploaded file.

### Accessing the File Object:

You may retrieve uploaded files from an `Phaseolies\Http\Request` instance using the `file` method. To retrieve a file that was uploaded with the request, use the file method and pass the name of the input field:

```php
$file = $request->file('file');
```

Here 'file' is the HTML form input name.

### File Existence Checks

In addition to retrieving uploaded files using the `file()` method, the `Phaseolies\Http\Request` class also provides convenience methods to check whether specific files or any files at all have been uploaded with the request.

You may use the `hasFile` method to determine if a file was uploaded for a given input name. This method will return true only if the file exists in the request and was uploaded successfully.

```php
if ($request->hasFile('avatar')) {
    $file = $request->file('avatar');
    // Process the uploaded avatar file
}
```

To check if the request contains any successfully uploaded files (regardless of their input names), you may use the hasFiles method.

```php
if ($request->hasFiles()) {
    // At least one file was uploaded
}
```

### Retrieving File Information

Once you have accessed the uploaded file, the Doppar framework provides a variety of helpful methods to inspect and interact with the file. You can check its validity, extract metadata such as the original name or MIME type, and even move the file to a desired storage location.

Here’s a comprehensive example:

```php
if($file = $request->file('file')) {
    $file->isValid(); // Return boolean true or false
    $file->getClientOriginalName(); // Retrieves the original file name.
    $file->getClientOriginalPath(); // Retrieves the temporary file path.
    $file->getClientOriginalType(); // Retrieves the MIME type.
    $file->getClientOriginalSize(); // Retrieves the file size in bytes.
    $file->getClientOriginalExtension(); // Retrieves the file extension (e.g., "jpg", "png").
    $file->generateUniqueName(); // Generate a unique name for the uploaded file
    $file->isMimeType(string|array $mimeType); // Checks if the uploaded file is of a specific MIME type
    $file->isImage(); // Check if the uploaded file is an image.
    $file->isVideo(); // Check if the uploaded file is a video.
    $file->isDocument(); // Check if the uploaded file is a document.
    $file->move(string $destination, ?string $fileName = null); // Moves the uploaded file to a new location.
    $file->getMimeTypeByFileInfo(); // Get the file's mime type by using the fileinfo extension.
    $file->getError();
}
```

### Example

```php
if ($file = $request->file('file')) {
    if ($file->isValid()) {
        $fileName = $file->getClientOriginalName();
        $filePath = $file->getClientOriginalPath();
        $fileType = $file->getClientOriginalType();
        $fileExtension = $file->getClientOriginalExtension();
        $fileSize = $file->getClientOriginalSize();
    }
}
```
