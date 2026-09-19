---
title: Flarion
description: Doppar flarion page
meta:
  - name: keywords
    content: flarion
---

## Flarion

### Introduction
Flarion provides a lightweight, stateless API authentication system for Doppar applications. It allows users to generate and manage multiple Personal Access Tokens (PATs) that can be used to authenticate requests from mobile applications, third-party clients, CLI tools, and other services that communicate with your API.

Each token is securely associated with a user and can be assigned specific abilities, allowing you to control exactly which operations the token is permitted to perform. Tokens can also have configurable expiration times, making it possible to implement fine-grained and time-limited access without relying on traditional cookies or session-based authentication.

Flarion is designed with security in mind throughout the token lifecycle. Tokens are generated using cryptographically secure random data, while an HMAC-SHA256 lookup hash is used for efficient token verification without storing the raw token in the database. The plain-text token is returned to the client only when it is created, allowing the client to store and use it for subsequent authenticated requests through the `Authorization: Bearer` header.

The authentication system also provides the tools needed to protect API routes, verify token abilities, revoke tokens, and manage token expiration. Its integration with Doppar's middleware and routing system makes it straightforward to apply authentication at the route, method, or controller level while keeping API authentication stateless and flexible.

## Security Analysis
When a token is issued, it’s never stored in plain text — only a secure lookup hash is saved. Incoming requests are verified via `HMAC` hashing algorithm and validated against defined abilities. Tokens can automatically expire, and each token’s permissions are strictly limited by its assigned scopes.

This approach offers simple, fast, and secure user authentication for APIs, CLI tools, or third-party integrations.

Doppar Flarion’s authentication system achieves enterprise-grade security, providing robust protection, auditable usage, and fine-grained access control for every token.
| Area                      | Status          | Description                                                                    |
| ------------------------- | --------------- | ------------------------------------------------------------------------------ |
| **Token generation**      | ✅ Secure        | Uses `random_bytes()` for cryptographically strong random data                 |
| **Token lookup**          | ✅ Secure        | Uses `HMAC-SHA256` with app key; prevents rainbow-table or brute-force lookups |
| **Token storage**         | ✅ Secure        | The actual token is never stored directly.                               |
| **Expiration**            | ✅ Configurable  | Tokens can automatically expire based on configuration                         |
| **Abilities**             | ✅ Scoped access | Fine-grained control using abilities array                                     |

## Installation
You may install Doppar Flarion via the `composer require` command:
```bash
composer require doppar/flarion
```

## Register Launcher
Next, register the Flarion launcher so that Doppar can initialize it properly. Open your `runtime/config/app.php` file and add the `FlarionLauncher` to the launchers array:
```php
'launchers' => [
    // Other launchers...
    \Doppar\Flarion\FlarionLauncher::class,
],
```
This step ensures that Doppar knows about Flarion and can load its functionality when the application boots.

## Publish Configuration
Now we need to publish the configuration files by running this pool command.
```bash
php pool vendor:publish --launcher="Doppar\Flarion\FlarionLauncher"
```

Now run migrate command to migrate `personal_access_token` table
```bash
php pool migrate
```

## API Token Authentication
Flarion allows you to issue API tokens / personal access tokens that may be used to authenticate API requests to your application. When making requests using API tokens, the token should be included in the Authorization header as a `Bearer token`.

To begin issuing tokens for users, your User model should use the `Doppar\Flarion\Tokenable` trait:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Doppar\Flarion\Tokenable;

class User extends Model
{
    use Tokenable;
}
```
To issue a token, you may use the createToken method. The `createToken` method returns a `Doppar\Flarion\NewAccessToken` instance.
```php
use Phaseolies\Support\Router\Attributes\Route;

class LoginController extends Controller
{
    #[Route(uri: 'create/token', name: 'login.api', methods: ['POST'])]
    public function createToken()
    {
        $token = $user->createToken('myToken');

        return ['token' => $token->plainTextToken];
    }
}
```

You may access all of the user's tokens using the tokens Entity relationship provided by the `Doppar\Flarion\Tokenable` trait:
```php
foreach ($user->tokens as $token) {
    // ...
}
```

## Token Abilities
Flarion allows you to assign "abilities" to tokens. Abilities serve a similar purpose as OAuth's "scopes". You may pass an array of string `abilities` as the third argument to the `createToken` method:
```php
$user->createToken('token-name', null, ['post-update'])->plainTextToken;
```
When handling an incoming request authenticated by Flarion, you may determine if the token has a given ability using the middleware and tokenCan method:
```php
#[Route(uri: 'token_scope', middleware: ['auth-api:post-update'])]
public function update()
{
    if ($request->user()->tokenCan('post-update')) {
        // ...
    }
}
```
Or if you are working with `Facades` based routing, then you can do it as follows:
```php
Route::post('token_scope', [PostController::class, 'update'])
    ->middleware('auth-api:post-update');
