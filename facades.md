## Facades
### Introduction
Throughout the Doppar documentation, you will see examples of code that interacts with Doppar's features via "facades". Facades provide a "static" interface to classes that are available in the application's service container. Doppar ships with many facades which provide access to almost all of Doppar's features.

Doppar facades serve as "static proxies" to underlying classes in the service container, providing the benefit of a terse, expressive syntax while maintaining more testability and flexibility than traditional static methods. It's perfectly fine if you don't totally understand how facades work - just go with the flow and continue learning about Doppar.

All of Doppar's facades are defined in the `Phaseolies\Support\Facades` namespace. So, we can easily access a facade like so:
```php
use Phaseolies\Support\Facades\Hash;
use Phaseolies\Support\Facades\Route;

Route::get('hash', function () {
    return Hash::make('text');
});
```

Throughout the Doppar documentation, many of the examples will use facades to demonstrate various features of the framework.

To complement facades, Doppar offers a variety of global "helper functions" that make it even easier to interact with common Doppar features. Some of the common helper functions you may interact with are view, response, url, config, and more. Each helper function offered by Doppar.

For example, instead of using the `Phaseolies\Support\Facades\Response` facade to generate a JSON response, we may simply use the response function. Because helper functions are globally available, you do not need to import any classes in order to use them:
```php
use Phaseolies\Support\Facades\Route;
use Phaseolies\Support\Facades\Response;

Route::get('users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('users', function () {
    return response()->json([
        // ...
    ]);
});
```

### When to Utilize Facades
Facades have many benefits. They provide a terse, memorable syntax that allows you to use Doppar's features without remembering long class names that must be injected or configured manually. Furthermore, because of their unique usage of PHP's dynamic methods, they are easy to test.

### Facades vs. Helper Functions
In addition to facades, Doppar includes a variety of "helper" functions which can perform common tasks like generating views, firing events, dispatching jobs, or sending HTTP responses. Many of these helper functions perform the same function as a corresponding facade. For example, this facade call and helper call are equivalent:
```php
return Phaseolies\Support\Facades\Response::view('profile');

return view('profile');
```

### How Facades Work
In a Doppar application, a facade is a class that provides access to an object from the container. The machinery that makes this work is in the Facade class. Doppar's facades, and any custom facades you create, will extend the base `Phaseolies\Support\Facades\BaseFacade` class.

The Facade base class makes use of the __callStatic() magic-method to defer calls from your facade to an object resolved from the container. In the example below, a call is made to the Doppar cache system. By glancing at this code, one might assume that the static get method is being called on the Cache class:
```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Phaseolies\Support\Facades\Hash;

class UserController extends Controller
{
    public function getAppName()
    {
        return Hash::make('plainText');
    }
}
```

If we look at that `Phaseolies\Support\Facades\Hash` class, you'll see that there is no static method make:
```php
<?php

namespace Phaseolies\Support\Facades;

use Phaseolies\Facade\BaseFacade;

class Hash extends BaseFacade
{
    protected static function getFacadeAccessor()
    {
        return 'hash';
    }
}
```

### Facade Class Reference
Below you will find every facade and its underlying class. This is a useful tool for quickly digging into the API documentation for a given facade root. The service container binding key is also included where applicable.
| Facade   | Class   |  Container Binding |
| -------- | ------- | -------- |
| Auth  | Phaseolies\Auth\Security\Auth    | auth |
| Abort  | Phaseolies\Http\Support\Abort   | abort |
| Config  | Phaseolies\Config\Config    | config |
| Crypt  | Phaseolies\Support\Encryption    | crypt |
| Hash  | Phaseolies\Auth\Security\Hash    | hash |
| Mail  | Phaseolies\Support\Mail\MailService    | mail |
| Redirect  | Phaseolies\Http\RedirectResponse    | redirect |
| Response  | Phaseolies\Http\Response    | response |
| Route  | Phaseolies\Support\Router    | route |
| Session  | Phaseolies\Support\Session    | session |
| URL  | Phaseolies\Support\UrlGenerator    | url |
| Storage  | Phaseolies\Support\Storage\StorageFileService    | storage |
| Log  | Phaseolies\Support\LoggerService    | log |
| Sanitize | Phaseolies\Support\Validation\Sanitizer    | sanitize |
| Cookie | Phaseolies\Support\CookieJar   | cookie |
| Cache | Phaseolies\Cache\CacheStore   | cache |
| App | Phaseolies\Application   | app |
| Schema | Phaseolies\Database\Migration\Schema   | schema |