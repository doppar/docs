## Routing: Getting Started
### Introduction
Doppar’s routing system, available through the Phaseolies\Support\Facades\Route namespace, provides a clean, expressive way to define your application’s URL structure and map it to the appropriate controller actions or closures. Inspired by modern frameworks but tailored for performance and clarity, Doppar routing makes it easy to build both small APIs and large-scale web applications.

With support for route prefix grouping, named routes, throttle route, middleware assignment, and RESTful resource routing, Doppar gives developers full control over how requests are handled—without unnecessary complexity. Whether you're defining a simple GET endpoint or building a full REST API, Doppar’s routing engine keeps your code concise, maintainable, and scalable.

### Supported HTTP Methods
Doppar supports the following HTTP methods for defining routes:
| **Method**  | **Description**                                                                  |
| ----------- | -------------------------------------------------------------------------------- |
| **GET**     | Retrieves data from the server. Commonly used for fetching resources.            |
| **POST**    | Submits new data to the server. Typically used for creating new records.         |
| **PUT**     | Replaces existing data with new data. Used for full updates.                     |
| **PATCH**   | Partially updates existing data. Ideal for minor changes to a resource.          |
| **DELETE**  | Removes data from the server.                                                    |
| **HEAD**    | Same as GET but returns only headers. Useful for checking existence or metadata. |
| **OPTIONS** | Describes available communication options. Often used for CORS preflight checks. |
| **ANY**     | Matches any HTTP method for the specified route.                                 |

### Basic Routing
The most basic Doppar routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files:
```php
<?php

use Phaseolies\Support\Facades\Route;

Route::get('/', fn() => "Welcome to Doppar");
```

### The Default Route Files
All Doppar routes are defined in your route files, located within the `routes` directory. These files are automatically loaded by the framework. The `routes/web.php` file is dedicated to defining routes for your web interface and is assigned the web middleware group, which enables essential features like session management and CSRF protection.

For most Doppar applications, you will start by defining routes inside the `routes/web.php` file. Any route declared in this file can be accessed through its corresponding URL in the browser. For example, the following route can be accessed by visiting `http://example.com/user` in your browser:

```php
use App\Http\Controllers\UserController;

Route::get('user', [UserController::class, 'index']);
```

### API Routes
Doppar provides separete api routes file localted in `routes/api.php`.

You can use **Doppar flarion**, which provides a robust, yet simple API token authentication system which can be used to authenticate third-party API consumers, or mobile applications.
```php
use Phaseolies\Support\Facades\Route;
use Phaseolies\Http\Request;

Route::get('user', function (Request $request) {
    return $request->user();
})->middleware('throttle:60,1', 'auth-api');
```

### Dependency Injection
You may type-hint any dependencies required by your route directly within the route’s callback signature. Doppar’s service container will automatically resolve and inject the appropriate instances for you. For example, you can type-hint the `Phaseolies\Http\Request` class to have the current HTTP request automatically injected into your route callback:
```php
use Phaseolies\Http\Request;
use App\Services\PaymentServiceInterface as Payment;

Route::post('payment', function (Request $request, Payment $payment) {
    //
});
```

### CSRF Protection
Keep in mind that any HTML forms targeting routes using the `POST`, `PUT`, `PATCH`, or `DELETE` methods—defined in the web.php routes file—must include a CSRF token field. Without this token, Doppar will reject the request for security reasons. To learn more, refer to the CSRF protection documentation.
```html
<form method="POST" action="{{ route('profile') }}">
    @csrf
    ...
</form>
```

### Working with PUT, PATCH, DELETE Route
In Doppar, when you need to handle `PUT`, `PATCH`, or `DELETE` requests (typically for updating or deleting data), you must follow a few conventions to make it work properly with HTML forms.

### Using HTTP Verb Spoofing in Forms
Since HTML forms only support GET and POST methods directly, Doppar provides Blade directives to spoof other HTTP methods like PUT, PATCH, and DELETE.

