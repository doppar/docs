---
title: Responses
description: Doppar Responses page
meta:
  - name: keywords
    content: Responses
---

## HTTP Responses

### Introduction

Every web application communicates with clients through HTTP responses. Whether you're building a REST API, a traditional web application, or a hybrid system, crafting proper responses is fundamental to creating reliable software.

The Doppar framework provides a powerful and expressive response system that handles everything from simple text output to complex streamed data transfers. The response layer abstracts HTTP protocol complexities while giving you fine-grained control when you need it.

This guide covers all aspects of HTTP responses in Doppar: basic response types, headers, status codes, redirects, file handling, streaming, caching, and advanced response patterns. By the end, you'll understand how to leverage Doppar's response system to build robust, efficient applications.

## Basic Response Concepts

### Understanding the Response Lifecycle

When a client makes a request to your application, Doppar processes that request through your routes and controllers. The value you return from these handlers determines what gets sent back to the client.

Doppar automatically converts your return values into proper HTTP responses:

- **Strings** become HTML responses
- **Arrays** and **objects** become JSON responses
- **Null** becomes an empty response
- **Response objects** are sent as-is with full control

## The Response Helper

The `response()` helper function is your gateway to the response system. Called without arguments, it returns a `ResponseFactory` instance that provides access to all response creation methods:

```php
use Phaseolies\Routing\Attributes\Route;

class HomeController
{
    #[Route(uri: '/')]
    public function index()
    {
        return response('Hello World');
    }
}
```

With arguments, it creates a response directly:

```php
#[Route(uri: '/welcome')]
public function welcome()
{
    return response('Welcome to Doppar', 200, [
        'Content-Type' => 'text/plain'
    ]);
}
```

## Simple Response Types

#### Returning Strings

The simplest response is a plain string. Doppar wraps it in a proper HTTP response with appropriate headers:

```php
#[Route(uri: '/hello')]
public function hello()
{
    return "Hello, World!";
}
```

This automatically sets the `Content-Type` header to `text/html` and returns a `200` status code.

#### Returning Arrays and Objects

When you return an array or object, Doppar automatically serializes it to JSON and sets the appropriate Content-Type header:

```php
#[Route(uri: '/user')]
public function getUser()
{
    return [
        'name' => 'Alex Thompson',
        'email' => 'alex@example.com',
        'role' => 'admin'
    ];
}
```

Response:

```json
{
  "name": "Alex Thompson",
  "email": "alex@example.com",
  "role": "admin"
}
```

#### Returning Collections

Doppar's collections can be returned directly from routes. They're automatically converted to JSON:

```php
#[Route(uri: '/numbers')]
public function getNumbers()
{
    return collect([1, 2, 3, 4, 5]);
}
```

#### Returning Entity Models

Database entity models and collections are automatically serialized, respecting any hidden or protected attributes you've defined:

```php
use App\Models\User;

#[Route(uri: '/users')]
public function getAllUsers()
{
    return User::all();
}
```

Response:

```json
[
  {
    "id": 1,
    "name": "John Doe",
    "email": "john@example.com",
    "role": "admin"
  },
  {
    "id": 2,
    "name": "Jane Doe",
    "email": "jane@example.com",
    "role": "user"
  }
]
```

#### Returning Null

If your handler returns `null`, Doppar converts it to an empty string response with a `200` status code:

```php
#[Route(uri: '/empty')]
public function emptyResponse()
{
    return null;
}
```

## Response Objects

For full control over your HTTP response, create explicit Response objects. This gives you access to status codes, headers, and advanced features:

```php
#[Route(uri: '/custom')]
public function customResponse()
{
    return response('Custom response content', 201)
        ->setHeader('X-Custom-Header', 'CustomValue');
}
```

#### Using the Response Factory

The `ResponseFactory` provides convenient methods for creating different response types:

```php
#[Route(uri: '/factory-example')]
public function factoryExample()
{
    // Create a text response
    return response()->text('Plain text content', 200);

    // Create a JSON response
    return response()->json(['status' => 'success'], 200);

    // Create an empty response
    return response()->noContent();
}
```

#### Injecting the Response Object

