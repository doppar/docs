---
title: Routing
description: Doppar Routing page
meta:
  - name: keywords
    content: Routing
---

## Routing
### Introduction
Doppar’s routing system, available through the `Phaseolies\Support\Facades\Route` namespace and using `Phaseolies\Utilities\Attributes\Route` attributes, provides a clean, expressive way to define your application’s URL structure and map it to the appropriate controller actions or closures.

Doppar’s routing engine offers features such as route prefix grouping, named routes, throttling, middleware assignment, and RESTful resource routing. Whether you’re building a lightweight API or a complex web application, Doppar’s routing system ensures your code remains consistent, maintainable, and scalable.

## Supported HTTP Methods
Doppar supports the following HTTP methods for defining routes:

| **Method**  | **Description**                                                                  |
| ----------- | -------------------------------------------------------------------------------- |
| **GET**     | Retrieves data from the server. Commonly used for fetching resources.            |
| **POST**    | Submits new data to the server. Typically used for creating new records.         |
| **PUT**     | Replaces existing data with new data. Used for full updates.                     |
| **PATCH**   | Partially updates existing data. Ideal for minor changes to a resource.          |
| **DELETE**  | Removes data from the server.                                                    |
| **HEAD**    | Same as GET but returns only headers. Useful for checking existence or metadata. |
| **OPTIONS** | Describes available communication options. Often used for CORS preflight checks. |                |

## Route Caching
Doppar supports route caching for improved performance in production environments. Route caching is controlled by the `APP_ROUTE_CACHE` environment variable: Set to true to enable route caching (recommended for production). Set to false to disable (default for development) Route cache files are stored in: `storage/framework/cache/`.
### Cache Routes
The `route:cache` command is used to compile your application's routes into a single, cached file. This improves performance by significantly speeding up route registration, especially in production environments.

When the route cache is enabled, the framework loads the routes from the generated cache file instead of parsing all route definitions on each request. This reduces overhead and improves response times.

To create the route cache, run:
```bash
php pool route:cache
```
It's important to ensure that all of your routes are working correctly before caching them. If there are any issues or syntax errors in your route definitions, the command will fail and output the relevant error.

## Clear Route Cache
The `route:clear` command is used to remove the route cache file. This is useful when you have made changes to your application's routes and want to ensure the framework uses the updated definitions.

In a production environment, route caching improves performance by loading a precompiled route list. However, if the route cache becomes outdated or corrupted, it can lead to unexpected behavior. Running this command will delete the cached route file, forcing the application to load routes directly from the source files on the next request.

You should run this command after deploying any updates that modify your routes:
```bash
php pool route:clear
```
Use this when making route changes in production or if experiencing route-related issues.

## Attribute Based Routing
Doppar also supports attribute-based routing, allowing you to define routes directly above your controller methods using PHP 8 attributes. This approach offers a cleaner, more localized way to declare routes, keeping route definitions close to the logic they handle. It reduces the need to manage separate route files for simple or self-contained controllers, improving code readability and maintainability

Each attribute-based route can specify its path, name, HTTP methods, and other configurations, just like traditional route definitions. The Doppar routing engine automatically detects and registers these routes when your application boots.

When defining a route using the Route attribute, only the first parameter — the route path — is mandatory.

The path specifies the `URI` that the route should respond to, while all other parameters such as `name`, `methods`, `middleware`, are optional and can be included as needed. This makes it simple to define quick routes with minimal configuration when defaults are sufficient.

Example:
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;

class UserController extends Controller
{
    #[Route(uri: 'user')]
    public function index()
    {
        // Handles GET /user
    }
}
```

Now you can view the details of your registered routes by running the following pool command:
```bash
php pool route:list
```

In this example, only the route uri `('user')` is provided. Doppar will automatically assume default values for other parameters — such as using the `GET` method and no assigned name or middleware.

See the below example with handling incoming HTTP `methods`
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;

class PostController extends Controller
{
    #[Route(uri: 'posts', methods: ['GET'])]
    public function index()
    {
        //
    }
}
```

In this example, the index method is mapped to the `/posts` endpoint, responding to `GET` requests.

In addition to `GET` requests, attribute-based routing in Doppar supports all common HTTP methods such as `POST`, `PUT`, `PATCH`, `DELETE` etc. This allows developers to define full RESTful endpoints directly within controllers, without relying solely on external route files.

See the below example with `methods` and `name`
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;

