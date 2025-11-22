---
title: Responses
description: Doppar Responses page
meta:
  - name: keywords
    content: Responses
---

## HTTP Responses
### Introduction
In any modern web application, generating meaningful responses to client requests is a core function. The Doppar framework makes this seamless by offering a flexible and expressive response system that supports everything from simple strings to complex JSON structures and custom response types.

Whenever a user interacts with your application—whether submitting a form, fetching data via an API, or simply loading a page—Doppar ensures that the correct response is constructed and returned to the browser. It abstracts the complexities of the underlying HTTP protocol, letting you focus on building robust application logic.

This section will guide you through various ways to generate HTTP responses in Doppar, including returning strings, arrays, JSON, views, and custom response objects. Whether you're building a basic blog or a full-featured API, Doppar provides the tools to respond cleanly and consistently.

## Creating Responses

#### Returning null
In Doppar, every route and controller is expected to return a response that will be sent back to the user's browser. Doppar provides a simple and intuitive way to handle responses directly from your routes and controllers.

If a route or controller returns null, Doppar will automatically convert the null value into an empty string. This ensures a consistent response format and prevents any unexpected behavior:
```php
Route::get('/', fn() => null);
```

#### Strings and Arrays
Every route and controller in Doppar should return a response that gets sent back to the user's browser. Doppar supports multiple ways to generate these responses. The simplest approach is to return a plain string from a route or controller. Doppar will automatically handle wrapping this string in a proper HTTP response:
```php
Route::get('/', fn() => "Hello World");
```

Similarly, you can return arrays or objects, and Doppar will automatically serialize them into JSON and generate the correct HTTP headers:
```php
Route::route('/', function() {
    return [
        'name' => 'Alex',
        'role' => 'Admin'
    ];
});
```
You can also return collection like
```php
Route::get('/', fn () => collect([1, 2, 3, 4]));
```

> Did you know you can also return Entity collections from your routes or controllers? They will automatically be converted to JSON.

## Response Objects
Typically, you won't just be returning simple strings or arrays from your route actions. Instead, you will be returning full `Phaseolies\Http\Response` instances.
```php
Route::get('/', function () {
    return response('Hello World', 200)
        ->setHeader('Content-Type', 'text/plain');
});
```

## Entity Models Collections
For Entity, Doppar usage its own Entity Model and Collection. So you can return Entity collection data as `Phaseolies\Http\Response`, Doppar will automatically convert the models and collections to JSON responses while respecting the model's hidden attributes:
```php
use App\Models\User;

Route::get('/', fn() => User::all());
```

## Attaching Headers to Responses
Remember, most response methods in Doppar are chainable, enabling you to build response objects in a fluent, readable style. For instance, you can use the setHeader method to attach multiple headers to the response before it's returned to the client:
```php
return response($content)
    ->setHeader('Content-Type', $type)
    ->setHeader('X-Header-One', 'Header Value')
    ->setHeader('X-Header-Two', 'Header Value');
```

Or, you may use the `withHeaders` method to specify an array of headers to be added to the response:
```php
return response($content)
    ->withHeaders([
        'Content-Type' => $type,
        'X-Header-One' => 'Header Value',
        'X-Header-Two' => 'Header Value',
    ]);
```

