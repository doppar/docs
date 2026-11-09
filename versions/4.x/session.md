---
title: Session
description: Doppar session page
meta:
  - name: keywords
    content: session, session driver, file driver, cookie driver, database driver, sessions table, stateless, session lifetime, session regenerate, garbage collection, custom session driver
---

## HTTP Session

HTTP is a stateless protocol, so each request is unaware of the ones before it. Sessions solve this by storing information about a visitor, such as an authenticated user, their preferences, or a message to show after a redirect, and making it available on every subsequent request from the same browser.

Doppar sessions are **lazy**. A session is only read when your code uses it, and it is only stored when it holds data. A request that never touches the session creates no storage and sends no cookie, which keeps API endpoints, health checks, and crawler traffic from leaving anything behind.

```php
$request->session()->put('name', 'doppar');

$name = $request->session()->get('name');
```

### Why Doppar Sessions

- Lazy → nothing is created until the session is used, and an empty session is never saved
- Driver based → the same API works with the `file`, `cookie`, `database`, and `array` drivers, or your own
- Non-blocking → concurrent requests from the same browser never wait for each other
- Self-cleaning → expired sessions are removed automatically, without a background job
- Stateless when you need it → API routes can opt out of sessions entirely

## Configuration

The session configuration file is located at `runtime/config/session.php`. Most options can be set from `env.toml`.

