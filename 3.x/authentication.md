---
title: Authentications
description: Doppar authentications page
meta:
  - name: keywords
    content: authentications
---

## Authentication
### Introduction
Authentication is a fundamental part of most web applications. It ensures that users can securely log in, access protected resources, and maintain session integrity. Doppar simplifies this process by providing built-in tools and scaffolding to help you implement user authentication quickly and securely.

With a single command, you can generate the routes, controllers, and views necessary for user `login`, `registration`, and `logout`. Doppar's authentication system is built on top of robust components like session handling and secure password hashing, ensuring your users' data remains protected.

In the following sections, we’ll walk through generating authentication scaffolding and implementing core features such as login and logout using the Auth facade provided by:
```php
use Phaseolies\Support\Facades\Auth;
```

## Generating Authentication Scaffolding
To generate the authentication system, simply run the following command in your terminal:
```bash
php pool make:auth
```
This will scaffold the essential authentication logic into your application, including:

- Login and logout routes
- Controllers for handling authentication
- Basic views
- Middleware for route protection

## Authentication Model Configuration
The `config/auth.php` file in Doppar defines the Entity model used for authentication.

By default, Doppar uses the `App\Models\User` model. However, if your application requires custom authentication logic or uses a different user-related model (e.g., Admin, Customer, or Member), you can update this value accordingly.
```php
<?php

return [
    // Specifies the model class used for authentication.
    // You can replace this with a custom model if needed.
    'model' => App\Models\User::class,
];
```
If you have a custom model like `App\Models\Admin`, you can update the configuration:
```php
'model' => App\Models\Admin::class,
```

## Custom Login Functionality
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

## Auth Via Remember Me
If you want to create auth using remember me, then you just need to pass, true as the second parameter in try method like
```php
if (Auth::try($request->passed(), true)) {
    // User is logged in
}
```

## Auth using login()
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
## Auth using loginUsingId()
The `loginUsingId()` method allows you to log in a user by their primary key (usually the id). This is particularly useful when you already know the user's ID and want to authenticate them directly—without querying for the full user object first.

This method also creates a persistent login by storing the user's session.

```php
if (Auth::loginUsingId($user->id)) {
    // User is logged in
}
```

## Auth using onceUsingId()
This method logs in a user for a single request only, meaning authentication is not stored in the session or cookies.
```php
if (Auth::onceUsingId(1)) {
    // User is logged in
}
```

## Custom Authentication Key
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

## Logout
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

## Get Authenticated User Data
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

## Two Factor Authentication
### Introduction
Doppar Framework provides a robust, secure, and developer-friendly implementation of Two-Factor Authentication (2FA) to enhance user account protection. Built on top of industry standards such as `TOTP (Time-Based One-Time Password Algorithm)`, this module allows seamless integration of 2FA into any user-based application using the framework.

- Secure TOTP secret generation using Base32 encoding.
- QR code provisioning URI generation, allowing users to easily set up 2FA with authenticator apps like Google Authenticator, Microsoft Authenticator, or Authy.
- Encryption of secrets and recovery codes using the framework's built-in Crypt facade for at-rest security.
- Recovery code management, including automatic validation and regeneration.
- Clock abstraction via `Psr\Clock\ClockInterface` to ensure consistent TOTP timing, testability, and future extensibility.
- Custom verification logic that supports both OTP and backup codes.
- Full integration with the framework's Auth system, ensuring smooth 2FA flows during login.

This system is designed with flexibility and developer ergonomics in mind. You can easily extend, override, or integrate this trait into your authentication workflow while relying on secure defaults.

In the sections that follow, we’ll explore how to enable, disable, verify, and customize 2FA for users within your Doppar-powered application.

### Activating Two-Factor Authentication
Before initiating two-factor verification, it’s essential to determine whether the user has 2FA enabled. This check helps ensure that only users with an active 2FA configuration are prompted to complete the additional authentication step. The following example demonstrates how to conditionally redirect users to the 2FA verification page after a successful login.
```php
if (Auth::enableTwoFactorAuth()) {
    // Activated Two-Factor Authentication
}
```
By performing this check, you can maintain a secure and user-aware authentication flow that integrates seamlessly with the rest of your application.

### Disable Two-Factor Authentication
When a user no longer wishes to use two-factor authentication, or during account recovery scenarios, the 2FA credentials can be removed from their account securely. The `disableTwoFactorAuth()` method clears both the encrypted secret and recovery codes associated with the current authenticated user.

