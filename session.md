## HTTP Session
### Introduction
Since HTTP is inherently stateless, Doppar provides session support to maintain user-specific data across multiple requests. Sessions allow your application to temporarily store information—such as user preferences, login status, or flash messages—that can persist between page loads.

In Doppar, session data is stored using a backend driver. Out of the box, Doppar supports file-based and cookie-based session drivers, giving you flexibility based on your application's scale and environment.

- File Driver: Stores session data as files on the server's filesystem. This is ideal for small to medium applications or development environments.
- Cookie Driver: Stores the entire session in an encrypted cookie on the client side. This avoids server-side storage and is suitable for lightweight data needs.

Doppar's session system is powered by a clean, unified API, allowing you to interact with sessions in a consistent way regardless of the storage driver you choose.

Whether you're storing a simple user ID or passing flash data between redirects, Doppar makes session handling intuitive and efficient.

### Configuration
Your application's session configuration file is stored at `config/session.php`. Be sure to review the options available to you in this file. By default, Doppar is configured to use the `file` session driver.

### Clearing Session Data in Doppar
Doppar provides a convenient command-line tool to manage session data during development or maintenance. If you ever need to wipe all stored session data—whether you're debugging, resetting user states, or cleaning up after a test—you can use the following console command:
```bash
php pool session:clear
```
#### What It Does
This command will:
- Delete all session files (if you're using the file driver).
- Invalidate all stored session data (note: if you're using the cookie driver, this is not affected).
- Effectively log out all users and reset any session-based data.

### Put and Get Session Data
Doppar makes it simple to store and retrieve session data using intuitive methods. This allows you to persist user-specific information across multiple requests, such as login details, preferences, or temporary messages.

To store data in the session, use the `put()` method:
```php
$request->session()->put($key, $value);
```

To retrieve data from the session, use the `get()` method:
```php
$request->session()->get($key);
```

Here’s how you can store a user’s name in the session and retrieve it later:
```php
$request->session()->put('name', 'doppar');
$name = $request->session()->get('name');
```
If the key doesn't exist in the session, get() will return null by default. You can also pass a fallback value:
```php
$request->session()->get('name', 'default');
```

### Determining if an Item Exists in the Session
You can check a key exists or not in a session data. To determine if an item is present in the session, you may use the has method. The has method returns `true` if the item is present and is not false:
```php
if($request->session()->has('name')){
    // Session key data exists
}
```

### Destroy Session
Doppar provides methods to remove specific session entries or clear all session data entirely. This is useful for logging users out, resetting state, or managing sensitive data securely.
#### Remove a Specific Key
To delete a single item from the session, use the `forget()` method:
```php
$request->session()->forget('key')
```
This will remove the session value associated with 'key' but leave the rest of the session data intact.
#### Clear All Session Data
To completely clear the session, use the `flush()` or `destroy()` method:
```php
$request->session()->flush();
$request->session()->destroy()
```
This wipes all session data for the current user and is typically used during logout or when resetting the application state.

### Session Facades
You can also handle session data using Session facades.
```php
use Phaseolies\Support\Facades\Session;

Session::put('name','mahedi');
Session::get('name');
```

### The Global Session Helper
You may also use the global session PHP function to retrieve and store data in the session. When the session helper is called with a single, string argument, it will return the value of that session key. When the helper is called with an array of key / value pairs, those values will be stored in the session:
```php
Route::get('/home', function () {
    // Retrieve a piece of data from the session...
    $value = session('key');

    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value']);
});
```

### Retrieving All Session Data
If you would like to retrieve all the data in the session, you may use the all method:

```php
$data = $request->session()->all();
```

### Regenerating the Session ID
Regenerating the session ID is often done in order to prevent malicious users from exploiting a session fixation attack on your application.

Doppar automatically regenerates the session ID during authentication. however, if you need to manually regenerate the session ID, you may use the regenerate method:
```php
$request->session()->regenerate();
```