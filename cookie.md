## Cookie
Doppar provides two ways to manage cookies in the application. One is `Phaseolies\Support\Facades\Cookie` facades and another is global `cookie()` helper fucntion. Each approach allows you to store, retrieve, and delete cookies efficiently. Let's explore both methods in detail.

### Storing Cookies
The Cookie facade provides a structured way to manage cookies. It allows you to store, retrieve, and remove cookies with additional options like expiration time, path, and security settings. To store cookies, simply use `store()` function.
```php
use Phaseolies\Support\Facades\Cookie;

Cookie::store('cookie_name', 'cookie_value');
```

You can do the same job using `cookie()` helper like that
```php
cookie()->store('cookie_name', 'cookie_value');
```

### Cookie with Expiration Time in Doppar
When working with cookies in Doppar, you can define a custom expiration time to control how long the cookie should remain on the user's browser. The Cookie::store() method allows for flexible expiration formats, giving you full control over cookie lifetime.

Here are the three common ways to set the expiration. You can define the expiration time as a Unix timestamp (in seconds):
```php
cookie()->store('user_token', 'abc123', ['expires' => time() + 3600]); // Expires in 1 hour
```

You can also pass a `DateTime` object to set a specific future date and time:
```php
$expireDate = new \DateTime('+1 day');
cookie()->store('user_token', 'abc123', ['expires' => $expireDate]); // Expires in 24 hours
```

You can provide a string that strtotime() can parse, for maximum readability:
```php
cookie()->store('user_token', 'abc123', ['expires' => '+1 week']); // Expires in 7 days
```

You can use Carbon also like that
```php
use Carbon\Carbon;

cookie()->store('temp_data', 'value456', [
    'expires' => Carbon::parse('next friday 3pm')
]);
```

### Cookie with Path
In Doppar, you can restrict a cookie to a specific URL path using the path option. This means the cookie will only be sent to the server when the request URI matches the defined path or a subpath.

#### Defining a Cookie with a Path
Here's how to create a cookie that's only accessible under `/settings:`
```php
cookie()->store('prefs', 'dark', ['path' => '/settings']);
```
In this example:
- `'prefs'` is the cookie name.
- `'dark'` is the value.
- `'path' => '/settings'` ensures the cookie is only available for URLs starting with `/settings.`

### Cookie With Domain
Doppar allows you to assign cookies to a specific domain (or subdomains) using the domain option. This is especially useful when your application spans multiple subdomains and you want the same cookie to be accessible across all of them.

```php
cookie()->store('session', 'xyz789', ['domain' => '.example.com']);
```

In this example:
- `'session'` is the cookie name.
- `'xyz789'` is the value.
- `'domain' => '.example.com'` allows the cookie to be shared across example.com, www.example.com, admin.example.com, etc

### Secure & HttpOnly
When storing cookies, security is a crucial aspect, especially for authentication and sensitive user data. The secure and httponly attributes help protect cookies from attacks such as cross-site scripting (XSS) and man-in-the-middle (MITM) attacks.
```php
cookie()->store('auth', 'secure123', [
    'secure' => true,    // HTTPS only
    'httponly' => true   // JavaScript cannot access
]);
```

### SameSite Attribute
The `SameSite` attribute helps protect your application against Cross-Site Request Forgery (CSRF) attacks by controlling when cookies are sent along with cross-origin requests. In Doppar, you can configure this easily when storing a cookie.
```php
use Phaseolies\Http\Response\Cookie;

cookie()->store('csrftoken', 'rand123', [
    'samesite' => Cookie::SAMESITE_STRICT
]);
```
### Partitioned (for CHIPS)
Setting `'partitioned' => true` allows a cookie to be accessed only within a specific cross-site context, without being shared across different origins. This helps prevent third-party tracking while still enabling functionalities like embedded content authentication.

```php
cookie()->store('tracking', 'user123', [
    'partitioned' => true  // For cross-site cookies
]);
```

### Raw Cookie (no URL encoding)
By default, cookies are URL-encoded, meaning special characters like = and & are automatically converted to a safe format. However, if you need to store data without encoding, you can use the raw option.
```php
cookie()->store('raw_data', 'some=value', [
    'raw' => true  // Disables URL encoding
]);
```

### Forever Cookie
A forever cookie is a cookie that does not expire or is set with a very distant expiration date, ensuring it remains stored in the user's browser for an extended period. These cookies are useful for remembering user preferences, authentication tokens, and long-term tracking. You can store forever cookie as like below by using forever method.
```php
cookie()->forever('remember_me', 'yes', [
    'path' => '/',
    'httponly' => true
]);
```

### Retrieving Cookies
Accessing cookies in Doppar is simple and can be done using either the Cookie facade or the cookie() helper. Both methods allow you to retrieve values that were previously stored in the user's browser.
```php
use Phaseolies\Support\Facades\Cookie;

Cookie::get('user_token');
cookie()->get('user_token');
```
Functionally identical to the facade, but this provides a cleaner syntax especially in more compact or helper-driven code.

### Check Cookie Existence
Before accessing a cookie, it’s best practice to check if it exists to avoid errors. Doppar provides a method to verify whether a specific cookie is present using `has()`
```php
if (cookie()->has('user_token')) {
    // Cookie exists
}
```

### Delete Cookie
When a cookie is no longer needed, it should be deleted to ensure proper data management and security. Doppar provides a way to remove cookies using the `remove()` method.
```php
cookie()->remove('user_token');

// With Path/Domain
cookie()->remove('old_cookie', [
    'path' => '/special',
    'domain' => '.example.com'
]);
```

### Advanced Cookie Usage: Create Without Sending
In some cases, you might want to create a cookie but not immediately send it to the user's browser. This can be useful for situations where you need to modify the cookie before sending it or store it for later in the same request cycle. Doppar provides a way to create cookies in advanced scenarios like this by using the make() method.
```php
use Phaseolies\Http\Response\Cookie;

$cookie = cookie()->make('preview_mode', 'enabled', [
    'expires' => '+30 minutes',
    'samesite' => Cookie::SAMESITE_LAX
]);

// Send later
cookie()->store($cookie);
```

### Modifying an Existing Cookie
In some cases, you may need to update the value or properties of an already created cookie. Doppar provides a convenient way to modify an existing cookie using methods like `withValue()` and `withExpires()`. These methods allow you to change the value or expiration of a cookie after it has been created.
```php
cookie()->make('existing_cookie', '1')
    ->modify()
    ->withValue('10')
    ->withExpires('+1 day')
    ->update();
```

You can extend it also like that
```php
Cookie::make('existing_cookie', 'light')
    ->modify()
    ->withValue('dark')
    ->withPath('/settings')
    ->withDomain('.example.com')
    ->withSecure(true)
    ->withSameSite('lax')
    ->update();
```
This flexible approach lets you fully control cookie behavior without recreating it from scratch.

### Fetch All Cookies
If you need to retrieve all cookies that are currently stored and sent with a request, Doppar provides two simple ways to access them. This is especially useful when debugging or processing multiple cookies at once.
```php
$request->cookies->all();
```
This retrieves all cookies as an associative array, where the keys are cookie names and the values are their respective data.

You can use `cookie()` helper also to get all cookies.
```php
cookie()->all();
```
This method gives you the same result—an array of all available cookies—without needing direct access to the request instance.