| Option | Environment key | Default | Description |
|--------|-----------------|---------|-------------|
| `driver` | `SESSION_DRIVER` | `file` | Where session data is stored: `file`, `cookie`, `database`, or `array` |
| `lifetime` | `SESSION_LIFETIME` | `120` | Minutes a session stays valid without activity |
| `expire_on_close` | `SESSION_EXPIRE_ON_CLOSE` | `false` | End the session when the browser is closed |
| `files` | | `storage/framework/sessions` | Directory used by the `file` driver |
| `connection` | | `null` | Database connection used by the `database` driver. `null` uses the default connection |
| `table` | | `sessions` | Table used by the `database` driver |
| `cookie` | `SESSION_COOKIE` | `{app name}_session` | Name of the session cookie. Characters other than letters, digits, `-`, and `_` are replaced with `_` |
| `path` | `SESSION_PATH` | `/` | URL path the cookie is valid for |
| `domain` | `SESSION_DOMAIN` | `null` | Domain, and subdomains, the cookie is valid for |
| `secure` | `SESSION_SECURE_COOKIE` | `null` | Send the cookie over HTTPS only. When `null`, it follows the current request |
| `http_only` | `SESSION_HTTP_ONLY` | `true` | Hide the cookie from JavaScript |
| `same_site` | `SESSION_SAME_SITE` | `lax` | `lax`, `strict`, or `none` |
| `stateless` | | `[]` | Request paths that never get a session. See [Stateless Routes](#stateless-routes) |
| `lottery` | | `[2, 100]` | Chance that a request also removes expired sessions. See [Garbage Collection](#garbage-collection) |
| `always_send_xsrf_cookie` | | `false` | Send the `XSRF-TOKEN` cookie on every stateful response, even before a token exists. See [The XSRF-TOKEN Cookie](/versions/4.x/csrf-protection#the-xsrf-token-cookie) |

```toml
SESSION_DRIVER = "file"
SESSION_LIFETIME = 120
```

> **Note:** `stateless`, `lottery`, `always_send_xsrf_cookie`, `connection`, and `table` are optional. A configuration file created before these options existed keeps working with the defaults shown above.

Browsers reject `SameSite=None` cookies that are not `Secure`. When `same_site` is `none`, Doppar always marks the session cookie as secure.

## Session Drivers

| Driver | Stored in | Best for | Limits |
|--------|-----------|----------|--------|
| `file` | Files on the server | Most applications | One server, or a shared filesystem |
| `cookie` | The browser, in an encrypted cookie | Small payloads and servers with no writable storage | About 4 KB in total. Cannot be revoked from the server |
| `database` | A database table | Several servers behind a load balancer, and listing or ending a user's sessions | Two primary-key lookups per request that uses the session |
| `array` | Process memory | Automated tests | Lost when the process ends |

### File Driver

The default driver stores each session as a file under `storage/framework/sessions`. Files are grouped into sub-directories by the first two characters of the session ID, which keeps directories small and cleanup fast, and are created readable by the application user only.

The file driver behaves the same on Linux, macOS, and Windows. Files are written atomically, requests never hold a lock while they run, and a storage path may contain any characters your operating system allows.

Session IDs are 40 character random hexadecimal strings. A session cookie that does not match that format is ignored, so a client can never influence which file Doppar opens.

### Cookie Driver

The cookie driver keeps the whole session inside the cookie. The data is encrypted and authenticated with AES-256-GCM using your `APP_KEY`, so the browser can neither read nor modify it. See [Encryption](/versions/4.x/encryption).

Keep these constraints in mind before choosing it:

- **Size.** A cookie is limited to roughly 4 KB. If the encrypted session grows beyond that, Doppar throws a `SessionException` that lists the largest keys, rather than letting the browser silently discard the cookie.
- **Serialization.** Values must come back from JSON unchanged. Store strings (valid UTF-8), numbers, booleans, `null` and arrays. Objects would silently turn into arrays and binary data cannot be encoded, so both throw a `SessionException` that names the key.
- **Revocation.** There is no server-side record, so `destroy()` and `session:clear` cannot invalidate a cookie that has already been issued. A copied cookie stays valid until it expires or you rotate `APP_KEY`.

> **Note:** Prefer the `file` or `database` driver for anything that holds authentication state or large values, such as validation errors and old input.

### Database Driver

The `database` driver stores each session as a row in a database table. Every server that shares the database shares the sessions, so it suits applications that run on several servers. It also records which user each session belongs to, which lets a user see where they are signed in and sign out of other devices. See [Logging Out Other Devices](/versions/4.x/authentication#logging-out-other-devices).

It works with MySQL and MariaDB, PostgreSQL, and SQLite.

Create the table, then switch the driver:

```bash
php pool session:table
php pool migrate
```

```toml
SESSION_DRIVER = "database"
```

`session:table` writes a migration for the table named by the `table` option:

```php
Schema::create('sessions', function (Blueprint $table) {
    $table->string('id', 128)->primary();
    $table->unsignedBigInteger('user_id')->nullable()->index();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->longText('payload');
    $table->unsignedBigInteger('last_activity')->index();
});
```

| Column | Holds |
|--------|-------|
| `id` | The session ID |
| `user_id` | The user signed in through the default auth actor, or `null` for a guest |
| `ip_address` | The client's IP address, as seen through your [trusted proxies](/versions/4.x/trust-proxies) |
| `user_agent` | The client's user agent, limited to 1024 printable characters |
| `payload` | The session data, serialized and base64 encoded |
| `last_activity` | When the session was last used, as a Unix timestamp |

> **Note:** If your users have UUID or ULID keys, change the `user_id` column to `$table->string('user_id', 36)->nullable()->index();` before you migrate.

The table and connection default to `sessions` and your default database connection. To change them, add these options to `runtime/config/session.php`; `connection` must name one of the connections in `runtime/config/database.php`:

```php
'connection' => env('SESSION_CONNECTION', 'sessions'),
'table' => env('SESSION_TABLE', 'sessions'),
```

Set them before you run `session:table`. When `connection` is set, the migration creates the table on that connection:

```php
Schema::connection('sessions')->create('sessions', function (Blueprint $table) {
    // ...
});
```

Run it on that connection, as the command reminds you:

```bash
php pool migrate --connection=sessions
```

The table name must be a plain name of letters, digits, and underscores, such as `user_sessions`. A schema-qualified name such as `app.sessions` is rejected. On PostgreSQL, use a schema other than `public` by setting the `search_path` of the session connection instead.

The driver is built to stay cheap on every request:

- Loading a session is one lookup by primary key, and saving it is a single upsert. Statements are prepared once per connection and reused.
- A request that does not change the session does not write it. It looks the session up once more at the end, to check that another request did not end or rotate it in the meantime. Its expiry, IP address, and user agent are refreshed at most about once a minute.
- The database is only contacted once the session is used, so requests that never touch it never connect.
- Expired sessions are removed in bounded batches using the `last_activity` index.

If the table is missing, the first request that uses the session throws a `SessionException` that says to run `php pool session:table`.

### Array Driver

The `array` driver keeps sessions in memory for the lifetime of the PHP process. It is intended for tests.

## Reading and Writing Session Data

You may interact with the session through the request instance, the `session()` helper, or the `Session` facade. All three operate on the same session.

```php
use Phaseolies\Support\Facades\Session;

$request->session()->put('name', 'doppar');

session()->put('name', 'doppar');

Session::put('name', 'doppar');
```

### Storing Data

Use `put()` to store a single value, or pass an array to store several at once.

```php
$request->session()->put('theme', 'dark');

$request->session()->put([
    'locale' => 'en',
    'timezone' => 'UTC',
]);
```

### Retrieving Data

Use `get()` to read a value. It returns `null` when the key does not exist, or the default you provide.

```php
$theme = $request->session()->get('theme');

$theme = $request->session()->get('theme', 'light');
```

To read a value and remove it in one step, use `pull()`:

```php
$status = $request->session()->pull('status');
```

To retrieve everything in the session, use `all()`:

```php
$data = $request->session()->all();
```

### Checking for a Key

The `has()` method returns `true` when the key exists and its value is **not `null`**. A value of `false`, `0`, or an empty string still counts as present.

```php
if ($request->session()->has('name')) {
    // The key exists and is not null
}
```

### The Session Helper

The global `session()` helper reads a value when given a string, and stores values when given an array.

```php
$value = session('key');

$value = session('key', 'default');

session(['key' => 'value']);
```

Called with no arguments, it returns the session instance.

```php
session()->put('key', 'value');
```

## Removing Session Data

| Method | Effect |
|--------|--------|
| `forget($key)` | Removes one key and keeps the rest of the session |
| `flush()` | Removes every key from the session |
| `destroy()` | Deletes the stored session and expires its cookie |
| `invalidate()` | Deletes the stored session, issues a new session ID, and clears all data. Use this on logout |

```php
$request->session()->forget('name');

$request->session()->flush();

$request->session()->invalidate();
```

A session that is left with no data is removed when the request finishes, and its cookie is expired, so clearing a session never leaves an empty one behind. If another request saved something into the session in the meantime, that is kept: the session is only removed when nothing is left once both requests' changes are taken together.

`destroy()`, `flush()`, and `invalidate()` end the session, and they are applied even if the request then fails with an exception, so a logout is never lost.

## Regenerating the Session ID

Regenerating the session ID prevents session fixation, where an attacker sets a known ID in a victim's browser before they sign in. Doppar regenerates the ID automatically when a user authenticates, and rotates it again when the session has gone a full `lifetime` without rotation.

An automatic rotation happens when the session is saved, and the old ID keeps leading to the new one for about a minute. Requests that were already running when the ID changed, or that still send the old cookie, carry on with the new session instead of splitting it or being signed out, and several requests that find the session due at the same time end up with the same new ID. Regenerating the ID yourself, which includes logging in, is stricter: the old ID stops working immediately and never leads to the new session, so an ID that an attacker planted before the login is useless afterwards.

To regenerate it manually, use the `regenerate()` method. The data is kept and moves to the new ID.

```php
$request->session()->regenerate();
```

By default the previous session is deleted. Pass `false` to leave it in place until it expires.

```php
$request->session()->regenerate(false);
```

You may read or assign the ID with `getId()` and `setId()`. `setId()` ignores any value that is not a well formed session ID.

## CSRF Token

The session also holds the CSRF token. It is created the first time it is requested, through `csrf_token()` or `token()`, so pages that never render a form do not create a session for it.

```php
$token = $request->session()->token();

$request->session()->regenerateToken();
```

See [CSRF Protection](/versions/4.x/csrf-protection) for how the token is verified.

## Flash Messages

Flash messages are stored in the session and consumed on the next request, which makes them the standard way to confirm an action after a redirect.

```php
return redirect('/register')->withSuccess('User created successfully!');
```

```php
session()->flash('success', 'User created successfully!');
```

Doppar saves the session before a redirect response is sent, so flashed data is available on the page you redirect to. See [Flash Message](/versions/4.x/flash-message) for the full guide.

## Stateless Routes

Endpoints that authenticate with tokens, such as a JSON API, do not need a session. Stateless routes behave like this:

- The session keeps working **in memory** for the duration of the request, so code that reads or writes it does not break
- Nothing is stored and no session cookie is sent
- A session cookie sent by the client is ignored

To make a group of paths stateless, list them in `runtime/config/session.php`. Patterns are matched against the request path and may use `*` as a wildcard. Matching ignores case, and the leading slash is optional, so `api/*` and `/api/*` are the same pattern. Paths that skip CSRF verification and `Request::is()` use the same rules.

```php
'stateless' => ['api/*', 'webhooks/*'],
```

To make a single route stateless, use the `StatelessSession` middleware. Register an alias for it in `App\Http\Gateway`. Aliases are looked up in the `web` group for normal requests and in the `api` group for `api/*` requests, so register it under whichever group your route belongs to.

```php
public array $routeMiddleware = [
    'web' => [
        'stateless' => \Phaseolies\Middleware\StatelessSession::class,
    ],
];
```

Then assign it to the route:

```php
Route::post('webhooks/payments', [PaymentWebhookController::class, 'handle'])
    ->middleware('stateless');
```

Or with an attribute, which does not need an alias:

```php
use Phaseolies\Middleware\Attributes\Middleware;
use Phaseolies\Middleware\StatelessSession;

#[Middleware([StatelessSession::class])]
public function handle()
{
    //
}
```

> **Note:** The CSRF middleware belongs to the `web` group. It sends the `XSRF-TOKEN` cookie only when the request already created a token, for example by rendering a form or calling `csrf_token()`, so visits that never need one do not create a session. Keep API routes out of that group, or list them under `stateless`.

## Garbage Collection

Expired sessions are removed automatically. When a session is saved, there is a small chance, controlled by `lottery`, that the request also deletes expired sessions from one portion of the session store. The sweep runs after the response has been sent, so it never delays it. Each sweep is bounded, so its cost stays constant however many sessions exist.

Two chances in a hundred
```php
'lottery' => [2, 100],
```

Set the first value to `0`, or set `lottery` to `null` or an empty array, to disable the automatic sweep, for example when you prefer to schedule cleanup yourself.

### Console Commands

```bash
php pool session:gc
```

Removes every expired session at once. Use it to clear a backlog, or run it from the [task scheduler](/versions/4.x/task-schedule).

```bash
php pool session:clear
```

Removes every stored session, which signs out all users. It has no effect on the `cookie` driver, because those sessions live in the browser.

```bash
php pool session:table
```

Creates the migration for the `database` driver's table. See [Database Driver](#database-driver).

## Concurrent Requests

A session is not locked while a request runs, so several requests from the same browser run in parallel, which is what makes pages that issue many AJAX calls fast.

When a request saves the session, the `file` and `database` drivers briefly lock it, reads the session as other requests have left it, and applies only what this request changed: keys it added, changed or removed, including inside arrays such as flash data. The `file` driver's lock covers a small group of sessions. The `database` driver locks the one session only, with `GET_LOCK()` on MySQL and MariaDB and an advisory lock on PostgreSQL; on SQLite, which allows one writer at a time, it uses a short write transaction. Either way, visitors who are unrelated to each other never wait for one another. Two requests that touch different keys, such as one adding to the cart while another flashes a message, both keep their changes.

Lists are merged by what each request added and removed, not by position. Two requests that each append to the same list both keep their item, and the same is true when both create the list for the first time.

```php
// The stored cart is ['x']. Request A appends 'apple', request B appends 'pear'.
// Once both have saved: ['x', 'apple', 'pear'], in either order of arrival.
$request->session()->put('cart', [...$request->session()->get('cart', []), 'apple']);
```

> **Note:** When requests race on the same list, the position of an item that one of them added or changed is not defined. Do not rely on the order of a list that several requests modify at once.

When two requests change the **same** value, the last one to finish wins, and a counter incremented by both can lose an update. Keep values that need atomic updates in your database or [cache](/versions/4.x/caching).

Ending a session always wins over a request that was still running on it:

- A request that was still running when the session was destroyed, by a logout, `invalidate()`, `destroy()`, or expiry, does not bring it back. Its changes are dropped and it sends no session cookie, so the user stays logged out.
- Calling `invalidate()` or `flush()` replaces the stored session instead of merging, so a logout always clears everything.
- Ending a session also ends the session an automatic rotation made from it, so a logout that overlaps a rotation cannot be undone by it.
- The `cookie` driver cannot merge or end a session from the server, because the data travels with each request.

If the lock cannot be acquired within two seconds, the session is saved without it rather than failing the request, and a warning is written to the log, since concurrent changes to that session may then be lost. A warning that appears regularly means requests are being held up long enough to be worth looking at.

## Custom Drivers

You may add a driver by implementing `Phaseolies\Session\Contracts\SessionStoreInterface` and registering it from a [launcher](/versions/4.x/launchers).

| Method | Responsibility |
|--------|----------------|
| `read(string $cookieValue): ?array` | Return `['id' => ..., 'data' => [...]]` for a valid session, or `null` when it is unknown, tampered with, or expired |
| `write(string $id, array $data): string` | Persist the session and return the value the cookie must carry |
| `touch(string $id): void` | Extend the lifetime without rewriting the data |
| `destroy(string $id): void` | Remove one session |
| `gc(int $maxLifetime): int` | Remove every expired session and return how many were removed |
| `sweep(int $maxLifetime): int` | Remove a bounded portion of the expired sessions. This runs on requests, so its cost must not grow with the number of sessions |
| `flush(): int` | Remove every session |
| `isClientSide(): bool` | Return `true` only when the data is stored in the cookie itself |

A driver that can lock a session may also implement `Phaseolies\Session\Contracts\LockableSessionStore` and its `synchronized(string $id, callable $callback): mixed` method. Doppar then merges concurrent changes inside it, as the file driver does.

A driver that knows which user each session belongs to may also implement `Phaseolies\Session\Contracts\UserSessionStore`. Its `userSessions(int|string $userId): array` and `destroyUserSessions(int|string $userId, array $keep = []): int` methods let `auth()->sessions()` and `auth()->logoutOtherDevices()` list and end a user's sessions, as the database driver does.

`$keep` holds the IDs of the current device's session: the one its cookie will carry, and the one it is still stored under when its ID was regenerated during the request. A session whose ID was rotated automatically is stored for a short time as a pointer to its new ID. Read it with `Phaseolies\Session\RotationPointer::target($data, false)`, and keep or end the session it leads to along with the one you were given.

```php
<?php

namespace App\Launchers;

use App\Session\RedisSessionStore;
use Phaseolies\Launchers\ServiceLauncher;
use Phaseolies\Session\SessionManager;

class SessionServiceLauncher extends ServiceLauncher
{
    public function register(): void
    {
        SessionManager::extend('redis', function (array $config) {
            return new RedisSessionStore($config['lifetime'] * 60);
        });
    }

    public function launch(): void
    {
        //
    }
}
```

The driver is then selected like any other:

```toml
SESSION_DRIVER = "redis"
```

> **Note:** Treat the value passed to `read()` as untrusted input, because it comes straight from the cookie. Validate its format before using it to build a key, a path, or a query.

## Error Handling

A misconfigured driver is reported when the session is first used, as a `Phaseolies\Session\Exceptions\SessionException`. It is handled like any other exception: the full message is shown when `APP_DEBUG` is `true`, and a generic server error page is shown otherwise. Internal details, such as file paths, are never included in the production response.

## Upgrading to Doppar 4.9

Sessions no longer use PHP's native session functions as of Doppar v4.9.0. If you are upgrading an application that stored sessions with an earlier version, be aware of the following:

- **Existing sessions are not carried over.** Users are signed out once, the first time they visit after the upgrade. Old `sess_*` files are removed by `php pool session:gc`.
- **Do not read `$_SESSION` or call `session_start()`.** Use `$request->session()`, the `session()` helper, or the `Session` facade, which work with every driver.
- **`csrf_token()` creates the token on demand.** Code should call it rather than reading `_token` from the session directly.
- **Session changes are not saved when a request ends with an uncaught exception.** The session is saved just before the response is sent. A request that ends in an unhandled error, a `500`, never gets that far, so flash data and other changes made before the failure are dropped. With PHP's native sessions they were saved when the script ended, so code that flashes a message and then throws must catch the exception or save the data another way. A logout is the exception: `destroy()`, `flush()`, and `invalidate()` are still applied, so a user is never left signed in by a request that failed after logging them out. Requests that end with an HTTP exception, such as `abort(404)`, and validation failures that redirect back with their errors, are saved as usual.
- **Changes made while a streamed response is sent are not saved.** The session is saved before the response is sent, because that is when its cookie is added. A `StreamedResponse` callback runs after that, so a value it writes to the session is lost. Write to the session before returning the response.
- **Error pages can use the session.** The page of an HTTP exception, such as a custom `403` view that renders a form with `csrf_token()`, is built before the session is saved, so what it writes is kept and its form can be submitted.
- **The `XSRF-TOKEN` cookie is sent lazily.** It is only sent once a token exists, for example after a page renders a form or calls `csrf_token()`. A front end that reads the cookie instead of a page, such as an axios or fetch based client that never loads a form, will get a `419` on its first `POST`. Expose the token with `csrf_token()` (a meta tag or a route that returns it), or set `'always_send_xsrf_cookie' => true` in `runtime/config/session.php`. See [The XSRF-TOKEN Cookie](/versions/4.x/csrf-protection#the-xsrf-token-cookie).
