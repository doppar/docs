---
title: Doppar OAuthic
description: Socialite-style OAuth 2.0 authentication package for Doppar
meta:
  - name: keywords
    content: OAuth, OAuth2, Socialite, Authentication, PHP, Doppar
---

## OAuthic

### Introduction

Doppar OAuthic is a modern, extensible authentication package that simplifies OAuth 2.0 integration in Doppar Framework. In addition to traditional, form-based authentication, Doppar OAuthic offers a clean, convenient interface for authenticating users via popular OAuth providers such as `Google`, `GitHub`, and `LinkedIn`. It also supports custom drivers, giving you full control over provider integrations.

Whether you're building a microservice, a SaaS product, or integrating with social platforms, Doppar OAuthic makes OAuth 2.0 secure, consistent, and easy to implement.

## Features

- **OAuth 2.0 Authentication** – Simple integration with major providers
- **Extensible Driver System** – Supports custom providers with ease
- **Customizable Flow** – Fine-tune redirect URLs, token exchange, and user info

## Installation

To get started with `OAuthic`, use the composer package manager to add the package to your doppar project's dependencies:

```bash
composer require doppar/oauthic
```

## Configuration
Before using Doppar OAuthic, you'll need to define the credentials for the OAuth providers your application intends to use. These credentials are typically obtained by registering your application with each provider's developer portal (e.g., Google Cloud Console, GitHub Developer Settings, or LinkedIn Developer Portal).

#### Creating the Configuration File
Doppar OAuthic does not come with a pre-generated config file out of the box. You must manually create a configuration file at
`config/oauthic.php`.

This file should return an associative array of your provider settings, where each key corresponds to a supported provider (e.g., google, github, linkedin). Below is an example structure using environment variables for sensitive credentials:
```php
<?php

return [
    'google' => [
        'client_id' => env('GOOGLE_CLIENT_ID'),
        'client_secret' => env('GOOGLE_CLIENT_SECRET'),
        'redirect' => env('GOOGLE_REDIRECT_URI'),
    ],
    'github' => [
        'client_id' => env('GITHUB_CLIENT_ID'),
        'client_secret' => env('GITHUB_CLIENT_SECRET'),
        'redirect' => env('GITHUB_REDIRECT_URI'),
    ],
    'linkedin' => [
        'client_id' => env('LINKEDIN_CLIENT_ID'),
        'client_secret' => env('LINKEDIN_CLIENT_SECRET'),
        'redirect' => env('LINKEDIN_REDIRECT_URI'),
    ]
];
```
> `Note:` The configuration keys (google, github, linkedin, etc.) must match the driver name you pass to `OAuthic::driver()` in your application.

Need support for custom drivers? Simply add another key with your own identifier, and provide the necessary OAuth credentials in the same format.

## Authentication
Doppar OAuthic provides a simple and fluent API for authenticating users with OAuth 2.0 providers such as GitHub, Google, and LinkedIn. The process typically involves two routes:

- Redirecting the user to the provider's authorization page
- Handling the callback and retrieving the authenticated user's details

### Redirect to the OAuth Provider
You initiate the OAuth flow by redirecting the user to the provider's login/consent screen:
```php
use Doppar\OAuthic\OAuthic;

Route::get('redirect', function () {
    return redirect(OAuthic::driver('github')->redirect());
});
```

### Handle the Callback and Retrieve User Info
After the user authenticates and approves access, the provider will redirect them back to your application. You can then retrieve the authenticated user's details:

```php
use Doppar\OAuthic\OAuthic;
use Phaseolies\Support\Facades\Auth;

Route::get('callback', function () {
    $provider = OAuthic::driver('github')->user();

    $provider->getId();       // Provider user ID
    $provider->getName();     // Full name
    $provider->getEmail();    // Email address
    $provider->getAvatar();   // Profile picture URL
    $provider->getRaw();      // Raw response array from provider

    // Save or Update the user

    // Now login
    Auth::login($user);
});
```

## Stateless Mode
The `stateless()` method disables session-based state verification. This is particularly useful when integrating social login into stateless APIs or applications that do not rely on cookie-based sessions, such as mobile apps or single-page applications (SPAs).
```php
use Doppar\OAuthic\OAuthic;

$provider = OAuthic::driver('github')->stateless()->user();
```

## Custom Provider Integration
Doppar OAuthic is built to be extensible, allowing you to register and authenticate with any custom OAuth 2.0 provider. Whether you're integrating with an internal SSO service or a third-party identity provider not officially supported, you can define your own driver.

### Registering Custom Provider
To register your custom provider class, use the `OAuthic::extend()` method. A common place to do this is within your `App\Providers\AppServiceProvider`:

```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;
use Doppar\OAuthic\OAuthic;
use App\OAuth\Providers\CustomProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot(): void
    {
        OAuthic::extend('custom', CustomProvider::class);
    }
}
```
Once registered, you can use your driver just like the built-in ones:
```php
OAuthic::driver('custom')->redirect();
```

Make sure to add your custom provider’s credentials to `config/oauthic.php`:
```php
'custom' => [
    'client_id' => env('CUSTOM_CLIENT_ID'),
    'client_secret' => env('CUSTOM_CLIENT_SECRET'),
    'redirect' => env('CUSTOM_REDIRECT_URI'),
],
```

## Custom Provider Class
Your custom provider class should extend `Doppar\OAuthic\Drivers\AbstractProvider`, which defines the structure and base functionality for all OAuth drivers. By extending this class, you gain access to shared behavior such as handling access tokens, managing scopes, and preparing HTTP requests using the built-in Axios client.

To support a fully functional OAuth 2.0 flow, your custom provider must implement the following methods:
```php
<?php

namespace App\OAuth\Providers;

use Doppar\OAuthic\User;
use Doppar\OAuthic\Drivers\AbstractProvider;

class CustomProvider extends AbstractProvider
{
    /**
     * Separator used when joining multiple scopes.
     *
     * @var string
     */
    protected string $scopeSeparator = " ";

    /**
     * Get the Custom authorization URL to redirect the user to.
     *
     * @return string
     */
    #[\Override]
    public function getAuthUrl(): string
    {
        //
    }

    /**
     * Get the Custom token endpoint URL.
     *
     * @return string
     */
    #[\Override]
    public function getTokenUrl(): string
    {
        //
    }

    /**
     * Fetch the authenticated user's data from Custom using the access token.
     *
     * @param string $token
     * @return array
     */
    #[\Override]
    public function getUserByToken(#[\SensitiveParameter] string $token): array
    {
        //
    }

    /**
     * Map the raw Custom user array to a standardized User object.
     *
     * @param array $user
     * @return \Doppar\OAuthic\User
     */
    #[\Override]
    public function mapUserToObject(array $user): User
    {
        return new User([
            //
        ]);
    }
}
```

By following this structure, you ensure that your custom provider integrates seamlessly with the rest of Doppar OAuthic and behaves just like the built-in drivers (google, github, linkedin).