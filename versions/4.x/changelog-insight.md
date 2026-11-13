---
title: Insight Changelog
description: Release history of Doppar Insight 4.x, generated from the doppar/insight GitHub releases
meta:
  - name: keywords
    content: changelog, release history, doppar 4.x, insight
---

## Changelog

Every tagged release of [`doppar/insight`](https://github.com/doppar/insight) in the 4.x line, newest first. Use the package selector above to switch between the framework and first-party packages. For the big picture of what changed between major versions, see the [Release Notes](/versions/4.x/releases).

## v4.1.3

_Released November 11, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/v4.1.3)

* Keep redirect chains out of the session and use the framework session API by [@techmahedy](https://github.com/techmahedy) in [#45](https://github.com/doppar/insight/pull/45)

## v4.1.2

_Released October 3, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/v4.1.2)

* Fix: the AJAX tab listed AJAX requests made by other pages and doppar v4.4.0 compatible by [@techmahedy](https://github.com/techmahedy) in [#44](https://github.com/doppar/insight/pull/44)

## v4.1.1

_Released September 26, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/v4.1.1)

* Fix Insight toolbar: register the profiler middleware through the router, and honor `enabled => false` by [@techmahedy](https://github.com/techmahedy) in [#43](https://github.com/doppar/insight/pull/43)

## v4.1.0

_Released September 26, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/v4.1.0)

* (feat): add ajax by [@rrr63](https://github.com/rrr63) in [#42](https://github.com/doppar/insight/pull/42)

## 4.0.1

_Released September 19, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/4.0.1)

* Fix: register ProfilerMiddleware through the router gateway on 4.x by [@techmahedy](https://github.com/techmahedy) in [#41](https://github.com/doppar/insight/pull/41)

## 4.0.0

_Released September 19, 2026_ · [View on GitHub](https://github.com/doppar/insight/releases/tag/4.0.0)

* Redesign Insight profiler toolbar and overview panel UI by [@sanzgrapher](https://github.com/sanzgrapher) in [#33](https://github.com/doppar/insight/pull/33)
* ready for doppar 4.x by [@techmahedy](https://github.com/techmahedy) in [#32](https://github.com/doppar/insight/pull/32)
* Fix Windows performance regression caused by Insight database profiling by [@rrr63](https://github.com/rrr63) in [#34](https://github.com/doppar/insight/pull/34)
* feat: harden Insight profile storage with atomic writes and disk limits by [@rrr63](https://github.com/rrr63) in [#35](https://github.com/doppar/insight/pull/35)
* feat: centralize sensitive data sanitization across Insight by [@rrr63](https://github.com/rrr63) in [#36](https://github.com/doppar/insight/pull/36)
* feat : display Redirected Request SQL in Database Queries by [@rrr63](https://github.com/rrr63) in [#37](https://github.com/doppar/insight/pull/37)
* fix: back to old css + show number of queries in toolbar including re… by [@rrr63](https://github.com/rrr63) in [#38](https://github.com/doppar/insight/pull/38)
* rm: sql toolbar ico by [@rrr63](https://github.com/rrr63) in [#39](https://github.com/doppar/insight/pull/39)
* readme.md updated for 4.x version: by [@techmahedy](https://github.com/techmahedy) in [#40](https://github.com/doppar/insight/pull/40)
