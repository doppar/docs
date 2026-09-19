---
title: Release notes
description: Doppar 4.x release notes — everything that changed moving from Doppar 3.x to 4.x
meta:
  - name: keywords
    content: release-notes, doppar 4.x, launchers, service providers, symfony mailer, upgrade guide, migration
---

## Release Notes

Welcome to the official Doppar PHP Framework Release Notes — your central hub for updates, improvements, and innovations across every Doppar version.

Doppar will ship one major version every year on December 1st, with minor and patch updates released as needed throughout the year. Each release focuses on performance, developer experience, and modern PHP capabilities.

## Versioning Scheme

Doppar will follow a predictable annual release cycle:

* **Major Release:** Every **December 1st** (may include breaking changes)
* **Minor Releases:** Periodically, without breaking changes
* **Patch Releases:** Bug fixes and security updates

We use a version constraint strategy similar to `^4.0` to ensure stable upgrades across the ecosystem.

## Support Policy

Each major version of Doppar receives the following support:

* **Bug Fixes:** 18 months
* **Security Fixes:** 24 months

| Version | PHP Version | Release Date     | Bug Fixes Until | Security Fixes Until |
| ------- | ----------- | ---------------- | ---------------- | --------------------- |
| 3.0.0   | 8.3+        | December 1, 2025 | June 1, 2027      | December 1, 2027      |
| 4.0.0   | 8.5+        | September 19, 2026  | April 1, 2028     | October 1, 2028     |

## Doppar v4.0.0 Release Notes

Doppar `v4.0.0` is the most significant release since the framework's public debut. It raises the minimum PHP version to **8.5**, replaces the service provider system with a leaner **launcher** architecture, reorganizes every attribute namespace under a consistent structure, reshapes the application skeleton around `src/`, `schema/`, `templates/`, and `runtime/`, and swaps PHPMailer for **Symfony Mailer**.

None of this is cosmetic. Every project upgrading from 3.x will touch its providers, its path helpers, its attribute imports, and its directory layout. The upgrade guide at the bottom of this page walks through all of it in order.