class PostController extends Controller
{
    #[Route(uri: 'posts', methods: ['GET'], name: 'post.index')]
    public function index()
    {
        //
    }
}
```

In this example, the index method is mapped to the `/posts` endpoint, responding to `GET` requests pointing to route name `post.index`.

Each route can declare one or multiple HTTP methods using the methods parameter, giving fine-grained control over how requests are handled.
```php
#[Route(uri: 'post/store', methods: ['POST', 'PATCH'])]
public function store(Request $request)
{
    // Handle creating or updating a post
}
```

In this example, the store method will respond to both `POST` and `PATCH` requests sent to the `/post/store` endpoint. This flexibility makes it easy to manage different request types for the same route, supporting both resource creation and partial updates within a single controller action.

### Route Prefix with `#[Mapper]`
Doppar supports controller-level route prefixes and middleware via the `#[Mapper]` attribute. This feature allows developers to declare common URI segments and middleware for all routes within a controller, reducing repetition and improving readability.

Basic usage example:
```php
use Phaseolies\Utilities\Attributes\Mapper;

#[Mapper(prefix: 'user', middleware: ['auth'])]
class UserController extends Controller
{
    #[Route(uri: '/{id}', middleware: ['admin'])]
    public function show($id)
    {
        // Endpoint: http://example.com/user/1
        // The auth middleware is applied to all routes.
        // Individual routes can still define -
        // Additional middleware (e.g., admin for the show method).
        return $id;
    }
}
```

### Attribute Routing with Middleware
Doppar’s attribute-based routing also supports middleware assignment directly within the route definition. This allows you to apply one or more middleware layers to a specific controller method without configuring them separately in a route file.

By specifying the middleware parameter inside the Route attribute, you can easily protect routes, apply request filters, or run any preprocessing logic before the controller action executes. Middleware are executed in the order they are listed, ensuring full control over the request lifecycle.
```php
#[Route(
    uri: '/post/store',
    methods: ['POST', 'PATCH'],
    name: 'post.store',
    middleware: ['auth', 'admin']
)]
public function store(Request $request): Response
{
    // Handle creating or updating a post
}
```
In this example, the `/post/store` route is protected by the `auth` and `admin` middleware. The request must first pass both middleware checks before the controller’s store method is executed, ensuring secure and validated access to the route.

> Note:
When using the Route attribute, you must pass registered middleware names, not class references.

For example, passing `middleware: ['auth']` will work correctly if auth is a middleware alias registered in your application’s middleware configuration.

However, passing `middleware: [Authenticate::class]` will not work, as attribute-based routing expects middleware names rather than class references.

If you need to use class-based middleware, apply them through the dedicated `#[Middleware(...)]` attribute instead.

### Passing Parameters
When using attribute-based routing in Doppar, you can enhance your routes by passing parameters directly to middleware. This feature allows attributes and middleware to work seamlessly together, giving you expressive, method-level control over your route behavior.

By defining middleware and their parameters right within the route attribute, your controller logic stays clean, self-contained, and easy to understand — with all route configurations centralized in one place.

Example:
```php
#[Route(
    uri: '/user',
    name: 'user.index',
    methods: ['GET', 'POST'],
    middleware: ['auth', 'response.break:admin']
)]
public function __invoke() 
{
    //
}
```
In this example, the `response.break` middleware receives the parameter `admin`, demonstrating how you can pass dynamic configuration directly through attributes. 

