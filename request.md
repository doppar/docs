## HTTP Requests
### Introduction
Doppar’s `Phaseolies\Http\Request` class offers a clean, object-oriented interface for working with the current HTTP request. It allows you to easily access request data such as input fields, cookies, headers, uploaded files, and more—all within a structured and expressive API tailored for modern web applications.

### Interacting With The Request
#### Accessing the Request
To access request data, Doppar has some default built in method. Doppar provides a powerful Request class to handle incoming HTTP requests. This class offers a wide range of methods to access and manipulate request data, making it easy to build robust web applications.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     */
    public function store(Request $request)
    {
        $name = $request->input('name');

        // Store the user...

        return redirect('/users');
    }
}
```

As mentioned, you may also type-hint the `Phaseolies\Http\Request` class on a route closure. The service container will automatically inject the incoming request into the closure when it is executed:
```php
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    // ...
});
```

### Dependency Injection and Route Parameters
If your controller method is also expecting input from a route parameter you should list your route parameters after your other dependencies. For example, if your route is defined like so:
```php
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

You may still type-hint the `Phaseolies\Http\Request` and access your id route parameter by defining your controller method as follows:
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;

class UserController extends Controller
{
    /**
     * Update the specified user.
     */
    public function update(Request $request, string $id)
    {
        // Update the user...

        return redirect('/users');
    }
}
```

### Interacting With HTML Form Request Data
Doppar makes working with form data intuitive and expressive via the `Phaseolies\Http\Request` class. Whether you're retrieving all inputs, checking for field presence, or filtering specific values, Doppar provides a rich API to simplify handling request data from HTML forms.
#### Accessing All Data
Retrieve the complete set of request input:
```php
$request->all();
```

#### Accessing Specific Field Values
Doppar provides various ways to interact with specific fields name. You can use any of them
```php
$request->name;
// Accesses the value of the 'name' field directly.

$request->input('name', 'default');
// Equivalent to the above, using the input() method.

$request->get('name', 'default');
// Same result, alternative syntax.
```

#### Retrieving Data with Exclusions
Doppar provides `except` method to interact with fields name. You can use like that to exclude some fields from the current requested data.

```php
$request->except('name');
// Returns all data except the 'name' field.

$request->except('name', 'age');
// Excludes multiple fields from the result.
```

#### Retrieving Specific Fields Only
Doppar provides `only` method to interact with fields name. You can use like that to get only those fields from the current requested data.
```php
$request->only('name');
// Returns only the 'name' field.

$request->only(['name', 'age']);
// Retrieves just the 'name' and 'age' fields.
```
#### Checking Field Existence
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


#### Matching Request Paths
Need to determine if the current request URL matches a specific pattern? The `is()` method allows you to compare the request path against a pattern with wildcard support. This is especially useful for applying conditional logic based on the current route.
```php
$request->is('/user/*');
// Matches routes like /user/profile, /user/settings, etc.
```

#### Accessing Validation Results
After validating form input, you might want to separate valid and invalid data. Doppar provides two handy methods for this:

- `passed():` Returns only the fields that passed validation.
- `failed():` Returns the fields that failed validation, typically used for error feedback.

```php
$request->passed();
// Returns only the data that passed validation.

$request->failed();
// Returns the data that failed validation checks.
```

### Accessing Session Data
Doppar's Request class provides seamless access to session data, allowing you to retrieve, store, and manage temporary user-specific data easily. This is particularly useful for flash messages, authentication info, and CSRF protection.

#### Retrieve CSRF Token
```php
$request->session()->token();
// Retrieves the CSRF token stored in the session.
```
Use this token to secure form submissions and protect against cross-site request forgery.

#### Get Session Value by Key
```php
$request->session()->get('key');
// Returns the value stored in the session under 'key'.
```

####  Set Session Data
```php
$request->session()->put('key', 'value');
// Stores 'value' in the session under the key 'key'.
```
You can use this to save custom data in the session for later retrieval.

#### Clear All Session Data
```php
$request->session()->flush();
// Deletes all session data.
```

#### Request Path, Host, and Method
The `Phaseolies\Http\Request` instance provides a variety of methods for examining the incoming HTTP request and usage some method from the `Symfony\Component\HttpFoundation\Request` class. We will discuss a few of the most important methods below.

#### Retrieving the Request Path
The path method returns the request's path information. So, if the incoming request is targeted at http://example.com/foo/bar, the path method will return `foo/bar`:
```php
$uri = $request->getPath();
```

#### Retrieving the Request URL
To retrieve the full URL for the incoming request you may use the url or fullUrl methods. The url method will return the URL without the query string, while the fullUrl method includes the query string:
```php
$url = $request->url();

