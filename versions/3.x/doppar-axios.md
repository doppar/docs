---
title: Axios
description: Doppar Axios page
meta:
  - name: keywords
    content: Axios
---

## Axios - Fluent HTTP Client for PHP

### Introduction

**Doppar Axios** is a modern, feature-rich HTTP client for PHP, inspired by the simplicity of JavaScript clients like Axios but built on top of Symfony's robust `HttpClient`. Does not depends on doppar internal core, you can use it any PHP application. It does not depend on any internal Doppar core, making it suitable for use in any PHP application.

## Features

- **Fluent Interface** – Clean, chainable syntax for building requests  
- **Dual Mode** – Supports both synchronous & asynchronous operations  
- **Batch Processing** – Parallel request handling with simple array syntax  
- **Robust Downloader** – Streamed file downloads with progress tracking  
- **File Upload Support** – Multipart form data handling for single and multiple files  
- **Automatic Retries** – Configurable retry logic for failed requests  
- **Middleware Support** – Extensible request/response processing  
- **Error Handling** – Comprehensive exception hierarchy  
- **Response Helpers** – Built-in JSON parsing, status checks, and callbacks  
- **No Framework Lock-in** – Works in any modern PHP application
## Why Doppar Axios?

- Built on **Symfony HttpClient** for performance and reliability
- **Memory-efficient** with streamed response handling
- **Familiar and intuitive** API similar to JS/Node ecosystems
- Handles real-world issues like timeouts, retries, and large file downloads
- Production-ready with composable, extensible design

## Perfect For:

- REST API integrations
- Web scraping tasks
- Microservice communication
- File downloads/uploads
- Any PHP app needing advanced HTTP features

## Installation
You may install Doppar Axios via the `composer require` command:
```bash
composer require doppar/axios
```

## Basic GET Request
Doppar Axios makes HTTP requests intuitive and expressive. Inspired by JavaScript's Axios, this PHP client wraps Symfony’s powerful HttpClient component under a fluent, chainable interface. When you call:

Here's what happens behind the scenes:
- The `to()` method sets the target URL for the request.
- The `get()` method defines the HTTP verb and dispatches the request synchronously.
- Internally, Doppar Axios uses Symfony's HTTP client for the actual transport.

Response handling is deferred until you call methods like `json()`, `status()`, or `body()`.

```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')->get();

$status = $response->status();

$data = $response->json();
```
The `status()` method returns the HTTP status code from the response, such as `200` for a successful request. The `json()` method decodes the JSON response body into a PHP array or object for easy access and manipulation.

##### Chained Example
You can also write it more concisely by chaining the calls:
```php
$data = Axios::to('https://example.com/posts')
    ->get()
    ->json();
```

This will return the decoded JSON content of the response directly.

When working with HTTP responses, you can use the following methods to easily access and evaluate the response content and status:
```php
$response->body() : string;
$response->object() : object;
$response->collect($key = null) : Phaseolies\Support\Collection;
$response->successful() : bool;
$response->redirect(): bool;
$response->failed() : bool;
$response->clientError() : bool;
$response->header($header) : string;
$response->headers() : array;
```

## Scoping and Reusing Clients
Axios supports scoped clients, allowing you to define reusable, preconfigured client instances. This improves consistency, reduces duplication, and enables context-specific overrides.

These methods clone the base client with merged options, so the original remains unchanged — perfect for building layered configurations in a clean, immutable style.

Here is the basic example you can use scope like this way
```php
use Doppar\Axios\Http\Axios;

// Base client
$client = Axios::withBaseUrl('https://api.example.com')
    ->withHeaders([
        'Accept' => 'application/json',
        'Authorization' => 'Bearer abc123'
    ]);

// Add scope for a specific microservice
$scopedClient = $client->withScope([
    'headers' => ['X-Service-Tag' => 'serviceA'],
    'query' => ['debug' => 'true'],
]);

$response = $scopedClient->to('/data')->get()->json();
```

Use `withBaseUrl()` to set or override the base URI for the client. All subsequent calls to `to()` will be relative to this base URL unless overridden.

```php
$client = Axios::withBaseUrl('https://example.com');

// Relative path is resolved against the base URL
$response = $client->to('/posts')->get()->json();
```