You can type-hint the `Response` object in your controller methods for direct access:

```php
use Phaseolies\Http\Response;

class ApiController
{
    #[Route(uri: '/api/data')]
    public function getData(Response $response)
    {
        return $response->json([
            'data' => $this->fetchData(),
            'timestamp' => time()
        ], 200);
    }
}
```

## Working with Headers

Remember, most response methods in Doppar are chainable, enabling you to build response objects in a fluent, readable style. For instance, you can use the setHeader method to attach multiple headers to the response before it's returned to the client:

#### Setting Individual Headers

Use the `setHeader()` method to add a single header to your response:

```php
#[Route(uri: '/with-header')]
public function withHeader()
{
    return response('Content with custom header')
        ->setHeader('X-API-Version', '1.0')
        ->setHeader('X-Request-ID', uniqid());
}
```

#### Setting Multiple Headers

The `withHeaders()` method accepts an array of headers for cleaner syntax:

```php
#[Route(uri: '/multiple-headers')]
public function multipleHeaders()
{
    return response('Response with multiple headers')
        ->withHeaders([
            'X-API-Version' => '1.0',
            'X-Request-ID' => uniqid(),
            'X-Rate-Limit' => '1000',
            'X-Rate-Remaining' => '999'
        ]);
}
```

#### Common Headers

Here are frequently used HTTP headers you might need:

```php
#[Route(uri: '/common-headers')]
public function commonHeaders()
{
    return response($content)
        ->withHeaders([
            'Content-Type' => 'application/json',
            'Cache-Control' => 'no-cache, no-store, must-revalidate',
            'Pragma' => 'no-cache',
            'Expires' => '0',
            'X-Frame-Options' => 'DENY',
            'X-Content-Type-Options' => 'nosniff',
            'X-XSS-Protection' => '1; mode=block'
        ]);
}
```

## HTTP Status Codes

Control the HTTP status code using the second parameter of `response()` or the `setStatusCode()` method:

```php
#[Route(uri: '/created')]
public function resourceCreated()
{
    // Using response() parameters
    return response('Resource created', 201);
}

#[Route(uri: '/not-found')]
public function notFound()
{
    // Using setStatusCode()
    return response('Resource not found')
        ->setStatusCode(404);
}
```

#### Common Status Codes

Success Responses (2xx)

```php
// 200 OK - Standard success response
return response($data, 200);

// 201 Created - Resource successfully created
return response(['id' => 123], 201);

// 202 Accepted - Request accepted for processing
return response(['message' => 'Processing started'], 202);

// 204 No Content - Success with no response body
return response()->noContent();
```

Redirection Responses (3xx)

```php
// 301 Moved Permanently
return response()->redirect('/new-location', 301);

// 302 Found (Temporary redirect)
return response()->redirect('/temporary-location', 302);

// 304 Not Modified (for caching)
return response()->setNotModified();
```

Client Error Responses (4xx)

```php
// 400 Bad Request
return response('Invalid input data', 400);

// 401 Unauthorized
return response('Authentication required', 401);

// 403 Forbidden
return response('Access denied', 403);

// 404 Not Found
return response('Resource not found', 404);

// 422 Unprocessable Entity
return response(['errors' => $validationErrors], 422);

// 429 Too Many Requests
return response('Rate limit exceeded', 429);
```

Server Error Responses (5xx)

```php
// 500 Internal Server Error
return response('Something went wrong', 500);

// 503 Service Unavailable
return response('Service unavailable', 503);

// 504 Gateway Timeout
return response('Gateway timeout', 504);
```

## Checking Response Status

The `Response` class provides methods to check the status category:

```php
$response = response()->json(['data' => 'test'], 200);

// Check specific status
if ($response->isOk()) {
    // Status is exactly 200
}

// Check status categories
$response->isSuccessful();    // 2xx
$response->isRedirection();   // 3xx
$response->isClientError();   // 4xx
$response->isServerError();   // 5xx

// Check specific statuses
$response->isForbidden();     // 403
$response->isNotFound();      // 404
$response->isRedirect();      // Redirect status codes

// Check validity
$response->isInvalid();       // Status < 100 or >= 600
$response->isInformational(); // 1xx
$response->isEmpty();         // 204 or 304
```