$urlWithQueryString = $request->fullUrl();
```

#### Retrieving the Request Host
You may retrieve the "host" of the incoming request via the host, scheme methods:
```php
$request->host();
$request->scheme();
```

#### Retrieving the Request Method
The method method will return the HTTP verb for the request. You may use the isMethod method to verify that the HTTP verb matches a given string:
```php
$method = $request->method(); // return 'get'
$method = $request->getMethod(); // return 'GET'

if ($request->isGet()) {
    //
 }
```

You will get the method like `isPost()`, `isDelete()`, `isPatch()`, `isPut()` etc to check incoming HTTP request type.

### Request Headers
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

#### Request IP Address
The `ip` method may be used to retrieve the IP address of the client that made the request to your application
```php
$ipAddress = $request->ip();
```
To retrieve an array of IP addresses — including all client IPs forwarded by proxies — you can use the `ips()` method on the `Phaseolies\Http\Request` instance. This method returns all available IPs in order, with the original client IP listed at the end of the array.
```php
$ipAddresses = $request->ips();
```
This is especially useful when your application is behind a load balancer or proxy and you want to trace the full route of the request

#### Content Negotiation
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

#### Merging Additional Input
Sometimes you may need to manually `merge` additional input into the request's existing input data. To accomplish this, you may use the `merge` method. If a given input key already exists on the request, it will be overwritten by the data provided to the `merge` method:
```php
$request->merge(['country' => 'bd']);
```
#### Retrieving Old Input
Doppar provides a global `old` helper. If you are displaying old input within a Blade template, it is more convenient to use the old helper to repopulate the form. If no old input exists for the given field, null will be returned:
```html
<input type="text" name="username" value="{{ old('username') }}">
```

### Cookies
#### Retrieving Cookies From Requests
 To retrieve a cookie value from the request, use the cookie method on an `Phaseolies\Http\Request` instance:
 ```php
 $value = $request->cookies->get('name');
 ```

 You can check request cookie existance using `hasCookie` method like that
 ```php
 $request->hasCookie('key');
 ```

 ### Servers
 In the Doppar framework, you can retrieve server-related variables (like headers, host info, request method, etc.) using the `Phaseolies\Http\Request` object. There are two main ways to access this data:

#### Retrieve All Server Variables
To get all server variables as an associative array:
```php
$serverData = $request->server();
```

#### Retrieve a Specific Server Variable
To get a single server variable by its key (e.g., 'HTTP_HOST', 'REQUEST_METHOD'):
```php
$host = $request->server->get('HTTP_HOST');
```
This allows you to target individual values without loading the entire array.


### Query String
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

#### With Default Value (Optional):
```php
$city = $request->query('city', 'unknown');
```
returns 'unknown' if 'city' is not present.

Returns the entire query string as a raw string, similar to $_SERVER['QUERY_STRING'].
```php
$queryString = $request->getQueryString();
```

This will give you the raw query string `'name=mahedi&school=academy'`.

### Get Authenticated User
When a user is authenticated (typically via session, token, or doppar flarion), Doppar provides simple methods to retrieve their information through the request object.
```php
$request->auth(); // Retrieves the authenticated user data.
$request->user(); // Retrieves the authenticated user data.
```

### Files
#### Retrieving Uploaded Files
You may retrieve uploaded files from an `Phaseolies\Http\Request` instance using the `file` method.
#### Accessing the File Object:
```php
$image = $request->file('file');
```
Here 'file' is the HTML form input name.

#### Retrieving File Information
To get file information, doppar provides various methods like
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

#### Example
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

### Global Request helper
The request() helper function in Doppar provides a convenient and globally accessible way to retrieve data from the current HTTP request. It’s an alias for accessing the `Phaseolies\Http\Request` instance without needing to inject or typehint

#### Basic Usage
```php
$name = request('name', 'default'); // Equivalent of $request->name ?? 'default'
```

What ever you are doing using `Phaseolies\Http\Request` instance, you can do the same job using `request()` helper any where without injecting `Phaseolies\Http\Request` as method params.