You can layer more scopes or use different base URLs for different services.
```php
$adminClient = $client->withBaseUrl('https://api.example.com/admin');
$response = $adminClient->to('/users')->get()->json();
```

Below is a complete example that demonstrates how to build a base client and reuse it with scoped modifications in different contexts, such as user requests and admin-level operations:
```php
// Create a base configured client
$client = Axios::withBaseUrl('https://jsonplaceholder.typicode.com')
    ->withHeaders(['Accept' => 'application/json'])
    ->withBearerToken($token)
    ->timeout(10.0);

// Reuse with different endpoints
$posts = $client->to('/posts')->get()->json();
$comments = $client->to('/comments')->get()->json();

// Create a more specific scope for admin access
$adminClient = $client
    ->withHeaders(['X-Admin' => 'true'])
    ->withQuery(['debug' => 'true']);

$adminData = $adminClient->to('/admin/stats')->get()->json();
```

## HTTP Version Controlling
Axios lets you control the HTTP protocol version on a per‑client basis. Whether you need to force` HTTP/2` for performance or fall back to `HTTP/1.1` for compatibility, these methods let you scope a client instance with your desired HTTP version—without mutating the original.

You can use `withHttp2()` to enable `HTTP/2` for your requests. `HTTP/2` offers multiplexing, header compression, and lower latency—ideal for modern APIs and high‑throughput scenarios.
```php
use Doppar\Axios\Http\Axios;

// Force HTTP/2 on HTTPS URLs (default behavior)
$client = Axios::withBaseUrl('https://api.example.com')
    ->withHttp2();

// Optionally, force HTTP/2 even for plain HTTP endpoints
$clientForceAll = Axios::withBaseUrl('http://intranet.local')
    ->withHttp2(true);

// The request will carry the HTTP/2 setting under the hood:
$response = $client->to('/v1/data')->get();
```

Revert to `HTTP/1.1` for maximum compatibility with legacy servers, proxies, or environments where `HTTP/2` may not be fully supported.
```php
use Doppar\Axios\Http\Axios;

// Create a base client that defaults to HTTP/1.1
$client = Axios::withBaseUrl('https://legacy.example.com')
    ->withoutHttp2();

$response = $client->to('/status')->get();
```
Each time a request is dispatched, Axios merges the result of `prepareHttp2Options()` into the Symfony client configuration:
```php
private function prepareHttp2Options(): array
{
    if ($this->httpVersion === null) {
        return [];
    }

    return [
        'http_version' => $this->httpVersion,
    ];
}
```
If no HTTP version has been scoped `$httpVersion === null`, Axios defers to Symfony’s default auto-negotiation. Otherwise, it enforces the specified protocol version.

See a basic full example of scoped HTTP versioning
```php
use Doppar\Axios\Http\Axios;

// Base client with auto-negotiation (default)
$defaultClient = Axios::withBaseUrl('https://api.example.com');

// Scoped for HTTP/2 on secure endpoints
$http2Client = $defaultClient->withHttp2();

// Scoped for legacy HTTP/1.1 only
$http1Client = $defaultClient->withoutHttp2();

// Make parallel requests with different versions
$response2 = $http2Client->to('/v2/stream')->get();
$response1 = $http1Client->to('/v1/legacy')->get();

// Inspect status codes
$status2 = $response2->status();  // e.g. 200
$status1 = $response1->status();  // e.g. 200
```
By scoping your HTTP version at the client level, you maintain a clean, immutable configuration model—letting different parts of your application use the protocol best suited to their needs.

## Handling Request Data
The `POST` method is used to send data to the server to create a new resource. This request typically includes the new resource's details in the request body. The server responds with the created resource, often including its new ID. Use the `post()` method to send data and create a new resource on the server. Doppar Axios automatically handles JSON serialization and returns a structured response for easy consumption.
```php
$response = Axios::to('https://example.com/posts')
    ->post([
        'title' => 'foo',
        'body' => 'bar',
        'userId' => 1
    ]);

$data = $response->json();

return $data;
```

The `PUT` method replaces the entire resource at the specified URL with the new data you provide. Use this when you want to fully overwrite an existing resource. The `put()` method replaces an existing resource with the provided data. Ideal for cases where the entire object needs to be overwritten on the server.
```php
$response = Axios::to('https://example.com/posts/1')
    ->put([
        'id' => 1,
        'title' => 'updated title',
        'body' => 'updated body',
        'userId' => 1
    ]);

$data = $response->json();

return $data;
```

The `PATCH` method is for partial updates. It changes only the fields you specify without replacing the whole resource. Use `patch()` when only part of the resource needs to be updated. Doppar Axios sends only the fields you define, making partial updates simple and efficient.
```php
$response = Axios::to('https://example.com/posts/1')
    ->patch([
        'title' => 'patched title'
    ]);

$data = $response->json();

return $data;
```

The `DELETE` method deletes the resource at the given URL. The server typically returns a status code indicating whether the deletion was successful. The `delete()` method sends a `DELETE` request to remove a resource. The response object allows inspection of status codes and any returned content.
```php
$response = Axios::to('https://example.com/posts/1')->delete();

return $response->status();
```

Always ensure that the API endpoint supports the HTTP method you intend to use. Handle responses appropriately by checking status codes and parsing returned data. For write operations like `POST`, `PUT`, `PATCH`, and `DELETE`, proper error handling is essential to manage failed requests gracefully.

## Handling Form Data
The Axios supports sending form-encoded data for HTTP requests such as `POST`, `PUT`, `PATCH`, and `DELETE`. This is commonly required when interacting with APIs expecting application/x-www-form-urlencoded data.
```php
Axios::to($this->getTokenUrl())
    ->asForm()
    ->post([
        //
    ]);
```
The `asForm()` method automatically sets the Content-Type header to:
```
Content-Type: application/x-www-form-urlencoded
```
It also converts the payload to a properly URL-encoded format, ensuring compatibility with most OAuth 2.0 token endpoints and APIs.

## with Query Parameters
Doppar Axios makes it easy to include query parameters in your requests using the `withQuery()` method. This method appends parameters to the request URL automatically, eliminating the need to manually build query strings.

Behind the scenes, `withQuery()` sets Symfony’s query option, ensuring that parameters are encoded and appended safely to the URL during request dispatch.

Example: Fetch Comments for a Specific Post
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/comments')
    ->withQuery(['postId' => 1])
    ->get()
    ->json();
```
The result will be an array of comments related to `postId = 1`, like:

## Sending Custom Headers
Doppar Axios allows you to send custom HTTP headers easily using the `withHeaders()` method. This is especially useful for sending tokens, API keys, or custom request metadata.

Under the hood, `withHeaders()` merges your provided headers into Symfony’s HTTP client configuration. You can call it multiple times to add or override headers dynamically.

Add a Custom Header
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->withHeaders([
        'X-Test-Header' => 'Value'
    ])
    ->get();

return $response->json();
```

You can combine multiple headers and even set them conditionally:
```php
Axios::to('https://example.com/api')
    ->withHeaders([
        'Authorization' => 'Bearer ' . $token,
        'Accept' => 'application/json',
    ])
    ->get()
    ->json();
```

### with Basic Authentication
Many APIs use Basic Authentication for secure access. With Doppar Axios, you can easily attach basic auth credentials using the `withBasicAuth()` method.

Under the hood, this sets Symfony's `auth_basic` option, which handles encoding the credentials into the correct Authorization: Basic ... header automatically.

Example: Authenticate with Username & Password
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->withBasicAuth('username', 'password')
    ->get();

return $response->json();
```

### with Bearer Authentication
Many modern APIs use Bearer Token Authentication (often in OAuth 2.0 flows). Doppar Axios provides a simple way to attach a bearer token to your request using the `withBearerToken()` method.

Internally, this sets the Authorization: `Bearer <token>` header using Symfony’s `auth_bearer` configuration option—ensuring secure and standards-compliant behavior.

Example: Send Bearer Token with Request
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->withBearerToken('fake-token')
    ->get();

return $response->json();
```

You can conditionally attach a token if available:
```php
$token = Auth::user()?->api_token;

Axios::to('https://example.com/data')
    ->ifSet($token, fn($req) => $req->withBearerToken($token))
    ->get()
    ->json();
```

## Inline Response Handling
Doppar Axios supports inline response hooks using `then()` and `catch()` methods. These allow you to define behavior based on the HTTP status code without breaking the fluent chain or writing separate if statements.

How It Works
- `then(callable)` will be triggered if the response status code is in the 2xx range
- `catch(callable)` will be triggered for 4xx or 5xx status codes