## JSON Responses

#### Basic JSON Responses

The `json()` method automatically sets the Content-Type header to `application/json` and encodes your data:

```php
#[Route(uri: '/api/user')]
public function getUser()
{
    return response()->json([
        'id' => 1,
        'name' => 'John Doe',
        'email' => 'john@example.com'
    ]);
}
```

#### JSON with Status Codes

Include HTTP status codes for proper API responses:

```php
#[Route(uri: '/api/users', method: 'POST')]
public function createUser()
{
    $user = $this->userService->create($this->request->all());

    return response()->json([
        'message' => 'User created successfully',
        'data' => $user
    ], 201);
}
```

## Working with Collections

Collections automatically convert to arrays when returned as JSON:

```php
use App\Models\User;

#[Route(uri: '/api/users')]
public function listUsers()
{
    $users = User::all();

    return response()->json([
        'data' => $users,
        'count' => $users->count()
    ]);
}
```

#### Manual Array Conversion

If you need explicit control over the array structure:

```php
#[Route(uri: '/api/users/array')]
public function usersAsArray()
{
    return response()->json([
        'data' => User::all()->toArray(),
        'meta' => [
            'timestamp' => time(),
            'version' => '1.0'
        ]
    ]);
}
```

Build complex nested JSON responses:

```php
#[Route(uri: '/api/users/nested')]
public function nestedUsers()
{
    return response()->json([
        'data' => User::all()->toArray(),
        'meta' => [
            'timestamp' => time(),
            'version' => '1.0'
        ],
        'links' => [
            'self' => '/api/users',
            'next' => '/api/users?page=2'
        ]
    ]);
}
```

## Redirects Response

Redirect responses are instances of the `Phaseolies\Http\RedirectResponse` class, and contain the proper headers needed to redirect the user to another URL. There are several ways to generate a Redirect instance. The simplest method is to use the global redirect helper or even you can use `Phaseolies\Support\Facades\Redirect` facades

#### Basic Redirects

Redirect users to different URLs using the redirect helper:

```php
#[Route(uri: '/old-page')]
public function oldPage()
{
    return redirect('/new-page');
}
```

Specify the redirect type with HTTP status codes:

```php
#[Route(uri: '/moved')]
public function movedPermanently()
{
    // 301 Permanent redirect
    return redirect('/new-location', 301);
}

#[Route(uri: '/temp')]
public function temporaryRedirect()
{
    // 302 Temporary redirect
    return redirect('/temporary-location', 302);
}
```

You can also use the `to()` method like this way

```php
return redirect()->to('/home/dashboard');
```

#### Redirect to Named Routes

When you call the redirect helper with no parameters, an instance of `Phaseolies\Routing\RedirectResponse` is returned, allowing you to call any method on the Redirect instance. For example, to generate a Redirect to a named route, you may use the route method:

Redirect to routes by their name instead of hardcoding URLs:

```php
#[Route(uri: '/login', name: 'login')]
public function loginPage()
{
    return view('auth.login');
}

#[Route(uri: '/authenticate', method: 'POST')]
public function authenticate()
{
    if (!$this->isAuthenticated()) {
        return redirect()->route('login');
    }

    return redirect()->route('dashboard');
}
```

#### Redirect with Route Parameters

If your route has parameters, you may pass them as the second argument to the route method:

```php
#[Route(uri: '/profile/{id}/{username}', name: 'profile')]
public function profile($id, $username)
{
    return view('profile', compact('id', 'username'));
}

#[Route(uri: '/view-profile')]
public function viewProfile()
{
    return redirect()->route('profile', [
        'id' => 1,
        'username' => 'johndoe'
    ]);
    // Redirects to: /profile/1/johndoe
}
```

## Redirect Back

Sometimes you may wish to redirect the user to their previous location, such as when a submitted form is invalid. You may do so by using the global back helper function.

Redirect users to their previous location:

```php
#[Route(uri: '/cancel')]
public function cancel()
{
    return redirect()->back();
}
```

Or use the global `back()` helper:

```php
#[Route(uri: '/cancel')]
public function cancel()
{
    return back();
}
```