Here’s how to disable 2FA:
```php
Auth::disableTwoFactorAuth();
```

This method will:
- Remove the user's TOTP secret
- Remove any existing recovery codes
- Persist the changes to the database

Use this action in settings or security management screens where users can manage their authentication preferences.

### Verify Two-Factor Authentication Code
The `verifyTwoFactorCode(string $code)` method is responsible for validating a user’s Time-based One-Time Password (TOTP) during the two-factor authentication process. This ensures that the submitted code matches the one generated from the user's encrypted secret using the current time window.

This method is typically used after a user submits their 2FA code during login.
```php
if (Auth::verifyTwoFactorCode($request->code)) {
    // Code is valid, complete login
}

// Submitted code is invalid
```

Key Points:
- The method decrypts the user's stored TOTP secret.
- It uses the current time (via ClockInterface) to verify the provided code.
- Accepts a small time window drift for better UX (e.g., ±30 seconds).
- Returns true if the code is valid, otherwise false.

Use this method as part of your 2FA verification controller or middleware to authenticate users securely.

### Verify Recovery Code
The `verifyRecoveryCode($user, $code)` method allows users to authenticate using one of their previously generated 2FA recovery codes. This is especially useful when the user loses access to their TOTP device (e.g., phone or authenticator app).
```php
if (Auth::verifyRecoveryCode($user, $request->code)) {
    // Recovery code is valid, complete login
}

// Recovery code is invalid
```

How it Works:
- Checks if the user has recovery codes stored.
- Decrypts and compares each recovery code (case-insensitive and whitespace-trimmed) with the user’s input.
- If a match is found:
  - The code is consumed (removed from the list).
  - The updated list is re-encrypted and saved.
  - Returns true for successful verification.
- If no match is found, or no codes exist, returns false.

> Security Note: Each recovery code is single-use by design to prevent replay attacks. Once a code is used, it is permanently removed from the user's account.

### Generate New Recovery Codes
The `generateNewRecoveryCodes()` method creates a fresh set of backup recovery codes for the currently authenticated user. These codes provide an alternative way to access the account if the user loses access to their two-factor authentication device.
```php
$recoveryCodes = Auth::generateNewRecoveryCodes();

// Display or provide these codes securely to the user
foreach ($recoveryCodes as $code) {
    //
}
```
Recovery codes are one-time use only and should be stored securely by the user. Generating new codes invalidates all previously issued codes.

### Determine 2FA Status
The `hasTwoFactorEnabled($user)` method determines whether a given user currently has two-factor authentication enabled by checking the presence of their encrypted 2FA secret.
```php
if (Auth::hasTwoFactorEnabled($user)) {
    // 2FA is enabled for this user
} else {
    // 2FA is not enabled
}
```

### Complete Two-Factor Authentication 
The `completeTwoFactorLogin()` method finalizes the 2FA login process after successful verification of the one-time password or recovery code. It authenticates the user into the application and manages session state accordingly.
```php
if (Auth::completeTwoFactorLogin()) {
    // User is fully authenticated, redirect to dashboard
}
```

This method is typically called immediately after verifying a valid 2FA code to establish a fully authenticated user session.

### 2FA with "Remember Me"
This example demonstrates how to perform user authentication with optional "remember me" functionality and enforce two-factor authentication if enabled.
```php
if (Auth::try($request->passed(), $remember)) {
    if (Auth::hasTwoFactorEnabled($user)) {
        // Redirect to 2FA verification if enabled
    }

    // Successful login without 2FA
}
```
`Auth::try()` attempts to authenticate the user with the provided credentials and optionally sets a persistent login cookie if `$remember` is true. This approach ensures that users with two-factor authentication enabled are required to complete the additional security step while still supporting “remember me” for convenience.

### Generate QR Code
The `generateTwoFactorQrCode(string $qrCodeUrl)` method generates a QR code image that encodes the TOTP provisioning URI. This QR code can be scanned by authenticator apps (such as Google Authenticator or Authy) to set up two-factor authentication for the user’s account.
```php
$qrCodeSvg = Auth::generateTwoFactorQrCode($totp->getProvisioningUri());

// Output or embed $qrCodeSvg in your view for the user to scan
```

Generates the QR code as an SVG markup string by default, ensuring high-quality scalable graphics. This method is essential for presenting users with a simple way to enroll their authenticator apps during the 2FA setup process.