> 💡 Learn more about passing parameters to middleware [middleware-parameters](middleware#middleware-parameters)

### Routing with Rate Limit
Though rate limiting can be implemented using middleware, it can now be defined directly within the route attributes.

See the example of rate limiting using middleware
```php
#[Route(
      uri: 'home',
      methods: ['GET'],
      middleware: ['throttle:10,1']
)]
public function home(): Response
{
      //
}
```
In this example, the throttle middleware restricts access to 10 requests per minute.

Now we can also implement this by passing `rateLimit` and `rateLimitDecay` naming params like this way
```php
#[Route(
    uri: 'home',
    methods: ['GET'],
    rateLimit: 3,       // Allow up to 3 requests
    rateLimitDecay: 1,  // Within 1 minute
)]
public function home(): Response
{
    //
}
```
This approach eliminates the need to manually specify middleware for simple throttling needs.

## Domain-Restricted Routing
Doppar's routing system supports domain-based route matching, allowing you to restrict specific routes to particular hostnames or subdomains. This is particularly useful for multi-tenant applications, API versioning across subdomains, or separating admin panels from public-facing websites.

Domain routing works seamlessly with both file-based and attribute-based route definitions, and supports exact domain matching, port-qualified hosts, wildcard subdomains with automatic parameter injection, and universal wildcards.

### Basic Domain Restriction
You can restrict a route to respond only when the incoming request matches a specific domain using the `domain()` method in file-based routes or the domain parameter in attribute-based routes.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;

class ApiController extends Controller
{
    #[Route(
        uri: 'status',
        methods: ['GET'],
        domain: 'api.example.com',
        name: 'api.status'
    )]
    public function status(): array
    {
        return ['status' => 'ok', 'version' => '1.0'];
    }
}
```
In this examples, the `status` route will only be accessible when the request is made to `api.example.com`. Requests to `example.com` or any other host will return a 404 error.

### Wildcard Subdomain Routing
One of the most powerful features of domain routing is the ability to capture subdomain segments as route parameters. This enables true multi-tenant applications where each tenant gets their own subdomain.

Use curly braces `{parameter}` in the domain pattern to capture the subdomain segment. The captured value is automatically injected into your controller method just like a regular route parameter.
```php
#[Route(
    uri: '/dashboard',
    methods: ['GET'],
    domain: '{tenant}.app.com',
    name: 'tenant.dashboard'
)]
public function dashboard(string $tenant): array
{
    // $tenant is automatically injected with the subdomain value
    // e.g., 'acme' from acme.app.com
    // $request->tenant also available

    $workspace = Workspace::where('slug', $tenant)->first();

    return [
        'workspace' => $workspace->name,
        'tenant'    => $tenant,
    ];
}
```
Notes:
- The wildcard parameter name `(e.g., {tenant})` becomes a route parameter accessible in your controller.
- You can combine wildcard subdomains with regular route parameters
- The bare domain `(e.g., app.com without a subdomain)` will not match a wildcard pattern like `{tenant}.app.com`

### Universal Wildcard Domain
Use the `*` wildcard to create a route that matches any host. This is useful for catch-all routes in grouped configurations or when you want to explicitly allow a route to work across all domains.
```php
#[Route(
    uri: '/health',
    methods: ['GET'],
    domain: '*'
)]
public function check(): array
{
    return ['status' => 'healthy'];
}
```
This route will respond to requests from any hostname: `example.com`, `api.example.com`, `localhost`, etc.

### Same Path, Different Domains
You can define multiple routes with the same URI path but different domain restrictions. Doppar's router will dispatch the request to the correct handler based on the incoming host header.
```php
class MarketingController extends Controller
{
    #[Route(uri: '/', domain: 'example.com', name: 'marketing.home')]
    public function home()
    {
        return view('welcome');
    }
}

class ApiController extends Controller
{
    #[Route(uri: '/', domain: 'api.example.com', name: 'api.health')]
    public function health(): array
    {
        return ['status' => 'ok'];
    }
}

class AdminController extends Controller
{
    #[Route(
        uri: '/',
        domain: 'admin.example.com',
        name: 'admin.dashboard',
        middleware: ['auth', 'admin']
    )]
    public function dashboard()
    {
        return view('admin.dashboard');
    }
}
```

Each controller's root route `(/)` is completely isolated by domain — no conflicts, no cross-contamination.

## Route Model Binding
Doppar introduces a powerful and expressive way to automatically resolve route parameters into model instances using PHP attributes. This feature allows you to specify how a model should be retrieved — by its ID, by a specific column, or with exception-handling behavior — directly in your controller method signature.

Below are examples demonstrating various use cases for the `#[Model]` attribute.
```php
use Phaseolies\Utilities\Attributes\Model;

#[Route('/profile/{user}', methods: ['GET'])]
public function show(#[Model] ?User $user) 
{
    // The $user instance is automatically fetched using the 'id' column.
    // If no matching user is found, null will be assigned by default.
    return $user;
}
```

In this example, the `{user}` route parameter is automatically resolved to a User model instance by matching the `id` column. If the user is not found, the `$user` variable will be null (no exception is thrown)

### Explicit Model Binding
The `#[Model('email')]` attribute clearly expresses that binding should occur based on the `email` column rather than the default `id`.

```php
#[Route('/profile/{user}', methods: ['GET'])]
public function show(#[Model('email')] ?User $user) 
{
    // Fetched using the 'email' column.
    return $user;
}
```

In this example, the `{user}` route parameter is automatically resolved to a User model instance by matching the `email` column. If the user is not found, the $user variable will be null (no exception is thrown)