You can chain the flash message with the redirect:

```php
return redirect()->back()->with('error', 'Error messages');
```

### Redirecting to External Domains

Sometimes you may need to redirect to a domain outside of your application. You may do so by calling the `away` method, which creates a `RedirectResponse` without any additional URL encoding, validation, or verification:

```php
return redirect()->away('https://www.google.com');
```

### Redirecting With Flashed Session Data

Redirecting to a new URL and flashing data to the session are usually done at the same time. Typically, this is done after successfully performing an action when you flash a success message to the session. For convenience, you may create a `RedirectResponse` instance and flash data to the session in a single, fluent method chain:

```php
#[Route(uri: '/save', method: 'POST')]
public function save()
{
    $this->saveData();

    return redirect('/dashboard')->with('success', 'Data saved successfully!');
}
```

After the user is redirected, you may display the flashed message from the session. For example, using Odo syntax:

```html
#if (session()->has('success'))
<div class="alert alert-success">[[ session()->pull('success') ]]</div>
#endif
```

### Multiple Flash Messages

You can flash multiple messages to the session by passing an array to the `with` method:

```php
#[Route(uri: '/process', method: 'POST')]
public function process()
{
    return redirect('/results')
        ->with('message', 'Processing complete')
        ->with('processedCount', 150)
        ->with('warnings', ['Warning 1', 'Warning 2']);
}
```

## View Responses

If you need control over the response's status and headers but also need to return a view as the response's content, you should use the view method:

```php
return response()->view('welcome')
    ->setHeader('Content-Type', 'text/html');
```

You can set headers also using `withHeaders` methods like this way

```php
return view('welcome')->withHeaders(['Content-Type' => 'text/html']);
```

## File Responses

The file method may be used to display a file, such as an image or PDF, directly in the user's browser instead of initiating a download. This method accepts the absolute path to the file as its first argument and an array of headers as its second argument:

#### Displaying Files

Display files directly in the browser (images, PDFs, etc.) without triggering a download:

```php
#[Route(uri: '/file/{filename}')]
public function showFile($filename)
{
    $path = storage_path("files/{$filename}");

    return response()->file($path);
}
```

Add custom headers to file responses:

```php
#[Route(uri: '/document/{id}')]
public function showDocument($id)
{
    $document = Document::find($id);
    $path = storage_path("documents/{$document->filename}");

    return response()->file($path, [
        'Content-Type' => 'application/pdf',
        'Cache-Control' => 'public, max-age=3600'
    ]);
}
```

#### Inline vs Attachment

Control whether files are displayed inline or downloaded:

```php
#[Route(uri: '/image/{id}')]
public function displayImage($id)
{
    $path = storage_path("images/{$id}.jpg");

    // Display inline in browser
    return response()->file($path, [
        'Content-Disposition' => 'inline; filename="image.jpg"'
    ]);
}
```

## File Downloads Response

The download method generates a response that triggers a file download in the user’s browser. It takes the file path as its primary argument. Optionally, you can specify a custom download filename as the second argument—this overrides the default name seen by the user. Additionally, an array of custom HTTP headers can be passed as a third argument for further control over the download behavior.

Trigger file downloads with the `download()` method:

```php
#[Route(uri: '/download/{file}')]
public function download($file)
{
    $path = storage_path("downloads/{$file}");

    return response()->download($path);
}
```

Specify a custom filename that differs from the actual file:

```php
#[Route(uri: '/export/users')]
public function exportUsers()
{
    $path = storage_path('exports/users.csv');

    return response()->download($path, 'user-export-' . date('Y-m-d') . '.csv');
}
```

Add custom headers to downloads:

```php
#[Route(uri: '/secure-download/{id}')]
public function secureDownload($id)
{
    $file = SecureFile::find($id);
    $path = storage_path("secure/{$file->filename}");

    return response()->download($path, $file->original_name, [
        'Content-Type' => $file->mime_type,
        'X-Download-ID' => $id
    ]);
}
```

> Doppar usages Symfony HttpFoundation `BinaryFileResponse` class, which manages file downloads, requires the file being downloaded to have an `ASCII` filename.

