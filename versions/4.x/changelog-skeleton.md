---
title: Application Skeleton Changelog
description: Release history of Doppar Application Skeleton 4.x, generated from the doppar/doppar GitHub releases
meta:
  - name: keywords
    content: changelog, release history, doppar 4.x, application skeleton
---

## Changelog

Every tagged release of [`doppar/doppar`](https://github.com/doppar/doppar) in the 4.x line, newest first. Use the package selector above to switch between the framework and first-party packages. For the big picture of what changed between major versions, see the [Release Notes](/versions/4.x/releases).

## v4.2.0

_Released November 11, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/v4.2.0)

* readme updated by adding ripple by [@techmahedy](https://github.com/techmahedy) in [#67](https://github.com/doppar/doppar/pull/67)
* Add stateless and lottery options to the session configuration by [@techmahedy](https://github.com/techmahedy) in [#68](https://github.com/doppar/doppar/pull/68)

## v4.1.3

_Released October 11, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/v4.1.3)

* Drop pre-8.5 PDO MySQL constant fallbacks by [@techmahedy](https://github.com/techmahedy) in [#66](https://github.com/doppar/doppar/pull/66)

## v4.1.2

_Released September 26, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/v4.1.2)

* Delete composer.lock by [@techmahedy](https://github.com/techmahedy) in [#65](https://github.com/doppar/doppar/pull/65)

## v4.1.1

_Released September 26, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/v4.1.1)

* fix: added missing insight middleware by [@techmahedy](https://github.com/techmahedy) in [#63](https://github.com/doppar/doppar/pull/63)
* remove hardcoded insight profiler middleware code by [@techmahedy](https://github.com/techmahedy) in [#64](https://github.com/doppar/doppar/pull/64)

## v4.1.0

_Released September 25, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/v4.1.0)

* (feat) add insight package to skeleton by [@rrr63](https://github.com/rrr63) in [#61](https://github.com/doppar/doppar/pull/61)
* Implement Authable on User for multi-actor auth by [@sanzgrapher](https://github.com/sanzgrapher) in [#62](https://github.com/doppar/doppar/pull/62)

## 4.0.1

_Released September 20, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/4.0.1)

* Fix PHPUnit session driver to use supported file backend by [@xentixar](https://github.com/xentixar) in [#58](https://github.com/doppar/doppar/pull/58)
* update test version check to 4.0.0 by [@xentixar](https://github.com/xentixar) in [#54](https://github.com/doppar/doppar/pull/54)
* Update .gitignore to remove composer.lock and package-lock.json by [@xentixar](https://github.com/xentixar) in [#59](https://github.com/doppar/doppar/pull/59)
* Update .gitignore to include schema/database.sqlite by [@xentixar](https://github.com/xentixar) in [#60](https://github.com/doppar/doppar/pull/60)

## 4.0.0

_Released September 19, 2026_ · [View on GitHub](https://github.com/doppar/doppar/releases/tag/4.0.0)

Doppar 4.0 is a major release. It targets PHP 8.5 and Symfony 8.1, ships a new application skeleton, and hardens the framework's security surface. It also brings attribute-driven DTO validation, a redesigned error page, and `env.toml` configuration.

Because this is a major version, some changes are breaking. Read **Breaking changes** before upgrading a 3.x app.

**Highlights**
- **New application skeleton.** The folder layout is reorganised: `app/` → `src/`, `config/` → `runtime/config/`, `routes/` → `runtime/routes/`, `bootstrap/` → `runtime/`, `database/` → `schema/`.
- **`env.toml` replaces `.env`.** Environment configuration is now TOML-based. Copy `env.toml.example` to get started.
- **Symfony Mailer replaces PHPMailer.** `Mail` now runs on `symfony/mailer` with a new `config/mail.php`.
- **Attribute-based DTO validation.** Validate `#[BindPayload]` DTOs with constraint attributes: `#[NotBlank]`, `#[StringType]`, `#[Integer]`, `#[Length]` and `#[Between]`.
- **Redesigned error page** for development.
- **`ServiceProvider` is now `ServiceLauncher`.** Generate one with `php doppar make:launcher`.
- **A cleaner core.** `Router` depends on `GatewayInterface` instead of extending `App\Http\Kernel`, and the base path is injected through the `Application` constructor instead of a global `BASE_PATH` constant.
- **`vendor:publish` prompts** you to choose which launcher to publish.

**Security hardening**
- **Encryption moves to AES-256-GCM (AEAD).** Existing AES-256-CBC ciphertexts still decrypt. This release also fixes an `APP_KEY` decoding bug.
- **Path traversal** fixed in `Storage` and `Support\File`.
- **Uploads are checked by content.** The `mimes` validation rule now inspects real file content, so spoofed MIME types are rejected.
- **SQL identifier injection** fixed in the query builder.
- **Mass assignment** fixed on `update()`.
- **Session ID regenerates** after a successful login.

**Fixes**
- `Schema::table()` generated `CREATE TABLE` instead of `ALTER TABLE` on every driver.
- Fixed a DI resolution-order bug.
- `Model::__get()` no longer swallows `\Throwable`.
- Fixed `after_created` hook ordering.
- Fixed PostgreSQL enum type validation.
- Terminating callback dependencies are now resolved through the container.
- Fixed `base_path()` handling of absolute paths across platforms, which also fixes a crash in `Router::getControllerClasses()`.
- `response()->view()` now goes through the Odo rendering engine.
- Fixed 2FA QR code generation for `chillerlan/php-qrcode` v6.
- `Request::capture()` is now separate from superglobal reads.

**Breaking changes**
| Area | What changed |
|---|---|
| PHP | Requires PHP **^8.5**. |
| Skeleton | Folder layout changed (see Highlights). Update `composer.json` autoload and your paths. |
| Environment | `.env` → `env.toml`. |
| Providers | `ServiceProvider` → `ServiceLauncher`; `make:provider` → `make:launcher`. |
| Attributes | Framework attributes moved out of `Phaseolies\Utilities\Attributes` (see below). |
| Mail | PHPMailer removed; update `config/mail.php` for Symfony Mailer. |
| Casting | `#[CastToDate]` removed. Use `#[ToDate]`. |
| Base path | The global `BASE_PATH` constant is removed. |

**Attribute namespaces**

| Attribute | New namespace |
|---|---|
| `#[Route]`, `#[Mapper]`, `#[Throttle]` | `Phaseolies\Support\Router\Attributes` |
| `#[Middleware]` | `Phaseolies\Middleware\Attributes` |
| `#[Bind]`, `#[Resolver]` | `Phaseolies\DI\Attributes` |
| `#[Transaction]` | `Phaseolies\Database\Attributes` |
| `#[Model]` | `Phaseolies\Database\Entity\Attributes` |
| `#[BindPayload]` | `Phaseolies\Http\Requests\Attributes` |

**Ecosystem**
The first-party packages (Queue, Embeds, Guard, Flarion, Oauthic, Orion, Airbend, Bloom, Axios, Notifier, Insight, Twig-bridge) are released alongside the core.

**Upgrading**
Documentation: https://doppar.com/versions/4.x/installation
What's new: https://doppar.com/versions/4.x/releases

* merged with 3.x to master by [@techmahedy](https://github.com/techmahedy) in [#36](https://github.com/doppar/doppar/pull/36)
* Merge pull request #36 from doppar/3.x by [@techmahedy](https://github.com/techmahedy) in [#37](https://github.com/doppar/doppar/pull/37)
* 4.x: introduce 4.x skeleton: by [@techmahedy](https://github.com/techmahedy) in [#38](https://github.com/doppar/doppar/pull/38)
* Rename ServiceProvider to Launcher by [@techmahedy](https://github.com/techmahedy) in [#39](https://github.com/doppar/doppar/pull/39)
* Route attribute namespace updated in 4.x by [@techmahedy](https://github.com/techmahedy) in [#40](https://github.com/doppar/doppar/pull/40)
* Migrate Mailer from PHPMailer to Symfony Mailer (Doppar 4.x) by [@techmahedy](https://github.com/techmahedy) in [#41](https://github.com/doppar/doppar/pull/41)
* bump welcome page docs version: by [@techmahedy](https://github.com/techmahedy) in [#42](https://github.com/doppar/doppar/pull/42)
* runtime bootstrap and cleanup by [@xentixar](https://github.com/xentixar) in [#43](https://github.com/doppar/doppar/pull/43)
* update default test: by [@techmahedy](https://github.com/techmahedy) in [#44](https://github.com/doppar/doppar/pull/44)
* Fix SQLite file path in post-create-project command by [@xentixar](https://github.com/xentixar) in [#45](https://github.com/doppar/doppar/pull/45)
* refactor: Router depends on GatewayInterface instead of extending Kernel by [@techmahedy](https://github.com/techmahedy) in [#46](https://github.com/doppar/doppar/pull/46)
* Environment configuration switches from .env to env.toml by [@techmahedy](https://github.com/techmahedy) in [#47](https://github.com/doppar/doppar/pull/47)
* .env to env.toml git test action hook by [@techmahedy](https://github.com/techmahedy) in [#48](https://github.com/doppar/doppar/pull/48)
* minimize the comments from runtime/app.php by [@techmahedy](https://github.com/techmahedy) in [#49](https://github.com/doppar/doppar/pull/49)
* welcome page updated for 4.x by [@techmahedy](https://github.com/techmahedy) in [#50](https://github.com/doppar/doppar/pull/50)
* finalize the doppar 4.x version welcome page by [@techmahedy](https://github.com/techmahedy) in [#51](https://github.com/doppar/doppar/pull/51)
* fix: database seeder namespace path: by [@techmahedy](https://github.com/techmahedy) in [#52](https://github.com/doppar/doppar/pull/52)
* readme.md updated by [@techmahedy](https://github.com/techmahedy) in [#53](https://github.com/doppar/doppar/pull/53)
* Boot the app once for PHPUnit by [@xentixar](https://github.com/xentixar) in [#55](https://github.com/doppar/doppar/pull/55)
* updated README.md file by [@techmahedy](https://github.com/techmahedy) in [#56](https://github.com/doppar/doppar/pull/56)
* set miminum stability to stable in composer.json by [@techmahedy](https://github.com/techmahedy) in [#57](https://github.com/doppar/doppar/pull/57)