### Exception Handling
This example demonstrates how Doppar can automatically enforce strict model resolution for route parameters. By setting `exception: true` in the `#[Model]` attribute, the framework will attempt to fetch the User model by the specified column (email in this case).

If a matching user is not found, a `NotFoundHttpException` is thrown immediately, preventing null values from being passed to the controller. This ensures that your route always receives a valid model instance or fails fast, making your controller logic simpler and safer.
```php
#[Route('/profile/{user}', methods: ['GET'])]
public function show(
    #[Model(column: 'email', exception: true)] ?User $user
) {
    // If no user is found,
    // NotFoundHttpException will be thrown automatically.
    return $user;
}
```
This ensures strict route validation and avoids passing null models to your logic.

## Globally Set Route Key Name
In Doppar, you can define a global route key name for your model to simplify and standardize how it’s resolved during route model binding.

This eliminates the need to repeatedly specify the binding column in your route definitions.

### Defining the Global Route Key
Inside your model, override the `getRouteKeyName()` method to specify which column should be used for route model binding.
```php
/**
 * Get the route key name for model binding.
 *
 * @return string
 */
#[\Override]
public function getRouteKeyName(): string
{
    return 'email';
}
```
With this method in place, Doppar will automatically use the `email` column whenever this model is bound in a route — no additional configuration is required.

### Overriding the Global Column
If you need to bind using a different column for a specific route, you can override the global route key by explicitly specifying the column in the `#[Model]` attribute:
```php
#[Route('/profile/{user}', methods: ['GET'])]
public function show(#[Model('username')] ?User $user)
{
    return $user;
}
```
Here, Doppar will temporarily ignore the global `email` key and use the `username` column instead.

## Routing with Route Facades
The most basic Doppar routes accept a URI and a closure, providing a very simple and expressive method of defining routes and behavior without complicated routing configuration files:
```php
<?php

use Phaseolies\Support\Facades\Route;

Route::get('/', fn() => "Welcome to Doppar");
```

## The Default Route Files
All Doppar routes are defined in your route files, located within the `routes` directory. These files are automatically loaded by the framework. The `routes/web.php` file is dedicated to defining routes for your web interface and is assigned the web middleware group, which enables essential features like session management and CSRF protection.

For most Doppar applications, you will start by defining routes inside the `routes/web.php` file. Any route declared in this file can be accessed through its corresponding URL in the browser. For example, the following route can be accessed by visiting `http://example.com/user` in your browser:

```php
use App\Http\Controllers\UserController;

Route::get('user', [UserController::class, 'index']);
```

## API Routes
Doppar provides separete api routes file localted in `routes/api.php`.

You can use **Doppar flarion**, which provides a robust, yet simple API token authentication system which can be used to authenticate third-party API consumers, or mobile applications.
```php
use Phaseolies\Support\Facades\Route;
use Phaseolies\Http\Request;

Route::get('user', function (Request $request) {
    return $request->user();
})->middleware('auth-api');

// Endpoint
// http://example.com/api/user
```

## Dependency Injection
You may type-hint any dependencies required by your route directly within the route’s callback signature. Doppar’s service container will automatically resolve and inject the appropriate instances for you. For example, you can type-hint the `Phaseolies\Http\Request` class to have the current HTTP request automatically injected into your route callback:
```php
use Phaseolies\Http\Request;

Route::post('payment', function (Request $request) {
    //
});
```

## CSRF Protection
Keep in mind that any HTML forms targeting routes using the `POST`, `PUT`, `PATCH`, or `DELETE` methods—defined in the web.php routes file—must include a CSRF token field. Without this token, Doppar will reject the request for security reasons. To learn more, refer to the CSRF protection documentation.
```html
<form method="POST" action="[[ route('profile') ]]">
    #csrf
    ...
</form>
```

## Handling Modified Request Route
In Doppar, when you need to handle `PUT`, `PATCH`, or `DELETE` requests (typically for updating or deleting data), you must follow a few conventions to make it work properly with HTML forms.

The HTTP methods PUT, PATCH, and DELETE define the intended action on a resource in RESTful APIs or web applications. Here's what each one does when a request is made:

| Method | Action                     | Body Required | Idempotent | Common Use Case          |
| ------ | -------------------------- | ------------- | ---------- | ------------------------ |
| PUT    | Full replace of resource   | ✅ Yes         | ✅ Yes      | Update full user record  |
| PATCH  | Partial update of resource | ✅ Yes         | ✅ Usually  | Change a single field    |
| DELETE | Remove resource            | ❌ Usually no  | ✅ Yes      | Delete an item or record |