## Streamed Downloads

At times, you may want to convert the string output of an operation into a downloadable response without storing it on disk. The `streamDownload` method allows you to achieve this by accepting a callback, filename, and an optional array of headers as parameters:

Generate and download content on-the-fly without saving to disk:

```php
#[Route(uri: '/stream-download/csv')]
public function streamCsvDownload()
{
    return response()->streamDownload(function() {
        $handle = fopen('php://output', 'w');

        // Write CSV headers
        fputcsv($handle, ['ID', 'Name', 'Email', 'Created']);

        // Stream data from database
        User::chunk(100, function($users) use ($handle) {
            foreach ($users as $user) {
                fputcsv($handle, [
                    $user->id,
                    $user->name,
                    $user->email,
                    $user->created_at
                ]);
            }
        });

        fclose($handle);
    }, 'users-export.csv');
}
```

Stream large JSON datasets:

```php
#[Route(uri: '/stream-download/json')]
public function streamJsonDownload()
{
    return response()->streamDownload(function() {
        echo '[';

        $first = true;
        User::chunk(100, function($users) use (&$first) {
            foreach ($users as $user) {
                if (!$first) echo ',';
                echo json_encode($user);
                $first = false;
            }
        });

        echo ']';
    }, 'users-export.json', [
        'Content-Type' => 'application/json'
    ]);
}
```

Streamed Downloads with Headers

```php
#[Route(uri: '/stream-download/report')]
public function streamReportDownload()
{
    $filename = 'report-' . date('Y-m-d-His') . '.txt';

    return response()->streamDownload(function() {
        echo "Report Generated: " . date('Y-m-d H:i:s') . "\n\n";

        // Stream large dataset
        foreach ($this->generateReportData() as $line) {
            echo $line . "\n";
            ob_flush();
            flush();
        }
    }, $filename, [
        'Content-Type' => 'text/plain',
        'X-Report-Version' => '1.0'
    ]);
}
```

## Streamed Responses

Streaming data to the client as it is generated can greatly reduce memory usage and enhance performance, particularly for extremely large responses.

#### Basic Streaming

Stream content to the client as it's generated to reduce memory usage:

```php
#[Route(uri: '/stream')]
public function stream()
{
    return response()->stream(function() {
        echo "Starting stream...\n";
        ob_flush();
        flush();

        for ($i = 1; $i <= 10; $i++) {
            echo "Chunk {$i}\n";
            ob_flush();
            flush();
            sleep(1); // Simulate processing delay
        }

        echo "Stream complete.\n";
    }, 200, [
        'Content-Type' => 'text/plain',
        'X-Accel-Buffering' => 'no' // Disable proxy buffering
    ]);
}
```

Use PHP generators for cleaner streaming code:

```php
class StreamController
{
    #[Route(uri: '/stream/generator')]
    public function streamWithGenerator()
    {
        return response()->stream(function() {
            foreach ($this->dataGenerator() as $chunk) {
                echo $chunk;
                ob_flush();
                flush();
            }
        }, 200, [
            'Cache-Control' => 'no-cache',
            'X-Accel-Buffering' => 'no'
        ]);
    }

    private function dataGenerator(): \Generator
    {
        yield "Starting process...\n";

        foreach (range(1, 100) as $num) {
            yield "Processing item {$num}\n";
            usleep(100000); // 100ms delay
        }

        yield "Process complete.\n";
    }
}
```

Stream database results to avoid loading everything into memory:

```php
#[Route(uri: '/stream/users')]
public function streamUsers()
{
    return response()->stream(function() {
        echo "[\n";

        $first = true;
        User::chunk(100, function($users) use (&$first) {
            foreach ($users as $user) {
                if (!$first) echo ",\n";
                echo "  " . json_encode($user);
                ob_flush();
                flush();
                $first = false;
            }
        });

        echo "\n]";
    }, 200, [
        'Content-Type' => 'application/json',
        'X-Accel-Buffering' => 'no'
    ]);
}
```

## Streamed JSON Responses

To stream JSON data incrementally, you can use the `streamJson` method. This is particularly beneficial for `large datasets` that need to be sent progressively to the browser in a format that JavaScript can easily parse