Both callbacks receive the raw response object, giving you full control

Handle Response Status Dynamically
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->get()
    ->then(function ($res) {
        echo "Success: " . $res->getStatusCode();
    })
    ->catch(function ($res) {
        echo "Failed: " . $res->getStatusCode();
    });

return $response->json();
```

Why Use It?
- Cleaner than writing if/else checks manually
- Keeps business logic close to the request itself
- Especially useful in APIs or CLI tools where you want real-time feedback

## with Custom Middleware
Doppar Axios lets you inject custom middleware using the `withMiddleware()` method. Middleware allows you to modify request options—such as headers, authentication, or logging—before the request is sent.

This gives you fine-grained control over request behavior and enables reusable logic like adding trace IDs, timestamps, debug flags, or custom headers dynamically.

Example: Add a Unique Trace Header
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->withMiddleware(function ($options) {
        $options['headers']['X-Trace-ID'] = uniqid();
        return $options;
    })
    ->get();

return $response->json();
```

You can call `withMiddleware()` multiple times to stack layers:
```php
Axios::to('https://example.com')
    ->withMiddleware($authInjector)
    ->withMiddleware($tracer)
    ->get();
```

Each middleware will be applied in the order you define them.

## with Automatic Retries
Real-world APIs can sometimes fail due to network issues, rate limiting, or temporary server errors. Doppar Axios provides built-in retry support via the `retry()` method, allowing you to automatically re-attempt failed requests.

How It Works
- `retry($maxRetries, $delayInMs)` configures the number of retry attempts and the delay (in milliseconds) between them.
- Retries are triggered only on:
  - Network-level errors (e.g., timeouts, unreachable host)
  - Server-side errors (status code 5xx)
- Retries are not triggered on 4xx errors (e.g., bad request, unauthorized).

Example: Retry a Request Up to 3 Times
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->retry(3, 100) // 3 retries with 100ms delay between attempts
    ->get()
    ->json();

return $response;
```
You can increase the delay for exponential backoff or adjust retries per endpoint:
```php
Axios::to('https://example.com/data')
    ->retry(5, 500) // up to 5 attempts, 0.5s apart
    ->get();
```

## with Request Timeout
In network communication, it’s best practice to set a timeout to avoid waiting forever for a slow or unresponsive server. With Doppar Axios, you can easily define the maximum number of seconds a request should wait using the `timeout()` method.

Example: Abort Request After 5 Seconds
```php
use Doppar\Axios\Http\Axios;

$response = Axios::to('https://example.com/posts')
    ->timeout(5) // max 5 seconds to complete the request
    ->get();

return $response->json();
```

##### What Happens If Timeout Is Reached?

If the server doesn’t respond in time:
- A NetworkException is thrown internally
- You can catch and handle this if needed:
```php
try {
    $data = Axios::to('https://example.com')
        ->timeout(2)
        ->get()
        ->json();
} catch (\Doppar\Axios\Exceptions\NetworkException $e) {
    // Log or handle timeout error
}
```

## Asynchronous Batch Requests
Doppar Axios supports sending multiple HTTP requests in parallel using `async()` and `wait()`. This is useful for speeding up APIs that don’t depend on each other, like fetching multiple resources at once.

Key Methods
- `async()` — enables async mode
- `to([...])` — accepts an array of URLs for batch processing
- `wait()` — waits for all async requests to complete
- `wait(true)` — decodes all responses to JSON automatically

### Asynchronous Batch with Raw Responses
When you need full control over the HTTP response objects—such as accessing headers, raw body content, or handling custom parsing—you can omit the `true` parameter in `wait()`. This returns the native response objects as-is. Each response retains Symfony’s full response API.
```php
use Doppar\Axios\Http\Axios;

[$products, $suppliers] = Axios::async()
    ->to([
        'https://example.com/products',
        'https://example.com/suppliers'
    ])
    ->get()
    ->wait();

dd($products->getContent(), $suppliers->getContent());
```

###  Async Batch with Auto JSON Decoding
Doppar Axios makes concurrent HTTP requests effortless with its fluent `async()` mode. By passing an array of URLs to `to()`, multiple GET requests are executed in parallel, significantly improving performance for I/O-bound operations. The `wait(true)` method ensures all responses are resolved and automatically decoded from JSON, returning structured PHP arrays or objects for immediate use.
```php
[$products, $suppliers] = Axios::async()
    ->to([
        'https://example.com/products',
        'https://example.com/suppliers'
    ])
    ->get()
    ->wait(true);