Here’s how you do it in your form:
```html
<form method="POST" action="/update-profile">
    @csrf
    @method('PUT')    {{-- For PUT Request --}}
    {{-- @method('PATCH')  For PATCH Request --}}
    {{-- @method('DELETE') For DELETE Request --}}
    <button type="submit">Submit</button>
</form>
```
> Note: You must include @csrf for CSRF protection, and `@method('PUT')`, `@method('PATCH')`, or `@method('DELETE')` to spoof the request type.

### Defining Routes in web.php
Once your form is set up, define the corresponding route in your routes file `(routes/web.php)` like this:
```php
use App\Http\Controllers\ProfileController;

// PUT Route
Route::put('/update-profile', [ProfileController::class, 'updateProfile']);

// PATCH Route
Route::patch('/update-profile', [ProfileController::class, 'updateProfile']);

// DELETE Route
Route::delete('/user/{id}', [ProfileController::class, 'delete']);
```

- - -
### Route Parameters
Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:
```php
Route::get('user/{id}', function (string $id) {
    return 'User '.$id;
});
```

You may define as many route parameters as required by your route:
```php
Route::get('posts/{post}/comments/{comment}', function (string $postId, string $commentId) {
    // ...
});
```
Route parameters in Doppar are defined by wrapping the parameter name in curly braces `{}` and should consist of alphabetic characters. You may also use underscores `(_)` in the parameter names. These parameters are automatically passed into your route callbacks or controller methods based on their position in the route `—` the actual variable names in the callback or method signature do not need to match the parameter names in the route.

### Optional Route Parameter
Doppar accept optional parameter and for this, you have nothing to do.
> ⚠️ Warning: By default, the Doppar controller callback takes parameters as optional. So, if you pass parameters in your route but do not accept them in the controller, it will not throw an error.
#### Example
```php
Route::get('user/{id}/{username}', [ProfileController::class, 'index']);
```
if you visit `http://example.com/user/1/mahedi-hasan`
```php
<?php

namespace App\Http\Controllers;

class ProfileController extends Controller
{
    public function index()
    {
        return "welcome"; // Still works and you will get the response
    }
}
```

### Named Routes
Doppar support convenient naming route structure. Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the name method onto the route definition:
```php

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\UserController;

Route::get('user/{id}/{name}', [UserController::class, 'profile'])->name('profile');
```

Now use this naming route any where using `route()` global method.
```html
 <form action="{{ route('profile', ['id' => 2, 'name' => 'mahedy']) }}" method="post">
    @csrf
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

If there is single param in your route, just use
```php
Route::get('user/{id}', [UserController::class, 'profile'])->name('profile');
```
Now call the route
```php
{{ route('profile', $user->id) }}
```

> ⚠️ Route names should always be unique.

### Generating URLs to Named Routes
Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via Doppar's route and redirect helper functions:
```php
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

### Route Group
Route::group is used to group multiple routes under a shared configuration like URL prefix. This helps in organizing routes cleanly and applying common logic to them.
```php
Route::group([
    'prefix' => 'your-prefix'
], function () {
    // Routes go here
});
```
#### Example
Look at the below example, we are using prefix as a group route.
```php
Route::group(['prefix' => 'login'], function () {
    Route::get('/action', function () {
        // Matches The "/login/action" URL
    });
});
```
`'prefix' => 'login'` This means all routes inside this group will be prefixed with `/login`.

### Route Middleware
Middleware in Doppar provides a convenient mechanism for filtering or modifying HTTP requests as they enter your application. Route middleware is typically used to perform tasks such as authentication, logging, CORS handling, input sanitization, and more — before the request reaches the controller or route logic.

You can assign middleware to routes in two primary ways:
#### Assigning Middleware to Individual Routes
You can attach middleware directly to a specific route using the middleware method:

```php
Route::get('dashboard', function () {
    // Only accessible to authenticated users
})->middleware('auth');
```

### Route Caching
Doppar supports route caching for improved performance in production environments. Route caching is controlled by the `APP_ROUTE_CACHE` environment variable: Set to true to enable route caching (recommended for production). Set to false to disable (default for development) Route cache files are stored in: `storage/framework/cache/`.
#### Cache Routes
Generate a route cache file for faster route registration:
```bash
php pool route:cache
```

### Clear Route Cache
```bash
php pool route:clear // Use this when making route changes in production or if experiencing route-related issues.
```