Stream JSON data incrementally for large datasets:

```php
#[Route(uri: '/stream-json/users')]
public function streamJsonUsers()
{
    return response()->streamJson([
        'users' => User::all(),
        'total' => User::count()
    ]);
}
```

You can control JSON encoding behavior:

```php
#[Route(uri: '/stream-json/pretty')]
public function streamJsonPretty()
{
    return response()->streamJson(
        ['data' => $this->getData()],
        200,
        [],
        JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES
    );
}
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
#[Route(uri: '/greeting/{name}')]
public function greeting($name)
{
    return view('greeting', ['name' => $name]);
}
```

## Disposition Types

Control the download behavior with different disposition types:

```php
#[Route(uri: '/attachment/{id}')]
public function attachment($id)
{
    $path = storage_path("files/{$id}.pdf");

    // Force download (default)
    return response()->download($path, 'document.pdf', [], 'attachment');
}

#[Route(uri: '/inline/{id}')]
public function inline($id)
{
    $path = storage_path("files/{$id}.pdf");

    // Display inline
    return response()->download($path, 'document.pdf', [], 'inline');
}
```

## Delete After Download

In Doppar, when you use the `response()->download()` method, you can also chain the `deleteFileAfterSend()` method if you want the file to be deleted from the server after it has been sent to the user.

```php
#[Route(uri: '/temporary-download/{token}')]
public function temporaryDownload($token)
{
    return response()->download($path)->deleteFileAfterSend();
}
```

## Response Caching

Response caching lets browsers and proxy servers store your HTTP responses so future requests can be served faster. By setting appropriate cache headers, you reduce server load and improve performance for repeat visitors.

Control how responses are cached by browsers and proxies.

```php
#[Route(uri: '/cached-content')]
public function cachedContent()
{
    return response($content)
        ->setHeader('Cache-Control', 'public, max-age=3600');
}
```

### Convenience Methods for Common Caching Patterns

These helper methods make it easier to apply standard caching behavior without manually crafting headers. They improve readability and reduce the chance of configuration mistakes.

Use convenience methods for common caching patterns:

```php
#[Route(uri: '/public-cache')]
public function publicCache()
{
    return response($content)
        ->setPublic()
        ->setMaxAge(3600); // Cache for 1 hour
}

#[Route(uri: '/private-cache')]
public function privateCache()
{
    return response($userContent)
        ->setPrivate()
        ->setMaxAge(1800); // Cache for 30 minutes
}
```

## No-Cache Responses

Sometimes data must never be stored by browsers or intermediaries, such as sensitive or frequently changing information. These headers explicitly instruct clients and proxies not to cache the response.

Prevent caching entirely:

```php
#[Route(uri: '/no-cache')]
public function noCache()
{
    return response($sensitiveData)
        ->withHeaders([
            'Cache-Control' => 'no-cache, no-store, must-revalidate',
            'Pragma' => 'no-cache',
            'Expires' => '0'
        ]);
}
```

## Shared Cache Control

Shared cache settings are useful when using CDNs or reverse proxies. They allow you to specify different cache lifetimes for shared caches versus individual browsers, giving you more fine-grained control over performance.

Control caching for shared caches (CDNs, proxies):

```php
#[Route(uri: '/api/public-data')]
public function publicData()
{
    return response()->json($data)
        ->setSharedMaxAge(7200) // 2 hours for shared caches
        ->setMaxAge(3600);      // 1 hour for private caches
}
```

## Response Chaining

Method chaining allows you to build complex responses in a clean and readable way. It keeps related response configuration together and makes your controller methods easier to maintain.

Build complex responses through method chaining:

```php
#[Route(uri: '/api/protected')]
public function protected()
{
    return response()->json(['data' => $this->getData()])
        ->setStatusCode(200)
        ->withHeaders([
            'X-API-Version' => '1.0',
            'X-RateLimit-Limit' => '1000',
            'X-RateLimit-Remaining' => '999'
        ])
        ->setPublic()
        ->setMaxAge(300)
        ->setVary(['Authorization', 'Accept-Language']);
}
```

The Doppar response system provides everything you need to build robust web applications and APIs:
