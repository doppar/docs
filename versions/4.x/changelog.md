---
title: Framework Changelog
description: Release history of Doppar Framework 4.x, generated from the doppar/framework GitHub releases
meta:
  - name: keywords
    content: changelog, release history, doppar 4.x, framework
---

## Changelog

Every tagged release of [`doppar/framework`](https://github.com/doppar/framework) in the 4.x line, newest first. Use the package selector above to switch between the framework and first-party packages. For the big picture of what changed between major versions, see the [Release Notes](/versions/4.x/releases).

## v4.9.1

_Released November 13, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.9.1)

* perf: optimize request routing and dependency resolution by [@techmahedy](https://github.com/techmahedy) in [#403](https://github.com/doppar/framework/pull/403)
* fix: align Carbon timezone and datetime precision handling by [@techmahedy](https://github.com/techmahedy) in [#404](https://github.com/doppar/framework/pull/404)
* fix: complete datetime boundary and microsecond consistency by [@techmahedy](https://github.com/techmahedy) in [#405](https://github.com/doppar/framework/pull/405)
* fix(session): follow-ups to the database session driver by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#406](https://github.com/doppar/framework/pull/406)

## v4.9.0

_Released November 11, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.9.0)

* feat:  Rewrite sessions as a lazy, store-based layer by [@techmahedy](https://github.com/techmahedy) in [#398](https://github.com/doppar/framework/pull/398)
* fix(http): make request/response handling consistent end to end by [@techmahedy](https://github.com/techmahedy) in [#399](https://github.com/doppar/framework/pull/399)
* fix(security): harden redirects, error views and debug output by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#401](https://github.com/doppar/framework/pull/401)
* fix: correct old input, validation rules, route lookups and error responses by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#402](https://github.com/doppar/framework/pull/402)
* feat(session): add a database session driver and logout from other devices by [@techmahedy](https://github.com/techmahedy) in [#400](https://github.com/doppar/framework/pull/400)

## v4.8.4

_Released October 31, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.8.4)

* Fix vendor:publish dumping all assets for unknown tags by [@xentixar](https://github.com/xentixar) in [#396](https://github.com/doppar/framework/pull/396)

## v4.8.3

_Released October 31, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.8.3)

* Session Interface Contract instead of concrete Session class by [@techmahedy](https://github.com/techmahedy) in [#381](https://github.com/doppar/framework/pull/381)
* Add a Contract for Password Hashing Services by [@techmahedy](https://github.com/techmahedy) in [#382](https://github.com/doppar/framework/pull/382)
* Add a Contract for Encryption Services by [@techmahedy](https://github.com/techmahedy) in [#383](https://github.com/doppar/framework/pull/383)
* Add a Contract for Request Abortion Services by [@techmahedy](https://github.com/techmahedy) in [#384](https://github.com/doppar/framework/pull/384)
* Add a Contract for Mail Services by [@techmahedy](https://github.com/techmahedy) in [#385](https://github.com/doppar/framework/pull/385)
* Add a Contract for Storage Services by [@techmahedy](https://github.com/techmahedy) in [#386](https://github.com/doppar/framework/pull/386)
* Add a Contract for URL Generation Services by [@techmahedy](https://github.com/techmahedy) in [#387](https://github.com/doppar/framework/pull/387)
* Address review: accept mixed in isValid, resolve url() via contract, reset builder state by [@techmahedy](https://github.com/techmahedy) in [#388](https://github.com/doppar/framework/pull/388)
* Introduce a string service contract by [@techmahedy](https://github.com/techmahedy) in [#389](https://github.com/doppar/framework/pull/389)
* Introduce a signal bus contract by [@techmahedy](https://github.com/techmahedy) in [#390](https://github.com/doppar/framework/pull/390)
* Fix save() leaving the model dirty after a quiet save by [@techmahedy](https://github.com/techmahedy) in [#391](https://github.com/doppar/framework/pull/391)
* Fix save() leaving the model dirty after an insert by [@techmahedy](https://github.com/techmahedy) in [#392](https://github.com/doppar/framework/pull/392)
* Fix StringService edge cases: mask, isPalindrome, random, truncate and more by [@techmahedy](https://github.com/techmahedy) in [#393](https://github.com/doppar/framework/pull/393)
* Fix validator regex alternation, array input 500s and duplicated errors by [@techmahedy](https://github.com/techmahedy) in [#394](https://github.com/doppar/framework/pull/394)
* Fix whereIn/orWhereIn with empty lists, qualify OR columns, add whereNotIn to the query builder by [@techmahedy](https://github.com/techmahedy) in [#395](https://github.com/doppar/framework/pull/395)

## v4.8.2

_Released October 26, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.8.2)

* Honor overridden query() on instance calls to the model by [@techmahedy](https://github.com/techmahedy) in [#380](https://github.com/doppar/framework/pull/380)

## v4.8.1

_Released October 19, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.8.1)

* Fix SQL injection via unvalidated table, view and column identifiers by [@techmahedy](https://github.com/techmahedy) in [#379](https://github.com/doppar/framework/pull/379)

## v4.8.0

_Released October 18, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.8.0)

* Optional relationship keys and typed relation classes by [@techmahedy](https://github.com/techmahedy) in [#377](https://github.com/doppar/framework/pull/377)
* (feat) Create and save records through relationships by [@techmahedy](https://github.com/techmahedy) in [#378](https://github.com/doppar/framework/pull/378)

## v4.7.1

_Released October 17, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.7.1)

* Fix temporal lookups that bring back deleted records by [@techmahedy](https://github.com/techmahedy) in [#372](https://github.com/doppar/framework/pull/372)
* Fix: Temporal diff() incorrectly compares cast values as strings by [@techmahedy](https://github.com/techmahedy) in [#373](https://github.com/doppar/framework/pull/373)
* Fix: Temporal orWhere() filtering accepts every model by [@techmahedy](https://github.com/techmahedy) in [#374](https://github.com/doppar/framework/pull/374)
* Fix: Suspended child contexts survive after their parent ends by [@techmahedy](https://github.com/techmahedy) in [#375](https://github.com/doppar/framework/pull/375)
* Fix: Context-ending listener failures can mask the original application failure by [@techmahedy](https://github.com/techmahedy) in [#376](https://github.com/doppar/framework/pull/376)
* (feat) Branchable temporal records with explainable history by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#367](https://github.com/doppar/framework/pull/367)

## v4.7.0

_Released October 17, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.7.0)

* Fix withoutHook() and add withHooks() for bulk queries by [@techmahedy](https://github.com/techmahedy) in [#369](https://github.com/doppar/framework/pull/369)
* Freeze #[Immutable] services registered with a closure or instance() by [@techmahedy](https://github.com/techmahedy) in [#370](https://github.com/doppar/framework/pull/370)
* Clearer errors for frozen services by [@techmahedy](https://github.com/techmahedy) in [#371](https://github.com/doppar/framework/pull/371)

## v4.6.0

_Released October 16, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.6.0)

* (feat) Application: Go-style defer() for guaranteed cleanup by [@techmahedy](https://github.com/techmahedy) in [#365](https://github.com/doppar/framework/pull/365)
* (feat) deliver domain notifications only after a transaction commits by [@techmahedy](https://github.com/techmahedy) in [#366](https://github.com/doppar/framework/pull/366)
* (feat) Request-scoped lifecycle and context propagation by [@techmahedy](https://github.com/techmahedy) in [#368](https://github.com/doppar/framework/pull/368)

## v4.5.2

_Released October 12, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.5.2)

* Fix: Encryption ignores APP_KEY from env.toml and silently uses a public test key by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#364](https://github.com/doppar/framework/pull/364)

## v4.5.1

_Released October 11, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.5.1)

* Drop pre-8.5 PDO MySQL constant fallbacks and tidy migrate:temporal output by [@techmahedy](https://github.com/techmahedy) in [#356](https://github.com/doppar/framework/pull/356)

## v4.5.0

_Released October 9, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.5.0)

* Fix: query builder treats column names like date/time as callbacks by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#353](https://github.com/doppar/framework/pull/353)
* (feat) ORM: soft deletes via #[SoftDeletes] by [@abdulmajidcse](https://github.com/abdulmajidcse) in [#354](https://github.com/doppar/framework/pull/354)
* Fix callback conditions in present() ifExists() being dropped or rewritten and add whereNotIn() to the Entity builder by [@techmahedy](https://github.com/techmahedy) in [#355](https://github.com/doppar/framework/pull/355)

## v4.4.0

_Released October 3, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.4.0)

* server:start: Doppar banner and per-request log with execution time by [@techmahedy](https://github.com/techmahedy) in [#346](https://github.com/doppar/framework/pull/346)
* (feat) Container: using() for temporary, self-restoring overrides by [@techmahedy](https://github.com/techmahedy) in [#348](https://github.com/doppar/framework/pull/348)
* Migrations: rollback, reset, refresh, status, db:wipe, pretend, locking and transactional runs by [@techmahedy](https://github.com/techmahedy) in [#347](https://github.com/doppar/framework/pull/347)
* Collection: bug fixes, dot notation, multi-key sorting, 21 new methods, higher-order proxies by [@techmahedy](https://github.com/techmahedy) in [#352](https://github.com/doppar/framework/pull/352)
* Cache: rate limiting fixes, Redis/prefix safety, Cache::store(), Cache::tags() by [@techmahedy](https://github.com/techmahedy) in [#351](https://github.com/doppar/framework/pull/351)
* Pool console: full signature syntax, original exceptions, and resilient command discovery by [@techmahedy](https://github.com/techmahedy) in [#350](https://github.com/doppar/framework/pull/350)
* (feat) Routing: route parameter constraints by [@techmahedy](https://github.com/techmahedy) in [#349](https://github.com/doppar/framework/pull/349)

## v4.3.0

_Released October 2, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.3.0)

* Refactor big-data processing trait, add chunkById(), remove Fiber variants by [@techmahedy](https://github.com/techmahedy) in [#343](https://github.com/doppar/framework/pull/343)
* (feat) Add blank, filled, with, rescue, retry and benchmark helpers by [@techmahedy](https://github.com/techmahedy) in [#344](https://github.com/doppar/framework/pull/344)
* Migration builder: table-level indexes, column modifiers, drop/rename/change, and fixes by [@techmahedy](https://github.com/techmahedy) in [#345](https://github.com/doppar/framework/pull/345)

## v4.2.0

_Released September 28, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.2.0)

* Make PresenterBundle directly returnable as a JSON API resource by [@techmahedy](https://github.com/techmahedy) in [#342](https://github.com/doppar/framework/pull/342)
* (feat) Router: compile controller actions into cached "action plans" instead of using Reflection on every request by [@techmahedy](https://github.com/techmahedy) in [#341](https://github.com/doppar/framework/pull/341)

## v4.1.1

_Released September 26, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.1.1)

* Pull request template file change and use latest version in action file by [@techmahedy](https://github.com/techmahedy) in [#336](https://github.com/doppar/framework/pull/336)
* Add Router::pushGlobalMiddleware() so launchers can register global middleware by [@techmahedy](https://github.com/techmahedy) in [#337](https://github.com/doppar/framework/pull/337)
* remove ErrorHistoryRecorder class from skeleton by adding it in framework by [@techmahedy](https://github.com/techmahedy) in [#338](https://github.com/doppar/framework/pull/338)
* Fix Paginator prev/next links leaking merged request input into URLs by [@techmahedy](https://github.com/techmahedy) in [#339](https://github.com/doppar/framework/pull/339)
* Fix cron:run and the scheduler: non-blocking background tasks by [@techmahedy](https://github.com/techmahedy) in [#340](https://github.com/doppar/framework/pull/340)

## v4.1.0

_Released September 25, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.1.0)

* Fix: Request State Isolation in Persistent Workers by [@techmahedy](https://github.com/techmahedy) in [#330](https://github.com/doppar/framework/pull/330)
* fix: correct facade @ method docblocks so PHPStan passes by [@techmahedy](https://github.com/techmahedy) in [#334](https://github.com/doppar/framework/pull/334)
* Extract auth into Authable base class by [@sanzgrapher](https://github.com/sanzgrapher) in [#333](https://github.com/doppar/framework/pull/333)

## v4.0.1

_Released September 20, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.0.1)

* fix: error page paths on Windows by [@rrr63](https://github.com/rrr63) in [#329](https://github.com/doppar/framework/pull/329)

## v4.0.0

_Released September 19, 2026_ · [View on GitHub](https://github.com/doppar/framework/releases/tag/v4.0.0)

* merged with master by [@techmahedy](https://github.com/techmahedy) in [#290](https://github.com/doppar/framework/pull/290)
* merged with master (#290) by [@techmahedy](https://github.com/techmahedy) in [#291](https://github.com/doppar/framework/pull/291)
* Introduce Doppar 4.x Application Skeleton by [@techmahedy](https://github.com/techmahedy) in [#289](https://github.com/doppar/framework/pull/289)
* Rename ServiceProvider to ServiceLauncher by [@techmahedy](https://github.com/techmahedy) in [#292](https://github.com/doppar/framework/pull/292)
* Redesign Error Page for 4.x Version by [@techmahedy](https://github.com/techmahedy) in [#293](https://github.com/doppar/framework/pull/293)
* Remove CastToDate for 4.x by [@techmahedy](https://github.com/techmahedy) in [#294](https://github.com/doppar/framework/pull/294)
* Relocate Framework Attributes for 4.x by [@techmahedy](https://github.com/techmahedy) in [#295](https://github.com/doppar/framework/pull/295)
* Prompt for launcher selection in vendor:publish by [@techmahedy](https://github.com/techmahedy) in [#297](https://github.com/doppar/framework/pull/297)
* Migrate Mailer from PHPMailer to Symfony Mailer (Doppar 4.x) by [@techmahedy](https://github.com/techmahedy) in [#298](https://github.com/doppar/framework/pull/298)
* session regenerate after successfull login: by [@techmahedy](https://github.com/techmahedy) in [#299](https://github.com/doppar/framework/pull/299)
* fix: mass-assignment on update: by [@techmahedy](https://github.com/techmahedy) in [#300](https://github.com/doppar/framework/pull/300)
* fix: SQL identifier injection by [@techmahedy](https://github.com/techmahedy) in [#302](https://github.com/doppar/framework/pull/302)
* fix: Route response()->view() through the Odo rendering engine by [@rrr63](https://github.com/rrr63) in [#301](https://github.com/doppar/framework/pull/301)
* fix : cross-platform absolute path handling in base_path() by [@rrr63](https://github.com/rrr63) in [#303](https://github.com/doppar/framework/pull/303)
* feat: add attribute-based DTO validation by [@rrr63](https://github.com/rrr63) in [#304](https://github.com/doppar/framework/pull/304)
* fix: base_path() ambiguity crashes Router::getControllerClasses() on getPrefixesPsr4() by [@techmahedy](https://github.com/techmahedy) in [#305](https://github.com/doppar/framework/pull/305)
* Fix #4 done: path traversal in Storage by [@techmahedy](https://github.com/techmahedy) in [#306](https://github.com/doppar/framework/pull/306)
* Fix #5 encryption — APP_KEY decode bug + move to AEAD by [@techmahedy](https://github.com/techmahedy) in [#307](https://github.com/doppar/framework/pull/307)
* Fix #9 after_created hook ordering by [@techmahedy](https://github.com/techmahedy) in [#308](https://github.com/doppar/framework/pull/308)
* Fix Model::__get() swallowing \Throwable by [@techmahedy](https://github.com/techmahedy) in [#309](https://github.com/doppar/framework/pull/309)
* resolve terminating callback dependencies via container by [@techmahedy](https://github.com/techmahedy) in [#310](https://github.com/doppar/framework/pull/310)
* fix: enforce enum column values on Postgres and SQLite by [@techmahedy](https://github.com/techmahedy) in [#311](https://github.com/doppar/framework/pull/311)
* fix: Schema::table() generated CREATE TABLE instead of ALTER TABLE on every driver by [@techmahedy](https://github.com/techmahedy) in [#312](https://github.com/doppar/framework/pull/312)
* Fix: DI resolution-order bug by [@techmahedy](https://github.com/techmahedy) in [#313](https://github.com/doppar/framework/pull/313)
* base path is now injected via the constructor, no global BASE_PATH constant by [@techmahedy](https://github.com/techmahedy) in [#315](https://github.com/doppar/framework/pull/315)
* refactor: Router depends on GatewayInterface instead of extending Kernel by [@techmahedy](https://github.com/techmahedy) in [#314](https://github.com/doppar/framework/pull/314)
* Environment configuration switches from .env to env.toml by [@techmahedy](https://github.com/techmahedy) in [#316](https://github.com/doppar/framework/pull/316)
* Fix 2FA QR code generation for chillerlan/php-qrcode v6 by [@techmahedy](https://github.com/techmahedy) in [#317](https://github.com/doppar/framework/pull/317)
* Redesign the 4.x welcome page across all frontend install stubs by [@techmahedy](https://github.com/techmahedy) in [#318](https://github.com/doppar/framework/pull/318)
* welcome page related change unit test issue fix: by [@techmahedy](https://github.com/techmahedy) in [#319](https://github.com/doppar/framework/pull/319)
* finalize the doppar 4.x version welcome page by [@techmahedy](https://github.com/techmahedy) in [#320](https://github.com/doppar/framework/pull/320)
* readme.md updated for 4.x version: by [@techmahedy](https://github.com/techmahedy) in [#321](https://github.com/doppar/framework/pull/321)
* Separate Request::capture() from superglobal reads by [@techmahedy](https://github.com/techmahedy) in [#322](https://github.com/doppar/framework/pull/322)
* fix: Harden Support\File against path traversal and spoofed MIME types by [@techmahedy](https://github.com/techmahedy) in [#323](https://github.com/doppar/framework/pull/323)
* Detect real file content for the mimes validation rule by [@techmahedy](https://github.com/techmahedy) in [#324](https://github.com/doppar/framework/pull/324)
* Bump dev master 3.x to 4.x by [@techmahedy](https://github.com/techmahedy) in [#325](https://github.com/doppar/framework/pull/325)
* readme.md updated by [@techmahedy](https://github.com/techmahedy) in [#326](https://github.com/doppar/framework/pull/326)
* README.md updated by [@techmahedy](https://github.com/techmahedy) in [#328](https://github.com/doppar/framework/pull/328)
