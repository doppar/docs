## Authentication
### Introduction
Authentication is a fundamental part of most web applications. It ensures that users can securely log in, access protected resources, and maintain session integrity. Doppar simplifies this process by providing built-in tools and scaffolding to help you implement user authentication quickly and securely.

With a single command, you can generate the routes, controllers, and views necessary for user `login`, `registration`, and `logout`. Doppar's authentication system is built on top of robust components like session handling and secure password hashing, ensuring your users' data remains protected.

In the following sections, we’ll walk through generating authentication scaffolding and implementing core features such as login and logout using the Auth facade provided by:
```php
use Phaseolies\Support\Facades\Auth;
```

### Generating Authentication Scaffolding
To generate the authentication system, simply run the following command in your terminal:
```bash
php pool make:auth
```
This will scaffold the essential authentication logic into your application, including:

- Login and logout routes
- Controllers for handling authentication
- Basic views
- Middleware for route protection


### Custom Login Functionality
Doppar's authentication system allows you to easily verify user credentials and establish a login session. Doppar provides `Phaseolies\Support\Facades\Auth` to create authentication functionalitis. Call the `try()` method and pass in the validated credentials. This will check if the credentials match an existing user and log them in if successful.
```php
<?php

namespace App\Http\Controllers\Auth;

use Phaseolies\Http\Request;
use Phaseolies\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class LoginController extends Controller
{
    public function login(Request $request)
    {
        $request->sanitize([
            'email' => 'required|email|min:2|max:100',
            'password' => 'required|min:2|max:20',
        ]);

        if (Auth::try($request->passed())) {
            // User is logged in
        }
    }
}
```

### Auth Via Remember Me
If you want to create auth using remember me, then you just need to pass, true as the second parameter in try method like
```php
if (Auth::try($request->passed(), true)) {
    // User is logged in
}
```

### Auth using login()
In addition to logging in users with credentials, Doppar allows you to authenticate a user directly using a user object. This is useful in cases such as after user registration, social login, or admin impersonation features.
```php
if (Auth::login($user)) {
    // User is logged in
}
```
In this example:
- The login() method accepts a valid user object.
- It sets the user as the currently authenticated user and establishes a session.

This approach bypasses credential checking since you're explicitly assigning the user object.

You can also use login() via remember_token by passing true as the second argument.
```php
if (Auth::login($user, true)) {
    // User is logged in with remember_token token
}
```
### Auth using loginUsingId()
The `loginUsingId()` method allows you to log in a user by their primary key (usually the id). This is particularly useful when you already know the user's ID and want to authenticate them directly—without querying for the full user object first.

This method also creates a persistent login by storing the user's session.

```php
if (Auth::loginUsingId($user->id)) {
    // User is logged in
}
```

### Auth using onceUsingId()
This method logs in a user for a single request only, meaning authentication is not stored in the session or cookies.
```php
if (Auth::onceUsingId(1)) {
    // User is logged in
}
```

### Custom Authentication Key
In some applications, authentication is not based on `email` but instead uses a mobile number or username. By default, Doppar uses `email` and `password` for authentication. However, you can change this behavior by overriding the `getAuthKeyName()` method in your User model.
```php
/**
 * Get the authentication key name used for identifying the user.
 * @return string
 */
public function getAuthKeyName(): string
{
    return "username"; // set any key for authentication
}
```
Now, instead of logging in with an `email`, Doppar will use the `username` field for authentication.

### Logout
To destroy user session, simply call logout function.

```php
<?php

namespace App\Http\Controllers\Auth;

use Phaseolies\Http\Request;
use Phaseolies\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class LoginController extends Controller
{
    public function logout()
    {
        Auth::logout(); // User login session is now destroyed
    }
}
```

### Get Authenticated User Data
To get the current authenticated user data, Doppar has `Auth::user()` method and `auth()` helper. Simply call
```php
use Phaseolies\Support\Facades\Auth;

// Using the Auth facade
Auth::user(); // Returns the authenticated user instance

// Using the auth() helper
auth()->user(); // Same as above

// From the current request instance
$request->user(); // Useful in controller methods

// Using the request() helper
request()->user(); // Same as $request->user()

// Alternatively, using auth() directly from request
request()->auth(); // Returns the authenticated user
```
Each of these methods returns the current authenticated user object, allowing you to access properties like `$user->name`, `$user->email`, etc.