```

Or you can did the same job like
```php
if ($request->user()->tokenCan('post-update')) {
    // ...
}
```

## Check Multiple Abilities
You can also check for multiple abilities by joining them with an ampersand (&). The token must have all listed abilities to pass the check:
```php
Route::post('token_scope', [PostController::class, 'update'])
    ->middleware('auth-api:post-update&post-another');
```
Each ability must exactly match one of the abilities assigned to the token. If any of the specified abilities are missing from the token, the request will be rejected with a `unauthorized` response.

## Protecting Routes
To ensure that all incoming API requests are authenticated, you should apply the Flarion authentication guard to any protected routes in your application — typically inside your `runtime/routes/api.php` file.

Flarion handles stateless authentication using API tokens, so every request must include a valid token in the Authorization header. This makes it perfect for mobile apps, external clients, or any token-driven access.

First register the flarion middleware inside your `App\Http\Kernel.php` file inside `api` array.
```php
public array $routeMiddleware = [
    'api' => [
        'auth-api' => \Doppar\Flarion\Http\Middleware\AuthenticateApi::class,
    ]
];
```

Now you can use `auth-api` middleware to protect your route.
```php
use Phaseolies\Http\Request;

#[Route(uri: 'store/product', middleware: ['auth-api'])]
public function store(Request $request)
{
    // Protected route
}
```

You can directly add `auth-api` middleware to your route in the following way. This approach provides a cleaner and more declarative way to assign middleware.

Example of method-level middleware
```php
<?php

namespace App\Http\Controllers\API;

use Phaseolies\Middleware\Attributes\Middleware;
use Phaseolies\Http\Request;
use Doppar\Flarion\Http\Middleware\AuthenticateApi;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    #[Middleware(AuthenticateApi::class)]
    public function getUser(Request $request)
    {

    }
}
```

Doppar supports attribute-based routing, allowing you to define routes directly above your controller methods for cleaner, more expressive code.
```php
use Phaseolies\Support\Router\Attributes\Route;

#[Route(uri: 'api/user', middleware:['auth-api'])]
public function getUser(Request $request)
{
   return $request->user();
}
```
This approach eliminates the need for traditional route files and keeps your route definitions close to their logic, improving readability and maintainability by using one line of code.

You can also apply middleware globally to all methods within a controller by placing the `#[Middleware(...)]` attribute above the class declaration:

Example of class-level middleware
```php
<?php

namespace App\Http\Controllers\API;

use Phaseolies\Middleware\Attributes\Middleware;
use Phaseolies\Http\Request;
use Doppar\Flarion\Http\Middleware\AuthenticateApi;
use App\Http\Controllers\Controller;

#[Middleware(AuthenticateApi::class)]
class UserController extends Controller
{
   //
}
```

## Revoking Tokens
You may "revoke" tokens by deleting them from your database using the tokens relationship that is provided by the `Doppar\Flarion\Tokenable` trait:
```php
// Revoke all tokens
$request->user()->tokens()->delete();

// Revoke the token that was used to authenticate the current request
$request->user()->currentAccessToken()->delete();

// Revoke a specific token
$request->user()->tokens()->where('id', $tokenId)->delete();
```

## Token Expiration
By default, Flarion tokens never expire and may only be invalidated by revoking the token. However, if you would like to configure an expiration time for your application's API tokens, you may do so via the expiration configuration option defined in your application's `runtime/config/flarion` configuration file. This configuration option defines the number of minutes until an issued token will be considered expired:
```php
'expiration' => 5,
```
This means all tokens will expire 5 minutes after creation unless otherwise specified.

If you would like to specify the expiration time of each token independently, you may do so by providing the expiration time as the second argument to the `createToken` method. You can override the global setting when generating a token:

```php
$user->createToken('token-name',  now()->addMinutes(5))->plainTextToken;
```
In this example, the token will expire 5 minutes from the current time, regardless of the global config.