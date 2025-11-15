---
title: Flarion
description: Doppar flarion page
meta:
  - name: keywords
    content: flarion
---

## Flarion - API Authentication
### Introduction
Flarion provides a lightweight API authentication system for the Doppar framework.

Flarion allows each user of Doppar application to generate and manage multiple API tokens. These tokens are stateless and designed for use in mobile apps, third-party clients, or any frontend that communicates via a simple token-based API.

Each token can be assigned specific abilities, defining what the token is allowed to do within the system. This makes it easy to implement fine-grained access control across your API routes, without relying on cookies or session state — fully aligned with Doppar's stateless architecture.

## How it Works
Instead of keeping a user logged in through a traditional session or cookie, Doppar Flarion uses Personal Access Tokens (PATs) — unique, cryptographically secure tokens linked to a user.
Each token can have:

- `Owner` (user)
- `Abilities (scopes)` — defining what the token is allowed to do
- `Expiration` — to control its lifetime
- `Lookup hash` — for efficient database searching without storing raw tokens. Lookup hash is created using `HMAC-SHA256` with your app key and your random string token.

This design allows external apps, mobile clients, or CLI tools to authenticate securely via headers like:
```php
Authorization: Bearer {token}
```

## Lifecycle of a Token
Every personal access token in Doppar Flarion follows a secure, well-defined lifecycle — from generation to validation and eventual expiration.

### Secure Token Generation
A cryptographically strong random string is created using PHP’s native `CSPRNG` function:
```php
bin2hex(random_bytes(40));
```
This guarantees that every token is unique and unpredictable.

### Hash-Based Lookup Creation
To allow safe database lookups without exposing the real token, an `HMAC-SHA256` hash is generated using the application key
```php
hash_hmac('sha256', $token, config('app.key'));
```

### Secure Storage
The raw token itself is never stored in the database. Instead, only the hashed lookup value and essential metadata (user ID, name, abilities, expiration time, etc.) are saved.

### Safe Return to Client
The method returns both the database record and the raw token string. The raw token is shown only once, allowing the client application to store it securely for future authenticated requests.

This process ensures that each token remains unique, verifiable, and protected throughout its entire lifespan.
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

## Register Provider
Next, register the Flarion service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `FlarionServiceProvider` to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Flarion\FlarionServiceProvider::class,
],
```
This step ensures that Doppar knows about Flarion and can load its functionality when the application boots.

## Publish Configuration
Now we need to publish the configuration files by running this pool command.
```bash
php pool vendor:publish --provider="Doppar\Flarion\FlarionServiceProvider"
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
use Phaseolies\Http\Request;

Route::post('create/token', function (Request $request) {
    $token = $user->createToken('myToken');

    return ['token' => $token->plainTextToken];
});
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
To ensure that all incoming API requests are authenticated, you should apply the Flarion authentication guard to any protected routes in your application — typically inside your `routes/api.php` file.

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

Route::get('user', fn (Request $request) => $request->user())
    ->middleware('auth-api');
```

You can directly add `auth-api` middleware to your route in the following way. This approach provides a cleaner and more declarative way to assign middleware.

Example of method-level middleware
```php
<?php

namespace App\Http\Controllers\API;

use Phaseolies\Utilities\Attributes\Middleware;
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
use Phaseolies\Utilities\Attributes\Route;

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

use Phaseolies\Utilities\Attributes\Middleware;
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
By default, Flarion tokens never expire and may only be invalidated by revoking the token. However, if you would like to configure an expiration time for your application's API tokens, you may do so via the expiration configuration option defined in your application's `config/flarion` configuration file. This configuration option defines the number of minutes until an issued token will be considered expired:
```php
'expiration' => 5,
```
This means all tokens will expire 5 minutes after creation unless otherwise specified.

If you would like to specify the expiration time of each token independently, you may do so by providing the expiration time as the second argument to the `createToken` method. You can override the global setting when generating a token:

```php
$user->createToken('token-name',  now()->addMinutes(5))->plainTextToken;
```
In this example, the token will expire 5 minutes from the current time, regardless of the global config.