You also use the Response object as the controller method params and get the same response like

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Response;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function index(Response $response)
    {
        return $response->text("Hello World", 200)
            ->withHeaders([
                'Content-Type' => 'text/plain',
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
    }
}
```

## Response Collection
To return a collection of data as a JSON response, you can use the `json()` helper. This is useful when sending structured API responses, especially in RESTful applications.

```php
return response()->json([
        'data' => User::all()
    ], 200);
```
In this example, all users are retrieved and returned in a JSON object under the data key, with an HTTP status code of 200 (OK).

## Response Collection As Array
If you want to convert a collection into a plain PHP array before returning it in a response, you can use the `toArray()` method. This is useful when you need full control over the array structure or plan to further manipulate the data before responding.

```php
return response()->json([
        'data' => User::all()->toArray()
    ], 200);
```
In this example, `User::all()` returns a collection, and `toArray()` converts it into a raw array format. The resulting array is then included in the JSON response under the data key.

### Attaching Headers to Responses
You can also use Facades to get the response. You can use `Phaseolies\Support\Facades\Response` facades like
```php
namespace App\Http\Controllers;

use Phaseolies\Support\Facades\Response;

class UserController extends Controller
{
    public function index()
    {
        return Response::text("Hello World", 200)
            ->withHeaders([
                'Content-Type' => 'text/plain',
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ]);
    }
}
```

### Check Response Status
You can check the status of a response using helper methods provided by the response object. This is particularly useful when working with responses programmatically in tests or middleware.
```php
$response = response()->json(['data' => 'Doppar'], 200);

return $response->isOk();
```
In this example, the `isOk()` method checks whether the response has a 200 OK status. Doppar supports various response status checkers such as:

```php
public function isSuccessful(): bool
public function isRedirection(): bool
public function isClientError(): bool
public function isServerError(): bool
public function isOk(): bool
public function isForbidden(): bool
public function isNotFound(): bool
public function isRedirect(?string $location = null): bool
public function isFresh(): bool
public function isValidateable(): bool
public function setPrivate(): Response
public function setPublic(): Response
public function setImmutable(bool $immutable = true): Response
public function isImmutable(): bool
public function setCache(array $options): static
public function isInvalid(): bool
```
## Redirects Response
Redirect responses are instances of the `Phaseolies\Http\RedirectResponse` class, and contain the proper headers needed to redirect the user to another URL. There are several ways to generate a Redirect instance. The simplest method is to use the global redirect helper or even you can use `Phaseolies\Support\Facades\Redirect` facades
```php
use Phaseolies\Support\Facades\Redirect;

Route::get('/dashboard', function () {

    // Both will give you the same output
    return redirect()->to('/home/dashboard');
    return Redirect::to('/home/dashboard');

    // Even you can use
    redirect('/home/dashboard');
});
```

Sometimes you may wish to redirect the user to their previous location, such as when a submitted form is invalid. You may do so by using the global back helper function.

```php
Route::get('/dashboard', function () {
    return redirect()->back();

    // Even you can use
    return back();
});
```

## Redirecting to Named Routes
When you call the redirect helper with no parameters, an instance of `Phaseolies\Routing\RedirectResponse` is returned, allowing you to call any method on the Redirect instance. For example, to generate a Redirect to a named route, you may use the route method:
```php
return redirect()->route('login');
```

If your route has parameters, you may pass them as the second argument to the route method:
```php
// For a route with the following URI: /profile/{id}/{username}
return redirect()->route('profile', ['id' => 1, 'username' => 'doppar']);
```

### Redirecting to External Domains
Sometimes you may need to redirect to a domain outside of your application. You may do so by calling the `away` method, which creates a `RedirectResponse` without any additional URL encoding, validation, or verification:
```php
return redirect()->away('https://www.google.com');
```

### Redirecting With Flashed Session Data
Redirecting to a new URL and flashing data to the session are usually done at the same time. Typically, this is done after successfully performing an action when you flash a success message to the session. For convenience, you may create a `RedirectResponse` instance and flash data to the session in a single, fluent method chain:
```php
return redirect('/dashboard')->with('success', 'You are in dashboard');
```

After the user is redirected, you may display the flashed message from the session. For example, using Odo syntax:
```html
#if (session()->has('success'))
    <div class="alert alert-success">
       [[ session()->pull('success') ]]
    </div>
#endif
```

### View Responses
If you need control over the response's status and headers but also need to return a view as the response's content, you should use the view method:
```php
return response()->view('welcome')
    ->setHeader('Content-Type', 'text/html');
```

You can set headers also using `withHeaders` methods like this way
```php
return view('welcome')->withHeaders(['Content-Type' => 'text/html']);
```

## JSON Responses
The json method will automatically set the Content-Type header to `application/json`, as well as convert the given array to JSON using the json_encode PHP function:
```php
 return response()->json([
        'name' => 'Nure',
        'state' => 'BD'
    ], 200);
```

## File Downloads Response
The download method generates a response that triggers a file download in the user’s browser. It takes the file path as its primary argument. Optionally, you can specify a custom download filename as the second argument—this overrides the default name seen by the user. Additionally, an array of custom HTTP headers can be passed as a third argument for further control over the download behavior.
```php
return response()->download($pathToFile);

return response()->download($pathToFile, $name, $headers);
```

> Doppar usages Symfony HttpFoundation `BinaryFileResponse` class, which manages file downloads, requires the file being downloaded to have an `ASCII` filename.

## Streamed Downloads
At times, you may want to convert the string output of an operation into a downloadable response without storing it on disk. The streamDownload method allows you to achieve this by accepting a callback, filename, and an optional array of headers as parameters:
```php

use App\Services\Doppar;

return response()->streamDownload(function () {
    echo Doppar::api('repo')
        ->contents()
        ->readme('doppar', 'doppar')['contents'];
}, 'doppar-readme.md');
```

## File Responses
The file method may be used to display a file, such as an image or PDF, directly in the user's browser instead of initiating a download. This method accepts the absolute path to the file as its first argument and an array of headers as its second argument:
```php
return response()->file($pathToFile);

return response()->file($pathToFile, $headers);
```

## Streamed Responses
Streaming data to the client as it is generated can greatly reduce memory usage and enhance performance, particularly for extremely large responses.
```php
public function streamedContent(): \Generator
{
    yield 'Hello, ';
    yield 'World!';
}

return response()->stream(function (): void {
    foreach ($this->streamedContent() as $chunk) {
        echo $chunk;
        ob_flush();
        flush();
        sleep(2); // Simulate delay between chunks...
    }
}, 200, ['X-Accel-Buffering' => 'no']);
```

## Streamed JSON Responses
To stream JSON data incrementally, you can use the `streamJson` method. This is particularly beneficial for `large datasets` that need to be sent progressively to the browser in a format that JavaScript can easily parse
```php
return response()->streamJson([
    'users' => User::all()
]);
```

### Views
In Doppar, it's not practical to return entire HTML document strings directly from your routes and controllers. Fortunately, views offer a clean and structured way to manage your application's UI by keeping HTML in separate files.

Views help separate your application's logic from its presentation layer, improving maintainability and readability. In Doppar, views are typically stored in the resources/views directory. The templating system in Doppar allows you to create dynamic and reusable UI components efficiently. A basic view file might look like this:
```html
<!-- View stored in resources/views/greeting.odo.php -->
<html>
    <body>
        <h1>Hello, [[ $name ]]</h1>
    </body>
</html>
```

Since this view is stored at `resources/views/greeting.odo.php`, we may return it using the global view helper like so:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```