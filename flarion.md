## Doppar Flarion
### Introduction
Flarion provides a lightweight API authentication system for the Doppar framework.

Flarion allows each user of Doppar application to generate and manage multiple API tokens. These tokens are stateless and designed for use in mobile apps, third-party clients, or any frontend that communicates via a simple token-based API.

Each token can be assigned specific abilities, defining what the token is allowed to do within the system. This makes it easy to implement fine-grained access control across your API routes, without relying on cookies or session state — fully aligned with Doppar's stateless architecture.

### How it Works
Doppar Flarion exists to solve one problems. Let's discuss it before digging deeper into the library.

### API Tokens
Flarion is a lightweight authentication package for the Doppar framework that allows you to issue API tokens to your users without the complexity of OAuth. This functionality is inspired by platforms like GitHub, where users can generate personal access tokens from their account settings. For example, your Doppar application might have a settings page where users can create API tokens to integrate with external services or automate workflows.

Flarion handles the generation, storage, and management of these tokens. Tokens are stored in a dedicated database table and can be assigned specific abilities (scopes) to control access. They typically have a long lifespan (often lasting for years), but users can delete them at any time for security.

Incoming HTTP requests are authenticated by checking the Authorization header for a valid token. If the token exists and is valid, Flarion will resolve the corresponding user and authorize the request accordingly — all in a stateless, token-based manner ideal for API-first applications.

### Installation
You may install Doppar Flarion via the `composer require` command:
```bash
composer require doppar/flarion
```

### Publish Configuration
Now we need to publish the configuration files by running this pool command
```bash
php pool vendor:publish --provider="Doppar\Flarion\FlarionServiceProvider"
```

Now run migrate command to migrate `personal_access_token` table
```bash
php pool migrate
```

### Register Provider
Next, register the Flarion service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `FlarionServiceProvider` to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Flarion\FlarionServiceProvider::class,
],
```
This step ensures that Doppar knows about Flarion and can load its functionality when the application boots.


### API Token Authentication
Flarion allows you to issue API tokens / personal access tokens that may be used to authenticate API requests to your application. When making requests using API tokens, the token should be included in the Authorization header as a `Bearer token`.

To begin issuing tokens for users, your User model should use the `Doppar\Flarion\Tokenable` trait:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;
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
    $token = $request->user()->createToken($request->token_name);

    return ['token' => $token->plainTextToken];
});
```

You may access all of the user's tokens using the tokens Eloquent relationship provided by the `Doppar\Flarion\Tokenable` trait:
```php
foreach ($user->tokens as $token) {
    // ...
}
```

### Token Abilities
Flarion allows you to assign "abilities" to tokens. Abilities serve a similar purpose as OAuth's "scopes". You may pass an array of string `abilities` as the third argument to the `createToken` method:
```php
return $user->createToken('token-name', null, ['post-update'])->plainTextToken;
```
When handling an incoming request authenticated by Flarion, you may determine if the token has a given ability using the middleware and tokenCan method:
```php
Route::post('token_scope', [PostController::class, 'update'])
    ->middleware('auth-api:post-update');
```

Or you can did the same job like
```php
if ($user->tokenCan('post-update')) {
    // ...
}
```

### Check Multiple Abilities
You can also check for multiple abilities by joining them with an ampersand (&). The token must have all listed abilities to pass the check:
```php
Route::post('token_scope', [PostController::class, 'update'])
    ->middleware('auth-api:post-update&post-another');
```
Each ability must exactly match one of the abilities assigned to the token. If any of the specified abilities are missing from the token, the request will be rejected with a `unauthorized` response.

### Protecting Routes
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

Route::get('user', function (Request $request) {
    return $request->user();
})->middleware('auth-api');
```

### Revoking Tokens
You may "revoke" tokens by deleting them from your database using the tokens relationship that is provided by the `Doppar\Flarion\Tokenable` trait:
```php
// Revoke all tokens...
$user->tokens()->delete();

// Revoke the token that was used to authenticate the current request...
$request->user()->tokens()->where('token', '=', $tokenId)->delete();
$request->user()->currentAccessToken()->delete();

// Revoke a specific token...
$user->tokens()->where('id', '=', $tokenId)->delete();
```

### Token Expiration
By default, Flarion tokens never expire and may only be invalidated by revoking the token. However, if you would like to configure an expiration time for your application's API tokens, you may do so via the expiration configuration option defined in your application's `config/flarion` configuration file. This configuration option defines the number of minutes until an issued token will be considered expired:
```php
'expiration' => 5,
```
This means all tokens will expire 5 minutes after creation unless otherwise specified.

If you would like to specify the expiration time of each token independently, you may do so by providing the expiration time as the second argument to the `createToken` method. You can override the global setting when generating a token:

```php
return $user->createToken('token-name',  now()->addMinutes(5))->plainTextToken;
```
In this example, the token will expire 5 minutes from the current time, regardless of the global config.