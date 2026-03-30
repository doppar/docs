---
title: Facades
description: Doppar Facades page
meta:
  - name: keywords
    content: Facades, Doppar Framework, Static Proxy, Service Container
---

## Facades

### Introduction

Doppar facades are static proxies to services bound in the application's
service container. They give you a clean, expressive way to access
framework features without manually injecting or instantiating classes.

Every facade in Doppar lives under the `Phaseolies\Support\Facades`
namespace and extends the base `Phaseolies\Facade\BaseFacade` class,
which uses PHP's `__callStatic()` magic method to forward calls to the
resolved container binding at runtime.
```php
use Phaseolies\Support\Facades\Hash;
use Phaseolies\Support\Facades\Cache;

$hashed = Hash::make('my-secret');
$value  = Cache::get('user:42');
```

There is no magic you need to understand to use facades productively.
Use them where they make your code cleaner, and reach for direct
injection where explicitness matters more.

### Facades vs. Helper Functions

Doppar also ships a set of global helper functions that cover the most
common operations. Many helpers are direct equivalents of a facade call.
Pick whichever style fits the context — both are first-class in Doppar.

These two are equivalent:
```php
use Phaseolies\Support\Facades\Response;

Response::json(['status' => 'ok']);
response()->json(['status' => 'ok']);
```

Helpers are globally available — no import required.

### Using Facades in Controllers

Facades are most naturally used inside controller methods, service
classes, or anywhere you want concise access to a framework service
without constructor injection.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Support\Facades\Cache;
use Phaseolies\Http\Request;

class UserController extends Controller
{
    #[Route(uri: 'users/{id}', methods: ['GET'])]
    public function show(Request $request, int $id): mixed
    {
        return Cache::stash("user:{$id}", 300, function () use ($id) {
            return User::find($id);
        });
    }
}
```

### How Facades Work

Every Doppar facade extends `Phaseolies\Facade\BaseFacade` and
implements a single method: `getFacadeAccessor()`. This returns the
container binding key that the facade proxies.
```php
<?php

namespace Phaseolies\Support\Facades;

use Phaseolies\Facade\BaseFacade;

class Hash extends BaseFacade
{
    protected static function getFacadeAccessor(): string
    {
        return 'hash';
    }
}
```

When you call `Hash::make('text')`, PHP hits `__callStatic()` on
`BaseFacade`, which resolves `hash` from the service container and
forwards the `make` call to the real `Phaseolies\Auth\Security\Hash`
instance. No static state. No singleton abuse. Just a clean proxy.

### Building a Custom Facade

You can expose any container-bound service as a facade in three steps.

**Step 1 — Bind the service:**
```php
public function register(): void
{
    $this->app->singleton('currency', fn() => new CurrencyConverter());
}
```

**Step 2 — Create the facade class:**
```php
<?php

namespace App\Facades;

use Phaseolies\Facade\BaseFacade;

class Currency extends BaseFacade
{
    protected static function getFacadeAccessor(): string
    {
        return 'currency';
    }
}
```

**Step 3 — Use it anywhere:**
```php
use App\Facades\Currency;

$converted = Currency::convert(100, 'USD', 'BDT');
```

### Facade Class Reference

Every built-in Doppar facade, its underlying class, and its container
binding key.

| Facade     | Underlying Class                                    | Binding    |
|------------|-----------------------------------------------------|------------|
| `App`      | `Phaseolies\Application`                            | `app`      |
| `Auth`     | `Phaseolies\Auth\ActorManager`                      | `auth`     |
| `Abort`    | `Phaseolies\Http\Support\Abort`                     | `abort`    |
| `Cache`    | `Phaseolies\Cache\CacheStore`                       | `cache`    |
| `Config`   | `Phaseolies\Config\Config`                          | `config`   |
| `Cookie`   | `Phaseolies\Support\CookieJar`                      | `cookie`   |
| `Crypt`    | `Phaseolies\Support\Encryption`                     | `crypt`    |
| `Hash`     | `Phaseolies\Auth\Security\Hash`                     | `hash`     |
| `Log`      | `Phaseolies\Support\LoggerService`                  | `log`      |
| `Mail`     | `Phaseolies\Support\Mail\MailService`               | `mail`     |
| `Redirect` | `Phaseolies\Http\RedirectResponse`                  | `redirect` |
| `Response` | `Phaseolies\Http\Response`                          | `response` |
| `Sanitize` | `Phaseolies\Support\Validation\Sanitizer`           | `sanitize` |
| `Schema`   | `Phaseolies\Database\Migration\Schema`              | `schema`   |
| `Session`  | `Phaseolies\Support\Session`                        | `session`  |
| `Storage`  | `Phaseolies\Support\Storage\StorageFileService`     | `storage`  |
| `URL`      | `Phaseolies\Support\UrlGenerator`                   | `url`      |


### When to Use Facades vs. Injection

Doppar gives you both tools. Here's a simple way to think about it:

Use **facades** when you want concise, readable access inside a method
and the dependency doesn't need to be swapped for testing or mocked.

Use **`#[Bind]` injection** when the dependency is central to the
class, you want it visible at the call site, or you need to mock it
in tests.

Facade — concise, fine for one-off use
```php
Hash::make($password);
```

`#[Bind]` injection — explicit, mockable, self-documenting
```php
public function store(
    #[Bind(UserRepository::class)] UserRepositoryInterface $user
) { }
```

Both are idiomatic Doppar. Neither is wrong.