dd($products, $suppliers);
```
> If you call `json()` before `wait()` on async batches, it will throw an exception. You can still chain `withHeaders()`, `withBearerToken()`, etc., even in batch mode.

## Handle Redirection
The `withFollowRedirects()` method enables automatic following of HTTP redirects. The $max parameter defines the maximum number of redirects to follow before failing.
```php
$client = Axios::withBaseUrl('https://example.com')
    ->withFollowRedirects(3);

$response = $client->to('/redirect')->get();
```

## Enable or Disable SSL Certificate
Use the `withVerifyPeer()` method to explicitly enable or disable SSL certificate verification, particularly useful when working with internal services or self-signed certs.
```php
// Disable SSL verification (use with caution in production!)
$client = Axios::withBaseUrl('https://localhost')
    ->withVerifyPeer(false);

$response = $client->to('/api/data')->get();
```

## File Downloads
Doppar Axios supports efficient file downloading using `stream mode`. This is ideal for large files and avoids loading the entire content into memory. You can perform downloads with or without progress feedback.

Key Features
- `Streamed download:` Memory-efficient even for large files
- `Progress callback:` Receive updates during the download
- `Auto directory creation:` Ensures destination folder exists
- `Robust error handling:` Cleans up incomplete or failed downloads

#### Basic File Download
To save the contents of a URL to a file:
```php
use Doppar\Axios\Http\Axios;

$url = 'https://example.com/file.jpg';
$savePath = __DIR__ . '/downloads/sample.jpg';

Axios::to($url)
    ->get()
    ->download($savePath);

echo "Download completed: $savePath\n";
```

#### Download with Progress Tracking
For large files or to enhance UX, you can attach a progress callback using `withProgress()`:
```php
use Doppar\Axios\Http\Axios;

$url = 'https://example.com/file.jpg';
$savePath = __DIR__ . '/downloads/sample.jpg';

$response = Axios::to($url)
    ->withProgress(function ($downloaded, $total) {
        $percent = $total > 0 ? round($downloaded / $total * 100) : 0;
        echo "Progress: $percent%\r";
    })
    ->get()
    ->download($savePath);

echo "\nDownload completed: $savePath\n";
```

Safety & Validation
- If the file cannot be saved, Axios throws a NetworkException.
- If the server responds with HTML or an invalid type, Axios detects it and fails early.
- Incomplete downloads are cleaned up automatically to avoid corrupted files.

## File Uploads (Multipart/Form-Data)
Doppar Axios supports multipart form uploads for sending files along with other form fields. Use `withMultipart()` to build the multipart request body easily.

#### Basic File Upload
Upload a single file with additional data using `\SplFileInfo:`
```php
use Doppar\Axios\Http\Axios;

Axios::to('https://example.com/upload')
    ->withMultipart([
        'file' => new \SplFileInfo('/path/to/file.jpg'),
        'description' => 'My file description'
    ])
    ->post();
```
You can also use other HTTP methods:
```php
Axios::to('https://example.com/upload')
    ->withMultipart([
       //
    ])
    ->put();

Axios::to('https://example.com/upload')
    ->withMultipart([
        //
    ])
    ->patch();
```

## Multiple Files and Nested Fields
Upload multiple files and complex data structures in a single `multipart/form-data` request. Doppar Axios handles array inputs by automatically serializing them into multiple form parts with the appropriate field names `(e.g., files[])`.
```php
Axios::to('https://example.com/upload')
    ->withMultipart([
        'files' => [
            new \SplFileInfo('/path/to/file1.jpg'),
            new \SplFileInfo('/path/to/file2.jpg')
        ],
        'user' => [
            'name' => 'John',
            'email' => 'john@example.com'
        ]
    ])
    ->post();
```

This request sends:
- Two files under the `files[]` field.
- Nested user information `(user[name], user[email])` serialized correctly in the form data.

> All files provided via `\SplFileInfo` must be readable on the server where this code runs. If a file cannot be read, an InvalidArgumentException is thrown immediately to prevent failed uploads.