> PHP 8.5 itself ships the pipe operator, first-class `Uri` objects, `clone()` with property overrides, and the `#[\NoDiscard]` attribute. Doppar 4.x's minimum version bump exists so the framework — and your application — can build on top of these once they're runtime-available, without a PHP 8.3/8.4 compatibility tax. See the [`PHP 8.5 release notes`](https://www.php.net/releases/8.5/en.php) for the full list.

### Highlights at a Glance

| Area | 3.x | 4.x |
| --- | --- | --- |
| Minimum PHP | 8.3+ | **8.5+** |
| Bootstrapping | Service Providers (`register()` / `boot()`) | **Launchers** (`register()` / `launch()`) |
| Provider generator | `php pool make:provider` | `php pool make:launcher` |
| Template path helper | `resource_path()` | `template_path()` |
| Database path helper | `database_path()` | `schema_path()` |
| Attribute namespaces | Flat under `Utilities\Attributes\*` | Split by concern (`DI`, `Database`, `Http`, `Middleware`, `Support\Router`, ...) |
| Date casting attribute | `#[CastToDate]` | Removed |
| HTTP middleware entry point | `App\Http\Kernel` (extended by `Router`) | **`App\Http\Gateway`**, via `GatewayInterface` |
| Package publishing | `vendor:publish --provider=` | `vendor:publish --launcher=` |
| Mail transport | PHPMailer | **Symfony Mailer** |
| DTO validation | Not available | **Attribute-based, via `#[BindPayload(validate: true)]`** |
| Semantic search | Not available | **`doppar/embeds`** — `#[Embeds]` attribute, `brute_force`/`pgvector`/`redis` drivers |
| App source root | `app/` | `src/` |
| Views & lang | `resources/` | `templates/` |
| Migrations & seeders | `database/` | `schema/` |
| Environment file | `.env` (untyped strings) | **`env.toml`** (typed values) |

### Minimum PHP Version Raised to 8.5

Doppar 3.x supported PHP 8.3+. Doppar 4.x requires **PHP 8.5 or higher** — there is no compatibility mode for older PHP versions.

> Upgrade your PHP runtime **before** touching your application code. Every change below assumes an 8.5 environment.

### Service Providers Are Now Launchers

The single biggest architectural change in 4.x: `Phaseolies\Providers\ServiceProvider` is gone, replaced by `Phaseolies\Launchers\ServiceLauncher`. The two lifecycle methods are renamed to match — `boot()` becomes `launch()` — and the class itself moves from `app/Providers` to `src/Launchers`.

**3.x — Service Provider**

```php
<?php

namespace App\Providers;

use Phaseolies\Providers\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register(): void
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot(): void
    {
        //
    }
}
```

**4.x — Launcher**

```php
<?php

namespace App\Launchers;

use Phaseolies\Launchers\ServiceLauncher;

class AppLauncher extends ServiceLauncher
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register(): void
    {
        //
    }

    /**
     * Launch any application services.
     *
     * @return void
     */
    public function launch(): void
    {
        //
    }
}
```

The console generator follows the same rename:

| | 3.x | 4.x |
| --- | --- | --- |
| Generate | `php pool make:provider MyServiceProvider` | `php pool make:launcher MyLauncher` |

`register()` keeps its meaning unchanged — container bindings and raw service wiring. `launch()` takes over what `boot()` used to do — anything that depends on other services already being registered (loading routes, views, migrations, translations, console commands).

Doppar 4.x also introduces **ghostable launchers**, which can be lazily registered the first time one of their declared services is actually resolved, instead of eagerly on every request. See the [Launchers](/versions/4.x/launchers) documentation for the full lifecycle and the `GhostableLauncher` contract.

### Path Helper Functions Renamed

Two global path helpers were renamed to match the new directory structure:

| 3.x | 4.x | Points to |
| --- | --- | --- |
| `resource_path()` | `template_path()` | `templates/` |
| `database_path()` | `schema_path()` | `schema/` |

```php
// 3.x
$view = resource_path('views/layouts/app.Odo.php');
$migrations = database_path('migrations/');

// 4.x
$view = template_path('views/layouts/app.Odo.php');
$migrations = schema_path('migrations/');
```

Search your codebase for every call site — these are plain function renames with no deprecated alias, so a leftover `resource_path()` or `database_path()` call will throw a fatal error under 4.x.

### Attribute Namespaces Reorganized

In 3.x, every PHP attribute Doppar ships lived under the same flat `Phaseolies\Utilities\Attributes` namespace regardless of what it did. In 4.x, attributes moved next to the subsystem they belong to:

| Attribute | 3.x namespace | 4.x namespace |
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

Every one of these is a namespace-only move — the attribute's name, signature, and behavior are unchanged. Update the `use` statement at the top of each controller, model, and middleware class; nothing else needs to change.

```php
// 3.x
use Phaseolies\Utilities\Attributes\Route;
use Phaseolies\Utilities\Attributes\Middleware;

// 4.x
use Phaseolies\Support\Router\Attributes\Route;
use Phaseolies\Middleware\Attributes\Middleware;
```

#### The `#[CastToDate]` Attribute Was Removed

`Phaseolies\Utilities\Attributes\CastToDate` no longer exists in 4.x — it was not moved, it was removed outright. If your 3.x models used it to format the `$timeStamps` property as a date instead of a datetime, that attribute has no direct 4.x replacement; remove the import and the `#[CastToDate]` line from every model that used it before upgrading.

### `vendor:publish` Now Targets Launchers

Because publishable resources are declared inside launchers rather than service providers, the `vendor:publish` command's identifying flag changed from `--provider` to `--launcher`:

```bash
# 3.x
php pool vendor:publish --provider="Vendor\PackageName\PackageServiceProvider"

# 4.x
php pool vendor:publish --launcher="Vendor\PackageName\PackageLauncher"
```

If you maintain a package for Doppar, update your `README` and any install scripts that reference the old flag — third-party consumers running 4.x will get an unrecognized-option error otherwise.

### HTTP Kernel Renamed to Gateway, and `Router` No Longer Extends It

`App\Http\Kernel` is now `App\Http\Gateway`. This is more than a rename: in 3.x, `Phaseolies\Support\Router` extended application's `Kernel` class directly. In 4.x, `Router` depends on a new `Phaseolies\Http\Contracts\GatewayInterface` and resolves `Gateway` through the container instead.

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

Your `$middleware`, `$middlewareGroups`, and `$routeMiddleware` arrays keep the exact same shape — only the file name, class name, and the interface implementation are new. The framework still finds your `Gateway` by convention (`App\Http\Gateway`), so no manual container binding is required in the common case.

### Directory Structure Overhaul

The application skeleton was restructured end to end. The application's own code now lives under `src/`, views and language files under `templates/`, database resources under `schema/`, and configuration/routes/environment bootstrapping consolidated under `runtime/`. `bootstrap/` is gone — its job is now done by `runtime/app.php`.

| 3.x directory | 4.x directory | Notes |
| --- | --- | --- |
| `app/` | `src/` | Application source root |
| `app/Providers/` | `src/Launchers/` | Providers → Launchers |
| `app/Http/` | `src/Http/` | Controllers, Middleware, Gateway, Exceptions |
| `app/Models/` | `src/Models/` | Unchanged in purpose |
| `app/Schedule/` | `src/Schedule/` | Unchanged in purpose |
| `config/` | `runtime/config/` | Config files, plus new `odo.php` |
| `routes/` | `runtime/routes/` | `api.php` / `web.php` |
| `bootstrap/` | `runtime/app.php` | Framework bootstrapping consolidated into one file |
| `database/migrations/` | `schema/migrations/` | |
| `database/seeds/` | `schema/seeders/` | Also renamed `seeds/` → `seeders/` |
| `resources/views/` | `templates/views/` | |
| `lang/` | `templates/lang/` | Moved under `templates/` |
| `storage/` | `storage/` | Unchanged |
| `public/` | `public/` | Unchanged |
| `tests/` | `tests/` | Now ships `BootstrapApplication.php` alongside `TestCase.php` |

**3.x skeleton**

```markdown
└── 📁doppar-framework
    └── 📁app
        └── 📁Http
            └── 📁Controllers
            └── Kernel.php
            └── 📁Middleware
        └── 📁Models
        └── 📁Providers
        └── 📁Schedule
            └── Schedule.php
    └── 📁bootstrap
    └── 📁config
    └── 📁lang
    └── 📁database
        └── 📁migrations
        └── 📁seeds
    └── 📁public
    └── 📁resources
        └── 📁views
    └── 📁routes
        └── api.php
        └── web.php
    └── 📁storage
        └── 📁app
            └── 📁public
        └── 📁cache
        └── 📁logs
        └── 📁sessions
    └── 📁tests
    └── .env
    └── .env.example
    └── .gitignore
    └── composer.json
    └── composer.lock
    └── pool
    └── server.php
```

**4.x skeleton**

```markdown
└── 📁skeleton
    └── 📁public
        ├── .htaccess
        ├── favicon.ico
        ├── index.php
        ├── logo.png
        ├── robots.txt
    └── 📁runtime
        └── 📁config
            ├── app.php
            ├── auth.php
            ├── caching.php
            ├── database.php
            ├── filesystem.php
            ├── hashing.php
            ├── log.php
            ├── mail.php
            ├── odo.php
            ├── session.php
        └── 📁routes
            ├── api.php
            ├── web.php
        ├── app.php
    └── 📁schema
        └── 📁migrations
            ├── 2025_04_17_143030_create_users_table.php
        └── 📁seeders
            ├── DatabaseSeeder.php
            ├── UserSeeder.php
    └── 📁src
        └── 📁Http
            └── 📁Controllers
                ├── Controller.php
                ├── WelcomeController.php
            └── 📁Exceptions
                ├── BeforeExceptionHandler.php
            └── 📁Middleware
                ├── Authenticate.php
                ├── GuestMiddleware.php
                ├── TrustProxies.php
                ├── VerifyTwoFactorUser.php
            ├── Gateway.php
        └── 📁Launchers
            ├── AppLauncher.php
        └── 📁Models
            ├── User.php
        └── 📁Schedule
            ├── Schedule.php
    └── 📁templates
        └── 📁lang
            └── 📁en
                ├── messages.php
                ├── validation.php
        └── 📁views
            ├── welcome.odo.php
    └── 📁tests
        └── 📁Unit
            ├── ExampleTest.php
        ├── BootstrapApplication.php
        ├── TestCase.php
    ├── .editorconfig
    ├── env.toml
    ├── env.toml.example
    ├── .gitattributes
    ├── .gitignore
    ├── .styleci.yml
    ├── composer.json
    ├── composer.lock
    ├── pool
    ├── README.md
    └── server.php
```

See the [Directory Structure](/versions/4.x/directory-structure) page for a full breakdown of what belongs in each folder.

### Environment Configuration Moves From `.env` to `env.toml`

`.env` files only ever produce strings — `APP_DEBUG=false` is the four-character string `"false"`, not a boolean, and `(bool) "false"` is `true` in PHP. 4.x replaces `.env`/`.env.example` with **`env.toml`**/**`env.toml.example`**, so environment values keep their real type all the way through `env()`.

```toml
# env.toml
APP_NAME = "Doppar"
APP_ENV = "local"
APP_DEBUG = false
APP_URL = "http://localhost:8000"

DB_CONNECTION = "mysql"
DB_PORT = 3306
```

`env('APP_DEBUG')` now returns an actual `bool`, `env('DB_PORT')` an actual `int` — no more `(bool) env(...)` casts that quietly misfire on the string `"false"`.

A few things carry over differently from `.env`:

- **Every value needs a type.** Strings must be quoted (`APP_ENV = "local"`, not `APP_ENV = local`); booleans and numbers are written bare (`APP_DEBUG = false`, `DB_PORT = 3306`).
- **`env.toml` must stay flat.** TOML supports `[section]` tables, but every `env()` call site expects a flat key — Doppar rejects a nested table in `env.toml` with a clear error rather than silently ignoring it.
- **No variable interpolation.** `.env`'s `MAIL_FROM_NAME="${APP_NAME}"` has no TOML equivalent — write the literal value instead.
- **A real OS/server environment variable still wins.** If `APP_DEBUG` is already set at the process level (Docker, systemd, your host's env), `env.toml`'s value for that key is skipped, same as `.env` always worked.

If `env.toml` is missing entirely, Doppar boots fine and falls back to whatever real environment variables are already set — useful for containers that inject configuration directly instead of shipping a file.

### Symfony Mailer Replaces PHPMailer

Doppar 4.x drops `phpmailer/phpmailer` in favor of [Symfony Mailer](https://symfony.com/doc/current/mailer.html). If you're coming from a 3.x project built on PHPMailer, here's exactly what does and doesn't change:

- **Mailables don't change.** `subject()`, `content()`, `attachment()` — same contract, same `make:mail` stub.
- **`runtime/config/mail.php` needs updating.** The `qmail` and `mail` (PHP `mail()`) mailers are gone — Symfony Mailer doesn't support either. Move to `smtp`, `sendmail`, or a DSN-based provider bridge.
- **Custom drivers need rewriting.** If you had a class implementing `MailDriverInterface`, replace it with a `TransportInterface` (Symfony's own contract) passed to `Mail::to(...)->driver(...)`.
- **CC/BCC now actually deliver.** If you were relying on the old behavior where CC/BCC recipients received headers but weren't always handed to the SMTP server as real envelope recipients, double check your downstream expectations — they now genuinely receive it.

Update your `composer.json`:

```json
// 3.x
"require": {
    "phpmailer/phpmailer": "^6.9"
}

// 4.x
"require": {
    "symfony/mailer": "^8.1"
}
```

Because Symfony Mailer is DSN-driven, a single `MAILER_DSN` environment variable can now replace a whole block of host/port/username/password settings:

```env
MAILER_DSN=smtp://user:pass@smtp.mailgun.org:587
```

See the [Mail](/versions/4.x/mail) documentation for the complete `runtime/config/mail.php` reference, including `failover` and `roundrobin` transport composition.

### Attribute-Based DTO Validation

Doppar 4.x adds validation constraints you can declare directly on DTO properties with PHP attributes, keeping the expected shape of a request payload next to the object that receives it.

```php
<?php

namespace App\DTO;

use Phaseolies\Validation\Attributes\Between;
use Phaseolies\Validation\Attributes\Integer;
use Phaseolies\Validation\Attributes\Length;
use Phaseolies\Validation\Attributes\NotBlank;
use Phaseolies\Validation\Attributes\StringType;

final class BookData
{
    #[NotBlank]
    #[StringType]
    #[Length(max: 255)]
    public string $title;

    #[NotBlank]
    #[Integer]
    #[Between(min: 1, max: 5)]
    public int $rating;
}
```

Set `validate: true` alongside `#[BindPayload]` to validate the incoming payload before the DTO is hydrated and handed to the controller:

```php
#[Route(uri: 'books', methods: ['POST'])]
public function store(
    #[BindPayload(strict: true, validate: true)]
    BookData $book,
) {
    Book::create($book->toArray());

    return redirect('/books');
}
```

Built-in constraints include `#[NotBlank]`, `#[StringType]`, `#[Integer]`, `#[Length]`, and `#[Between]`. See the [Attribute-Based DTO Validation](/versions/4.x/validation#attribute-based-dto-validation) documentation for the full constraint reference and the [Requests](/versions/4.x/requests#attribute-based-dto-binding-in-controllers) page for `#[BindPayload]` details.

### Semantic Search via `doppar/embeds`

Doppar 4.x introduces [`doppar/embeds`](/versions/4.x/doppar-embeds), a new package that adds attribute-driven semantic search to your models. Mark a column with `#[Embeds]`, and Doppar keeps a numeric representation of its meaning — a vector — in sync automatically, so `Product::whereSimilarTo('description', 'a durable waterproof backpack')` finds "rugged daypack for hiking in the rain" even though the two share almost no words.

```php
use Phaseolies\Database\Entity\Model;
use Doppar\Embeds\Attributes\Embeds;
use Doppar\Embeds\Concerns\Embeddable;

class Product extends Model
{
    use Embeddable;

    #[Embeds]
    protected $description;
}

$results = Product::whereSimilarTo('description', 'a durable waterproof backpack', limit: 10);
```

Embeddings are computed locally via `doppar/ai` — no external API call, no API key, no per-request cost. Vector storage and ranking is handled by a swappable driver: the zero-setup `brute_force` driver works on every database Doppar supports, while `pgvector` and `redis` push ranking into a real HNSW index for large tables. See the [Doppar Embeds](/versions/4.x/doppar-embeds) documentation for installation, driver configuration, and the full API reference.

### Upgrading From 3.x to 4.x

Work through these in order — each step assumes the previous one is done.

1. **Upgrade your PHP runtime to 8.5+.** Nothing below this line will run otherwise.
2. **Move the application skeleton**: `app/` → `src/`, `database/` → `schema/` (`seeds/` → `seeders/`), `resources/` → `templates/`, `config/` → `runtime/config/`, `routes/` → `runtime/routes/`, and fold `bootstrap/` into `runtime/app.php`.
3. **Convert every Service Provider to a Launcher**: move `app/Providers/*ServiceProvider.php` to `src/Launchers/*Launcher.php`, extend `Phaseolies\Launchers\ServiceLauncher` instead of `Phaseolies\Providers\ServiceProvider`, and rename `boot()` to `launch()`. Update the `launchers` array in `runtime/config/app.php` to point at the new classes.
4. **Replace path helpers**: `resource_path()` → `template_path()`, `database_path()` → `schema_path()`, everywhere they're called.
5. **Update attribute imports** to their new namespaces (see the table above), and remove any remaining `#[CastToDate]` usage.
6. **Update package-publishing flags**: any script or documentation using `vendor:publish --provider=` needs `--launcher=` instead.
7. **Convert `.env` to `env.toml`**: quote every string value, write booleans and numbers bare (`APP_DEBUG = false`, `DB_PORT = 3306`), flatten out any `${VAR}` interpolation into a literal value, and update `.gitignore` (`.env` → `env.toml`, `!.env.example` → `!env.toml.example`).
8. **Swap your mail transport**: remove `phpmailer/phpmailer`, add `symfony/mailer: ^8.1`, update `runtime/config/mail.php` off of `qmail`/`mail`, and rewrite any custom `MailDriverInterface` implementation against Symfony's `TransportInterface`.
9. **Run your test suite.** CC/BCC delivery behavior changed under the hood — if anything asserted on the old PHPMailer envelope behavior, expect it to need updating.

> Do this on a dedicated branch. The provider-to-launcher rename and the directory move are both mechanical but touch every file in the project — a clean diff makes review far easier than a mixed refactor-plus-feature branch.

## Doppar v3.0.0 Release Notes (December 1, 2025)

Doppar `v3.0.0` is the first official public release of the Doppar PHP Framework. This foundation release introduces a modern, performance-focused architecture built with PHP 8.3+, featuring attribute-powered routing, the ODO templating engine, Entity ORM, Doppar AI, and a daemon based native task runtime designed for real-time applications.

This release sets the tone for Doppar's philosophy: clarity, precision, and performance-first engineering.
