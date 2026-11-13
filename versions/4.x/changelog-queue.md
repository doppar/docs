---
title: Queue Changelog
description: Release history of Doppar Queue 4.x, generated from the doppar/queue GitHub releases
meta:
  - name: keywords
    content: changelog, release history, doppar 4.x, queue
---

## Changelog

Every tagged release of [`doppar/queue`](https://github.com/doppar/queue) in the 4.x line, newest first. Use the package selector above to switch between the framework and first-party packages. For the big picture of what changed between major versions, see the [Release Notes](/versions/4.x/releases).

## v4.2.0

_Released October 16, 2026_ · [View on GitHub](https://github.com/doppar/queue/releases/tag/v4.2.0)

* (feat) Worker: run every job in its own `defer()` scope by [@techmahedy](https://github.com/techmahedy) in [#32](https://github.com/doppar/queue/pull/32)
* support defer and execution context for each job by [@techmahedy](https://github.com/techmahedy) in [#33](https://github.com/doppar/queue/pull/33)

## v4.1.0

_Released September 25, 2026_ · [View on GitHub](https://github.com/doppar/queue/releases/tag/v4.1.0)

* Driver-based queue: Redis and memory drivers, priority, unique jobs, backoff, leases and crash recovery by [@techmahedy](https://github.com/techmahedy) in [#31](https://github.com/doppar/queue/pull/31)

## 4.0.0

_Released September 19, 2026_ · [View on GitHub](https://github.com/doppar/queue/releases/tag/4.0.0)

* ready for doppar 4.x by [@techmahedy](https://github.com/techmahedy) in [#28](https://github.com/doppar/queue/pull/28)
* fix phpstan error by [@techmahedy](https://github.com/techmahedy) in [#29](https://github.com/doppar/queue/pull/29)
* readme.md updated for 4.x version: by [@techmahedy](https://github.com/techmahedy) in [#30](https://github.com/doppar/queue/pull/30)
