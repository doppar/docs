---
title: Upgrade Guide
description: A step-by-step guide for upgrading a Doppar 3.x application to Doppar 4.x
meta:
  - name: keywords
    content: upgrade guide, doppar 4.x, migration, launchers, breaking changes
---

  - [Upgrade Guide](#upgrade-guide)
  - [Estimated Upgrade Time](#estimated-upgrade-time)
  - [PHP 8.5 Required](#php-85-required)
  - [Application Skeleton Moved](#application-skeleton-moved)
  - [Service Providers Became Launchers](#service-providers-became-launchers)
  - [Path Helpers Renamed](#path-helpers-renamed)
  - [Attribute Imports Moved](#attribute-imports-moved)
  - [`#[CastToDate]` Was Removed](#casttodate-was-removed)
  - [HTTP Kernel Renamed to Gateway](#http-kernel-renamed-to-gateway)
  - [`vendor:publish` Flag Renamed](#vendorpublish-flag-renamed)
  - [Environment File Converted to TOML](#environment-file-converted-to-toml)
  - [Mail Transport Switched to Symfony Mailer](#mail-transport-switched-to-symfony-mailer)
  - [Checklist](#checklist)

## Upgrade Guide

This page is a focused, task-oriented walkthrough for moving an existing Doppar 3.x application to 4.x. It pulls the breaking changes out of the [4.x release notes](/versions/4.x/releases) and orders them the way you'll actually touch them in a real codebase — runtime first, then structure, then call sites.

Most of what's below is mechanical: a rename, a move, or a namespace swap that doesn't change how your application *behaves*, only where things live and what they're called. The one exception is the `.env` → `env.toml` switch — that one does change behavior, since environment values go from being untyped strings to real booleans, integers, and floats.

## Estimated Upgrade Time

**~30 minutes**, depending on how many providers, attribute imports, and custom mail drivers your project has. Budget more time if you have third-party packages that publish assets or config, since those need their own 4.x-compatible release.

## PHP 8.5 Required

Nothing else on this page matters until your runtime is on **PHP 8.5+**. Doppar 4.x has no compatibility shim for 8.3 or 8.4.

```bash
php -v
# PHP 8.5.0 (cli) or newer
```

Upgrade PHP first, in isolation, and confirm your existing 3.x app still boots before you touch any application code.

## Application Skeleton Moved

The project root was reorganized so that application code, views, and database resources each get their own top-level directory instead of sharing `app/` and `resources/`.

| Move | From | To |
| --- | --- | --- |
| Application source | `app/` | `src/` |
| Providers | `app/Providers/` | `src/Launchers/` |
| Views & language files | `resources/` | `templates/` |
| Migrations & seeders | `database/` | `schema/` (and `seeds/` → `seeders/`) |
| Config | `config/` | `runtime/config/` |
| Routes | `routes/` | `runtime/routes/` |
| Bootstrapping | `bootstrap/` | `runtime/app.php` |

Do this move as its own commit, before renaming anything inside the moved files — a pure `git mv` pass is easy to review, a combined move-and-rewrite is not. See [Directory Structure](/versions/4.x/directory-structure) for what belongs where.

## Service Providers Became Launchers

`Phaseolies\Providers\ServiceProvider` no longer exists. Every provider becomes a **launcher**, and `boot()` becomes `launch()`:

```php
// 3.x — src/Providers/AppServiceProvider.php (before the move)
namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    public function register(): void { /* ... */ }
    public function boot(): void { /* ... */ }
}
```

```php
// 4.x — src/Launchers/AppLauncher.php
namespace App\Launchers;

use Phaseolies\Launchers\ServiceLauncher;

class AppLauncher extends ServiceLauncher
{
    public function register(): void { /* ... */ }
    public function launch(): void { /* ... */ }
}
```

`register()` keeps doing exactly what it did before — container bindings only. `launch()` is where everything that depended on other services being ready now goes (routes, views, migrations, translations, console commands). Update the `launchers` array in `runtime/config/app.php` to point at the renamed classes once you're done.

If a provider only registers bindings and never touched `boot()`, look at whether it qualifies as a **ghostable launcher** — see [Launchers](/versions/4.x/launchers) for the lazy-registration contract.

## Path Helpers Renamed

Two globals were renamed to match the new directory names. There's no deprecated alias — a leftover call to either old name throws a fatal error under 4.x.

| 3.x | 4.x |
| --- | --- |
| `resource_path()` | `template_path()` |
| `database_path()` | `schema_path()` |

Grep your codebase for both before you consider this step done:

```bash
grep -rn "resource_path(\|database_path(" src/
```

## Attribute Imports Moved

Every attribute Doppar ships used to live under one flat namespace. In 4.x each attribute sits next to the subsystem it belongs to — same name, same behavior, different `use` line.

| Attribute | 3.x import | 4.x import |
| --- | --- | --- |
| `#[Bind]` | `Phaseolies\Utilities\Attributes\Bind` | `Phaseolies\DI\Attributes\Bind` |
| `#[Resolver]` | `Phaseolies\Utilities\Attributes\Resolver` | `Phaseolies\DI\Attributes\Resolver` |
| `#[Transaction]` | `Phaseolies\Utilities\Attributes\Transaction` | `Phaseolies\Database\Attributes\Transaction` |
| `#[Model]` | `Phaseolies\Utilities\Attributes\Model` | `Phaseolies\Database\Entity\Attributes\Model` |
| `#[BindPayload]` | `Phaseolies\Utilities\Attributes\BindPayload` | `Phaseolies\Http\Requests\Attributes\BindPayload` |
| `#[Middleware]` | `Phaseolies\Utilities\Attributes\Middleware` | `Phaseolies\Middleware\Attributes\Middleware` |
| `#[Route]` | `Phaseolies\Utilities\Attributes\Route` | `Phaseolies\Support\Router\Attributes\Route` |
| `#[Mapper]` | `Phaseolies\Utilities\Attributes\Mapper` | `Phaseolies\Support\Router\Attributes\Mapper` |
| `#[Throttle]` | `Phaseolies\Utilities\Attributes\Throttle` | `Phaseolies\Support\Router\Attributes\Throttle` |

This only touches the `use` statement at the top of each file — the attribute itself is unchanged, so a find-and-replace across your controllers, models, and middleware is sufficient.

## `#[CastToDate]` Was Removed

`Phaseolies\Utilities\Attributes\CastToDate` has no 4.x replacement — it wasn't moved, it was dropped. If a 3.x model used it to format `$timeStamps` as a date instead of a datetime, remove both the import and the attribute from that model before upgrading; there's nothing to swap it for.

## HTTP Kernel Renamed to Gateway

`App\Http\Kernel` is now `App\Http\Gateway`, and it must implement `Phaseolies\Http\Contracts\GatewayInterface`. This isn't just a rename — `Router` no longer extends class directly. It depends on the interface and resolves `Gateway` through the container, so the framework never references application code by name.

```php
// 3.x — src/Http/Kernel.php
namespace App\Http;

use Phaseolies\Middleware\Middleware;

class Kernel extends Middleware
{
    public array $middleware = [/* ... */];
    public $middlewareGroups = [/* ... */];
    public array $routeMiddleware = [/* ... */];
}
```

```php
// 4.x — src/Http/Gateway.php
namespace App\Http;

use Phaseolies\Middleware\Middleware;
use Phaseolies\Http\Contracts\GatewayInterface;

class Gateway extends Middleware implements GatewayInterface
{
    public array $middleware = [/* ... */];
    public $middlewareGroups = [/* ... */];
    public array $routeMiddleware = [/* ... */];

    public function getGlobalMiddleware(): array
    {
        return $this->middleware;
    }

    public function getMiddlewareGroups(): array
    {
        return $this->middlewareGroups;
    }

    public function getRouteMiddleware(): array
    {
        return $this->routeMiddleware;
    }
}
```

Rename the file and class, add the `implements GatewayInterface` clause, and add the three getter methods — your existing `$middleware`, `$middlewareGroups`, and `$routeMiddleware` arrays don't change. The framework finds your class by convention (`App\Http\Gateway`), so no additional wiring is required.

## `vendor:publish` Flag Renamed

Because publishable assets now live inside launchers, the identifying flag on `vendor:publish` changed:

```bash
# 3.x
php pool vendor:publish --provider="Vendor\PackageName\PackageServiceProvider"

# 4.x
php pool vendor:publish --launcher="Vendor\PackageName\PackageLauncher"
```

If you maintain a package for Doppar, this is a breaking change for your own consumers — update your install docs and any setup scripts that shell out to `vendor:publish`.

## Mail Transport Switched to Symfony Mailer

`phpmailer/phpmailer` is gone; Doppar 4.x sends mail through [Symfony Mailer](https://symfony.com/doc/current/mailer.html) instead. Your `Mailable` classes are unaffected — `subject()`, `content()`, and `attachment()` keep the same contract.

```json
// composer.json
// 3.x
"require": { "phpmailer/phpmailer": "^6.9" }

// 4.x
"require": { "symfony/mailer": "^8.1" }
```

Three things to check in `runtime/config/mail.php` and your codebase:

- The `qmail` and PHP `mail()` mailers are gone. Move to `smtp`, `sendmail`, or a DSN provider bridge — a single `MAILER_DSN` environment variable can now replace a whole block of host/port/credential settings.
- A custom `MailDriverInterface` implementation needs rewriting against Symfony's `TransportInterface`.
- CC/BCC recipients now genuinely reach the SMTP envelope. If a test or integration asserted on the old PHPMailer behavior, expect it to need updating.

See [Mail](/versions/4.x/mail) for the full `runtime/config/mail.php` reference.

## Environment File Converted to TOML

`.env`/`.env.example` are gone; Doppar 4.x reads `env.toml`/`env.toml.example` instead. This isn't just a rename — `.env` only ever produced strings (`APP_DEBUG=false` was the string `"false"`, and `(bool) "false"` is `true` in PHP), while `env.toml` gives you real types: `APP_DEBUG = false` is an actual boolean, `DB_PORT = 3306` an actual integer.

```bash
# 3.x — .env
APP_NAME="Doppar"
APP_ENV=local
APP_DEBUG=false
DB_PORT=3306
```

```toml
# 4.x — env.toml
APP_NAME = "Doppar"
APP_ENV = "local"
APP_DEBUG = false
DB_PORT = 3306
```

Converting means, for every line:

- Quote every string value — TOML doesn't accept a bare unquoted `local` the way `.env` does.
- Write booleans and numbers bare, not quoted — `APP_DEBUG = false`, not `APP_DEBUG = "false"`. The quoted form is a non-empty string, which PHP treats as truthy, silently defeating the whole point of the switch.
- Flatten any `${VAR}` interpolation (a `phpdotenv` feature) into a literal value — TOML has no variable substitution.
- Keep it flat. TOML supports `[section]` tables, but every `env()` call site expects a flat key — a nested table in `env.toml` throws rather than being silently misread.

Update `.gitignore` too: `.env` → `env.toml`, `!.env.example` → `!env.toml.example`.

If `env.toml` doesn't exist at all, Doppar boots fine and relies entirely on real OS/server environment variables — useful for containers that inject configuration directly.

## Checklist

Work through these in order — each step assumes the previous one is already done, and each is independently testable.

1. Upgrade the runtime to PHP 8.5+.
2. Move the skeleton: `app/` → `src/`, `database/` → `schema/`, `resources/` → `templates/`, `config/` → `runtime/config/`, `routes/` → `runtime/routes/`, fold `bootstrap/` into `runtime/app.php`.
3. Convert every provider to a launcher and update `runtime/config/app.php`.
4. Replace `resource_path()` / `database_path()` call sites.
5. Update attribute `use` imports and remove any `#[CastToDate]` usage.
6. Rename `src/Http/Kernel.php` to `Gateway.php`, implement `GatewayInterface`, and add the three getter methods.
7. Update `vendor:publish --provider=` to `--launcher=` anywhere it's referenced.
8. Convert `.env`/`.env.example` to `env.toml`/`env.toml.example` — quote strings, write booleans/numbers bare, flatten any `${VAR}` interpolation, and update `.gitignore`.
9. Swap the mail transport dependency and update `runtime/config/mail.php`.
10. Run the full test suite, paying particular attention to anything that asserted on mail delivery behavior.

> Do this on a dedicated branch. The steps are mechanical but touch nearly every file in the project — keeping it separate from feature work makes the diff reviewable.

For the full list of what's new in 4.x beyond these breaking changes, see the [Release Notes](/versions/4.x/releases).