## HTTP Verb Spoofing in Forms
Since HTML forms only support GET and POST methods directly, Doppar provides Odo directives to spoof other HTTP methods like PUT, PATCH, and DELETE.

Here’s how you do it in your form:
```html
<form method="POST" action="[[ route('update-profile') ]]">
    #csrf
    @method('PUT')    [[-- For PUT Request --]]
    [[-- @method('PATCH')  For PATCH Request --]]
    [[-- @method('DELETE') For DELETE Request --]]
    <button type="submit">Submit</button>
</form>
```

::: warning
Always include `#csrf` to protect against CSRF attacks. The `@method` directive tells Doppar to treat the request as the specified HTTP verb.
:::

## Any Route
The `Route::any()` method is used to register a route that responds to any HTTP method (GET, POST, PUT, DELETE, etc.). This is particularly useful for routes where the HTTP method doesn’t matter, such as catch-all pages, testing endpoints, or webhook receivers.

```php
Route::any('welcome*', fn() => 'welcome');
```

The `*` acts as a wildcard, so this route matches any URL starting with welcome (e.g., `/welcome`, `/welcome-home`, /`welcome123`, `welcome/hello`. This handle multiple methods without defining them separately.

## Defining Modified Request Routes
Once your form is set up to spoof `PUT`, `PATCH`, or `DELETE` methods, you need to define the corresponding routes in your `routes/web.php` file. These routes will map the specific HTTP methods to the appropriate controller actions.

In Doppar, this is typically done using the `Route::put`, `Route::patch`, and `Route::delete` methods provided by the routing system.
```php
use App\Http\Controllers\ProfileController;

// PUT Route
Route::put('update-profile', [ProfileController::class, 'update']);

// PATCH Route
Route::patch('update-profile', [ProfileController::class, 'update']);

// DELETE Route
Route::delete('user/{id}', [ProfileController::class, 'delete']);
```

## Route Parameters
Sometimes you will need to capture segments of the URI within your route. For example, you may need to capture a user's ID from the URL. You may do so by defining route parameters:
```php
Route::get('user/{id}', function (string $id) {
    return 'User '.$id;
});
```

You may define as many route parameters as required by your route:
```php
Route::get('posts/{post}/comments/{comment}', function (
    string $postId, string $commentId
) {
        echo $postId;
        echo $commentId;
});
```
Route parameters in Doppar are defined by wrapping the parameter name in curly braces `{}` and should consist of alphabetic characters. You may also use underscores `(_)` in the parameter names. These parameters are automatically passed into your route callbacks or controller methods based on their position in the route `—` the actual variable names in the callback or method signature do not need to match the parameter names in the route.

## Named Routes
Doppar support convenient naming route structure. Named routes allow the convenient generation of URLs or redirects for specific routes. You may specify a name for a route by chaining the name method onto the route definition:
```php

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\UserController;

Route::get('user/{id}/{name}', [UserController::class, 'profile'])
    ->name('profile');
```

Now use this naming route any where using `route()` global method.
```html
 <form action="[[ route('profile', ['id' => 2, 'name' => 'abc']) ]]"
    method="post">
    #csrf
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
```

If there is single param in your route, just use
```php
Route::get('user/{id}', [UserController::class, 'profile'])
    ->name('profile');
```
Now call the route
```php
[[ route('profile', $user->id) ]]
```

> Route names should always be unique.

## Generating URLs to Named Routes
Once you have assigned a name to a given route, you may use the route's name when generating URLs or redirects via Doppar's route and redirect helper functions:
```php
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

## File-Based Domain Restriction
You can restrict a route to respond only when the incoming request matches a specific domain using the `domain()` method in file-based routes 
```php
use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\ApiController;

Route::get('/status', [ApiController::class, 'status'])
    ->domain('api.example.com')
    ->name('api.status');
```
Only responds to requests on `api.example.com`.

### Wildcard Subdomain Routing
One of the most powerful features of domain routing is the ability to capture subdomain segments as route parameters. This enables true multi-tenant applications where each tenant gets their own subdomain.

Use curly braces `{parameter}` in the domain pattern to capture the subdomain segment. The captured value is automatically injected into your controller method just like a regular route parameter.
```php
Route::get('/dashboard', [TenantController::class, 'dashboard'])
    ->domain('{tenant}.app.com')
    ->name('tenant.dashboard');
```

Matches any subdomain: `acme.app.com`, `globex.app.com`, etc.

Notes:
- The wildcard parameter name `(e.g., {tenant})` becomes a route parameter accessible in your controller
- You can combine wildcard subdomains with regular route parameters
- The bare domain `(e.g., app.com without a subdomain)` will not match a wildcard pattern like `{tenant}.app.com`

### Universal Wildcard Domain
Use the `*` wildcard to create a route that matches any host. This is useful for catch-all routes in grouped configurations or when you want to explicitly allow a route to work across all domains.
```php
Route::get('/health', [HealthController::class, 'check'])
    ->domain('*');
```
This route will respond to requests from any hostname: `example.com`, `api.example.com`, `localhost`, etc.

## Defining Bundle Routes
The bundle method allows you to register a complete set of CRUD routes for a controller.
You can customize which routes are created and the names assigned to them using the `only`, `except`, and `names` options.

Example
```php
use Phaseolies\Support\Router\Route;
use App\Http\Controllers\ProductController;

Route::bundle('products', ProductController::class);
```

Generated route for this `products` bundle
```markdown
| Method | URI                        | Action                     | Name              |
| ------ | -------------------------- | -------------------------- | ----------------- |
| GET    | `/products`                | `ProductController@index`  | `products.index`  |
| GET    | `/products/create`         | `ProductController@create` | `products.create` |
| POST   | `/products`                | `ProductController@store`  | `products.store`  |
| GET    | `/products/{id}/show`      | `ProductController@show`   | `products.show`   |
| GET    | `/products/{id}/edit`      | `ProductController@edit`   | `products.edit`   |
| PUT    | `/products/{id}/update`    | `ProductController@update` | `products.update` |
| DELETE | `/products/{id}/delete`    | `ProductController@delete` | `products.delete` |
```


By default, bundle routes use the `primaryKey` field as the route key for model binding along with fallback to `id`.  

If your model does not have an `id` column, or you want to bind routes using a different column (for example, `slug`), you can override the `getRouteKeyName()` method in your Entity model.

```php
/**
 * Get the route key name for model binding.
 *
 * @return string
 */
#[\Override]
public function getRouteKeyName(): string
{
    return 'slug';
}
```

Now, instead of resolving routes by default `primaryKey`, Doppar will automatically resolve models using the `slug` field.

## Bundle Model & Controller Naming Convention
`If you use custom route key binding`, then you must follow this model and controller naming convention. It is important to follow a consistent naming convention for models and controllers. This ensures that automatic model resolution and route binding work correctly.

#### Controller → Model Mapping
```markdown
| Controller                       | Model               |
|----------------------------------|---------------------|
| `ProductController`              | `Product`           |
| `ProductDetailsController`       | `ProductDetails`    |
```

> You must follow this convention before using bundle routes `If you use custom route key binding`. This ensures that Doppar can correctly infer the corresponding controller from the model and vice versa.

## Bundle Routes with Options
The bundle method registers a full set of CRUD routes for a given controller.
You can customize which routes are generated and what names they use by passing the `only`, `except`, `names` and `method` options.

Example
```php
use App\Http\Controllers\ProductController;
use Phaseolies\Support\Router\Route;

Route::bundle('products', ProductController::class, [
    'only' => ['index', 'show', 'update'],
    'names' => [
        'index' => 'products.all',
        'show' => 'products.view'
    ],
    'methods' => [
        'update' => 'POST'
    ]
]);
```

Explanation:
- The only option ensures that only the `index`, `show` and `update` routes are registered.
- The names option `overrides` the `default route names` with custom ones.
- Now the `update` endpoint uses `POST` rather than `PUT`

##### Generated Routes:
```markdown
| Method | URI                   | Action                    | Name            |
| ------ | --------------------- | ------------------------- | --------------- |
| GET    | `/products`           | `ProductController@index` | `products.all`  |
| GET    | `/products/{id}/show` | `ProductController@show`  | `products.view` |
```

## API Bundle Routes
The `apiBundle` method works like bundle, but it excludes the `create` and `edit` routes. This is ideal for API endpoints, where HTML form views are not needed, and you only want routes for data operations.

Example:
```php
use App\Http\Controllers\InviteController;
use Phaseolies\Support\Router\Route;

Route::apiBundle('posts', InviteController::class);
```

Example with Options:
```php
Route::apiBundle('posts', InviteController::class, [
    'only' => ['index', 'show']
]);
```

Generated Routes:

```markdown
| Method | URI                    | Action                   | Name         |
| ------ | ---------------------- | ------------------------ | ------------ |
| GET    | `/api/posts`           | `InviteController@index` | `posts.index`|
| GET    | `/api/posts/{id}/show` | `InviteController@show`  | `posts.show` |
```

## Reminder on API Usage
When you are building APIs that accept file uploads, this note is important:
> Reminder on API Usage
If you are uploading files using `form-data` rather than `x-www-form-urlencoded`, Doppar does not support file handling for `PUT` and `PATCH` requests. Always use `POST` request when your API endpoint accepts files. Using `PUT/PATCH` with `form-data` uploads may cause unexpected issues.

By default, Doppar’s `Route::apiBundle` will generate a `PUT` request for the update action.
Since `PUT` does not support file uploads for `form-data`, you can override it to `POST`:
```php
Route::apiBundle('file', FileController::class, [
    'methods' => [
        'update' => 'POST' // Now the update endpoint uses POST
    ]
]);
```

This ensures your update endpoint can handle file uploads without issues.

## Nested Bundle Routes
The `nestedBundle` method registers a full CRUD route set for a child resource that is nested under a parent resource.
It automatically includes the parent resource parameter in the URI, making it easy to work with relationships such as “`posts → comments`” or “`categories → products`”.

Example of Nested Bundle

```php
use App\Http\Controllers\CommentController;
use Phaseolies\Support\Router\Route;

Route::nestedBundle('posts', 'comments', CommentController::class);
```

Example Controller
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use App\Http\Controllers\Controller;

class CommentController extends Controller
{
    public function index($postId) {}
    public function create($postId) {}
    public function store(Request $request, $postId) {}
    public function show($postId, $id) {}
    public function edit($postId, $id) {}
    public function update(Request $request, $postId, $id) {}
    public function delete($postId, $id) {}
}
```

Generated Routes

```markdown
| Method | URI                                 | Action                     | Name                    |
| ------ | ----------------------------------- | -------------------------- | ----------------------- |
| GET    | `/posts/{post}/comments`            | `CommentController@index`  | `posts.comments.index`  |
| GET    | `/posts/{post}/comments/create`     | `CommentController@create` | `posts.comments.create` |
| POST   | `/posts/{post}/comments`            | `CommentController@store`  | `posts.comments.store`  |
| GET    | `/posts/{post}/comments/{id}/show`  | `CommentController@show`   | `posts.comments.show`   |
| GET    | `/posts/{post}/comments/{id}/edit`  | `CommentController@edit`   | `posts.comments.edit`   |
| PUT    | `/posts/{post}/comments/{id}/update`| `CommentController@update` | `posts.comments.update` |
| DELETE | `/posts/{post}/comments/{id}/delete`| `CommentController@delete` | `posts.comments.delete` |
```

You can also pass options in nested bundle

```php
Route::nestedBundle('posts', 'comments', CommentController::class, [
    'except' => ['create', 'edit']
]);
```

> `Note:` Bundle routes do not support method chaining like `->middleware('auth')`.
To apply middleware, use [attribute-based middleware](https://doppar.com/versions/3.x/middleware#attribute-based-middleware-loading) calling directly in your bundle controller.

## Route Group
`Route::group` is used to group multiple routes under a shared configuration like URL prefix. This helps in organizing routes cleanly and applying common logic to them.
```php
Route::group([
    'prefix' => 'your-prefix'
], function () {
    // Routes go here
});
```
### Example
Look at the below example, we are using prefix as a group route.
```php
Route::group(['prefix' => 'login'], function () {
    Route::get('/action', function () {
        // Matches The "/login/action" URL
    });
});
```
`'prefix' => 'login'` This means all routes inside this group will be prefixed with `/login`.

## Route Middleware
Middleware in Doppar provides a convenient mechanism for filtering or modifying HTTP requests as they enter your application. Route middleware is typically used to perform tasks such as authentication, logging, CORS handling, input sanitization, and more — before the request reaches the controller or route logic.

You can assign middleware to routes in two primary ways:
### Assigning Middleware to Individual Routes
You can attach middleware directly to a specific route using the middleware method. This allows you to apply custom logic, such as authentication or request throttling, to just that route.

```php
Route::get('dashboard', function () {
    // Only accessible to authenticated users
})->middleware('auth');
```
In this example, the auth middleware ensures that only authenticated users can access the `dashboard` route. If a user is not authenticated, they will be redirected or denied access based on your application's configuration.

You may also chain multiple middleware by passing them as an array:
```php
Route::post('settings', function () {
    // Protected by multiple middleware
})->middleware(['auth', 'verified']);
```
This approach gives you fine-grained control over access and behavior at the route level.

You can assign middleware directly to a specific route using the middleware method. This lets you apply route-level behavior such as authentication, authorization, or request handling as multiple string arguments like this way.
```php
Route::post('settings', function () {
    // Protected by multiple middleware
})->middleware('auth', 'verified');
```
Both approaches are fully supported in Doppar and function the same. Use whichever style best fits your project's conventions or coding preferences.


## Route Redirection
The `Route::redirect()` method allows you to define quick and clean route redirections in your application. Whether you're migrating old URLs, setting up aliases, or redirecting to external destinations, this method provides a declarative and consistent way to do so.

Syntax
```php
Route::redirect($from, $to, $status = 302);
```
- `$from` — The original URI pattern to match.
- `$to` — The target location: can be a URL, a named route, or an external link.
- `$status` — Optional HTTP status code (default: 302 Found). Use 301 for permanent redirects.

Redirecting to a New URL
```php
Route::redirect('old', 'new', 301);
```

Redirecting to a Named Route
```php
Route::redirect('legacy', 'user.profile');
```

Redirecting to an External URL
```php
Route::redirect('blog', 'https://doppar.com');
```

## Route Helper Method
The `Route` Facades provide convenient ways to interact with the current request and route information in your application. They allow you to easily retrieve route names, check if a route exists, determine the current route or middleware, and inspect the controller or action handling the request.

These helpers are designed to make it easier to write clean and expressive logic inside controllers, middleware, or services without manually parsing routes.

### Get All Route Names
The `getRouteNames()` method returns an array of all the named routes registered in your application.
This is useful when you want to inspect or debug available route names, or programmatically loop through them.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\Controller;

class ProductController extends Controller
{
    public function index()
    {
        return Route::getRouteNames();
    }
}
```

If you have routes defined like this:
```php
Route::get('/products', [ProductController::class, 'index'])->name('products.index');
Route::get('/products/{id}', [ProductController::class, 'show'])->name('products.show');
Route::post('/products', [ProductController::class, 'store'])->name('products.store');
```

Then calling `/products` (with the controller above) will return something like:
```php
{
    "products.index": "/products",
    "products.show": "/products/{id}",
    "products.store": "/products"
}
```

### Get Route Middlewares
The `getCurrentMiddlewareNames()` method returns an array of all middleware names that are applied to the current request’s route.
This is helpful when you want to check which middlewares are active for the request being processed.
```php
Route::getCurrentMiddlewareNames();
```
If the current route does not have any middleware applied, the method will return null.

### Checking Route Name Existance
The `has()` method checks if a named route exists in your application. It returns a boolean (true or false) depending on whether the given route name has been registered.

This is useful when you need to confirm the existence of a route before generating URLs or performing route-specific logic.
```php
if (Route::has('products.index')) {
    return "Route 'products.index' exists!";
}

return "Route 'products.index' does not exist.";
```

### Check Current Route Matching
The `is()` method checks if the current request’s route matches the given route name. It compares the request path against the named route definition.
```php
if (Route::is('products.index')) {
    return "You are on the products index page.";
}

return "This is not the products index page.";
```

### Get Current Route Name
The `currentRouteName()` method returns the name of the current route being executed. If no matching named route is found, it will return null.

This is especially useful when you need to check or display the current route name (e.g., for navigation highlighting or conditional logic).
```php
Route::currentRouteName();
```

### Get Current Route Action
The `currentRouteAction()` method returns the action associated with the current route. The action can be returned in one of the following forms:

- `String:` ControllerClass@method
- `Closure:` Closure
- `Array:` [ControllerClass, 'method'] (converted to string `ControllerClass@method`)
- `Null:` if no callback is found

This is useful for debugging, logging, or conditionally checking which controller or action is handling the current request.

Example
```php
Route::currentRouteAction();
```

You will get the response like this
```php
App\Http\Controllers\ProductController@index
```

### Check Current Route Uses Specific Controller
The `currentRouteUsesController(`) method checks if the current route is handled by a specific controller class. It returns true if the current route’s action belongs to the given controller, and false otherwise.

Check if the current route uses a specific controller
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\Controller;

class ProductController extends Controller
{
    public function index()
    {
        if (Route::currentRouteUsesController(ProductController::class)) {
            return "This route is handled by ProductController.";
        }

        return "This route is handled by another controller.";
    }
}
```

This is useful for conditional logic that depends on the controller handling the current request, such as middleware checks, logging, or dynamic UI adjustments.
