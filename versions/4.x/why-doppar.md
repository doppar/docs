---
title: Why Doppar
description: Doppar Why Doppar page
meta:
  - name: keywords
    content: Why Doppar, Doppar Framework, PHP Framework, Doppar ORM, Doppar Entity Builder
---

## Why Doppar?

Most PHP frameworks make you negotiate. You get expressiveness, but you pay in performance. You get speed, but you inherit complexity. Doppar ends that negotiation.

Every layer of the framework — from routing to data access — is engineered for both beauty and throughput simultaneously. Repeated executions are memoized intelligently. Dependencies are minimal by design. The result is a framework that reads like prose and runs like a machine.

Entity ORM and Entity Builder are built entirely from Doppar's core. Complex relationships, expressive queries, and high-performance data access, all native.

Write code you're proud of. Ship software that holds up. Whether you're a seasoned PHP developer or just diving in, Doppar makes it easy to build powerful applications quickly and cleanly.

## Benchmark Snapshot

Doppar is designed to feel expressive without giving away raw
throughput. To give that claim some concrete shape, here is a simple
real-world benchmark from a local Doppar application running on
**Nginx + PHP-FPM + PHP 8.5** with `wrk` on **Ubuntu 22**.

The benchmark target was intentionally small and easy to reason about:
the `/` endpoint returning a single record from SQLite.

```php
#[Route(uri: '/', name: 'home')]
public function welcome()
{
    return User::find(1);
}
```

### Benchmark Environment

This benchmark was run with the following setup:

- web server: `nginx`
- PHP runtime: `PHP 8.5`
- process manager: `php-fpm`
- database: `SQLite`
- load generator: `wrk` on `Ubuntu 22`
- benchmark route: `/`
- benchmark behavior: fetch `User::find(1)` and return the model

### Benchmark Commands

The following `wrk` commands were used:

```bash
wrk -t2 -c50 -d20s http://localhost
wrk -t4 -c200 -d30s http://localhost
wrk -t8 -c500 -d60s http://localhost
```

### Benchmark Results

Here is the measured output summary:

| Threads | Connections | Duration | Avg Latency | Requests/sec | Transfer/sec | Notes |
|---------|-------------|----------|-------------|--------------|--------------|-------|
| `2`     | `50`        | `20s`    | `24.16ms`   | `2066.98`    | `1.23MB/s`   | Stable baseline |
| `4`     | `200`       | `30s`    | `94.76ms`   | `2103.41`    | `1.25MB/s`   | Best throughput in this run |
| `8`     | `500`       | `60s`    | `252.25ms`  | `1959.45`    | `1.17MB/s`   | Saturation starts to show |

### Benchmark Highlights

These three cards make the run easier to scan at a glance before
reading the detailed interpretation.

<div style="display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:16px;margin:20px 0 28px 0;">
    <div style="padding:18px 20px;border-radius:18px;border:1px solid var(--vp-c-divider);background:linear-gradient(180deg,var(--vp-c-bg-alt),rgba(56,189,248,0.10));box-shadow:0 12px 30px rgba(15,23,42,0.08);">
        <div style="font-size:12px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--vp-c-text-3);margin-bottom:8px;">Peak Throughput</div>
        <div style="font-size:28px;font-weight:800;color:var(--vp-c-text-1);line-height:1.1;">2103.41 req/s</div>
        <div style="margin-top:10px;font-size:14px;color:var(--vp-c-text-2);">Observed at <strong style="color:var(--vp-c-text-1);">4 threads / 200 connections</strong>.</div>
    </div>
    <div style="padding:18px 20px;border-radius:18px;border:1px solid var(--vp-c-divider);background:linear-gradient(180deg,var(--vp-c-bg-alt),rgba(34,197,94,0.10));box-shadow:0 12px 30px rgba(15,23,42,0.08);">
        <div style="font-size:12px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--vp-c-text-3);margin-bottom:8px;">Best Balance</div>
        <div style="font-size:28px;font-weight:800;color:var(--vp-c-text-1);line-height:1.1;">94.76 ms</div>
        <div style="margin-top:10px;font-size:14px;color:var(--vp-c-text-2);">Average latency at the strongest sustained throughput profile.</div>
    </div>
    <div style="padding:18px 20px;border-radius:18px;border:1px solid var(--vp-c-divider);background:linear-gradient(180deg,var(--vp-c-bg-alt),rgba(245,158,11,0.10));box-shadow:0 12px 30px rgba(15,23,42,0.08);">
        <div style="font-size:12px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--vp-c-text-3);margin-bottom:8px;">Highest Load Tested</div>
        <div style="font-size:28px;font-weight:800;color:var(--vp-c-text-1);line-height:1.1;">500 conns</div>
        <div style="margin-top:10px;font-size:14px;color:var(--vp-c-text-2);">Throughput stayed near 2k req/s, with pressure appearing through higher latency and socket reads.</div>
    </div>
</div>

### Throughput Visualization

The throughput stayed close to the 2k requests/sec range across all
three runs, with the middle profile producing the best result.

<div style="margin:18px 0 28px 0;padding:18px 18px 12px 18px;border-radius:22px;border:1px solid var(--vp-c-divider);background:linear-gradient(180deg,var(--vp-c-bg-soft),var(--vp-c-bg-alt));box-shadow:0 18px 40px rgba(15,23,42,0.08);overflow:auto;">
    <div style="font-size:13px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--vp-c-text-3);margin-bottom:10px;">Requests Per Second</div>
    <svg viewBox="0 0 860 280" width="100%" role="img" aria-label="Benchmark throughput chart showing requests per second across three wrk runs">
        <defs>
            <linearGradient id="dopparThroughputBar1" x1="0" y1="0" x2="1" y2="0">
                <stop offset="0%" stop-color="#0ea5e9" />
                <stop offset="100%" stop-color="#38bdf8" />
            </linearGradient>
            <linearGradient id="dopparThroughputBar2" x1="0" y1="0" x2="1" y2="0">
                <stop offset="0%" stop-color="#22c55e" />
                <stop offset="100%" stop-color="#4ade80" />
            </linearGradient>
            <linearGradient id="dopparThroughputBar3" x1="0" y1="0" x2="1" y2="0">
                <stop offset="0%" stop-color="#f59e0b" />
                <stop offset="100%" stop-color="#fbbf24" />
            </linearGradient>
        </defs>
        <rect x="0" y="0" width="860" height="280" rx="18" fill="transparent" />
        <g stroke="rgba(148,163,184,0.26)" stroke-width="1">
            <line x1="215" y1="55" x2="800" y2="55" />
            <line x1="215" y1="115" x2="800" y2="115" />
            <line x1="215" y1="175" x2="800" y2="175" />
            <line x1="215" y1="235" x2="800" y2="235" />
        </g>
        <g fill="var(--vp-c-text-3)" font-size="12" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="215" y="42">0 req/s</text>
            <text x="510" y="42" text-anchor="middle">1,050 req/s</text>
            <text x="800" y="42" text-anchor="end">2,100 req/s</text>
        </g>
        <g fill="var(--vp-c-text-2)" font-size="13" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="16" y="92">2 threads / 50 connections</text>
            <text x="16" y="152">4 threads / 200 connections</text>
            <text x="16" y="212">8 threads / 500 connections</text>
        </g>
        <rect x="215" y="68" width="577" height="26" rx="13" fill="url(#dopparThroughputBar1)" />
        <rect x="215" y="128" width="588" height="26" rx="13" fill="url(#dopparThroughputBar2)" />
        <rect x="215" y="188" width="548" height="26" rx="13" fill="url(#dopparThroughputBar3)" />
        <g fill="var(--vp-c-text-1)" font-size="14" font-weight="700" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="806" y="86">2066.98</text>
            <text x="806" y="146">2103.41</text>
            <text x="767" y="206">1959.45</text>
        </g>
    </svg>
</div>

### Latency Visualization

As expected, latency rose as concurrency increased.

<div style="margin:18px 0 28px 0;padding:18px 18px 12px 18px;border-radius:22px;border:1px solid var(--vp-c-divider);background:linear-gradient(180deg,var(--vp-c-bg-soft),var(--vp-c-bg-alt));box-shadow:0 18px 40px rgba(15,23,42,0.08);overflow:auto;">
    <div style="font-size:13px;font-weight:700;letter-spacing:.08em;text-transform:uppercase;color:var(--vp-c-text-3);margin-bottom:10px;">Average Latency</div>
    <svg viewBox="0 0 860 280" width="100%" role="img" aria-label="Benchmark latency chart showing average latency across three wrk runs">
        <defs>
            <linearGradient id="dopparLatencyLine" x1="0" y1="0" x2="1" y2="1">
                <stop offset="0%" stop-color="#8b5cf6" />
                <stop offset="100%" stop-color="#ec4899" />
            </linearGradient>
        </defs>
        <rect x="0" y="0" width="860" height="280" rx="18" fill="transparent" />
        <g stroke="rgba(148,163,184,0.26)" stroke-width="1">
            <line x1="170" y1="55" x2="810" y2="55" />
            <line x1="170" y1="105" x2="810" y2="105" />
            <line x1="170" y1="155" x2="810" y2="155" />
            <line x1="170" y1="205" x2="810" y2="205" />
            <line x1="170" y1="255" x2="810" y2="255" />
        </g>
        <g fill="var(--vp-c-text-3)" font-size="12" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="170" y="42">0 ms</text>
            <text x="383" y="42" text-anchor="middle">84 ms</text>
            <text x="597" y="42" text-anchor="middle">168 ms</text>
            <text x="810" y="42" text-anchor="end">252 ms</text>
        </g>
        <g fill="var(--vp-c-text-2)" font-size="13" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="120" y="246" text-anchor="middle">2t / 50c</text>
            <text x="380" y="246" text-anchor="middle">4t / 200c</text>
            <text x="640" y="246" text-anchor="middle">8t / 500c</text>
        </g>
        <polyline fill="none" stroke="url(#dopparLatencyLine)" stroke-width="5" stroke-linecap="round" stroke-linejoin="round" points="120,233 380,177 640,55" />
        <circle cx="120" cy="233" r="8" fill="#8b5cf6" />
        <circle cx="380" cy="177" r="8" fill="#a855f7" />
        <circle cx="640" cy="55" r="8" fill="#ec4899" />
        <g fill="var(--vp-c-text-1)" font-size="13" font-weight="700" font-family="Inter, ui-sans-serif, system-ui, sans-serif">
            <text x="120" y="214" text-anchor="middle">24.16 ms</text>
            <text x="380" y="158" text-anchor="middle">94.76 ms</text>
            <text x="640" y="36" text-anchor="middle">252.25 ms</text>
        </g>
    </svg>
</div>

### What This Result Shows

This benchmark is not meant to be a synthetic "hello world" victory
lap. The endpoint still enters the framework, resolves routing, touches
SQLite through the ORM, and returns a real model response.

Even with that full request path, Doppar sustained roughly **2,000
requests per second** on this setup. The most balanced run in this set
was:

- `4` threads
- `200` connections
- `30` seconds
- `2103.41` requests/sec
- `94.76ms` average latency

At the highest concurrency profile, throughput remained strong but the
system began to show pressure through higher latency and `604` socket
read errors. That is a useful signal: the benchmark did not collapse,
but it did reveal where this particular machine and stack started to
push beyond a comfortable steady state.

### How To Read These Numbers

Benchmarks always depend on workload, hardware, web server tuning,
database choice, opcode cache settings, and what the endpoint is
actually doing. A JSON API, a rendered view, a cache hit, and a complex
join-heavy query will all produce different numbers.

So treat this as a baseline snapshot of Doppar under a real but narrow
workload:

- a real HTTP request
- a real ORM lookup
- a real SQLite-backed response
- a standard `nginx + php-fpm` deployment shape

That baseline is encouraging because it shows Doppar can stay expressive
and still deliver strong request throughput on a straightforward PHP
stack.

## What Makes Doppar Different
Building a framework is easy. Building one that makes you genuinely rethink how PHP should feel — that's harder.

Every feature in Doppar exists because we ran into a wall with existing solutions. A dependency injection system that buried bindings in launchers far from where they were used. A task scheduler that still needed crontab, Supervisor, or systemd to actually run. An ORM that secretly pulled in a dozen packages just to function. A request object that did one thing — carry data — when it could do so much more.

Doppar was built to fix those walls. Not with workarounds, but with first-principles rethinking of what each feature should actually do. The result is a set of capabilities that don't exist anywhere else in the PHP ecosystem — not as plugins, not as packages, and not as optional add-ons. They're native, opinionated, and designed to work together.

What follows are the features that set Doppar apart.

## Frontend Ready
Doppar ships with a first-class frontend workflow built directly into the framework. Whether you prefer React, Vue, Svelte, or Vanilla JavaScript, Doppar can scaffold the client setup, wire your Odo layout, configure Vite, and prepare your assets for both development and production in one guided flow.

Instead of forcing you to stitch together build tools, entry files, layout integration, and deployment output by hand, Doppar handles the boilerplate for you. Your server-rendered foundation stays intact, while modern client-side interfaces layer in cleanly on top.

With `php pool frontend:install`, you can choose your preferred frontend stack, CSS setup, and TypeScript support, then let Doppar generate the structure, config, and integration automatically. The result is a frontend-ready PHP framework that feels cohesive from the first render to the final build.

## Dual-Mode Task Scheduling
Doppar provides a powerful dual-mode scheduling engine that lets you run tasks the way your application needs — either through traditional cron or a real-time daemon loop. This flexibility makes Doppar suitable for everything from standard automation to high-frequency, second-based operations.

`Standard Mode` — triggered by a single cron entry, runs due tasks every minute. Perfect for lightweight automation, low-resource usage, and everyday jobs.

### One Scheduler, Two Execution Styles
Whether you need basic automation or high-frequency task execution, Doppar gives you both — fully integrated into one cohesive scheduling system.

- Cron mode → simple, dependable, minimal
- Daemon mode → fast, reactive, always running

Choose the mode that fits your application, or combine both for maximum power.

## API Ready
With API authentication, rate limiting, and JSON-first controllers, Doppar is ready for your next backend. Doppar is built from the ground up with API development in mind. Whether you're creating RESTful services, backend systems for mobile apps, or headless applications for modern frontend frameworks like Vue, React, or Angular—Doppar provides all the tools you need to build secure, scalable, and high-performance APIs.

###  JSON-First Philosophy

At the core of Doppar’s API readiness is its JSON-first controller structure. Responses are standardized, consistent, and optimized for API consumption. Doppar controllers can be easily configured to return JSON by default, with helpers to format responses, handle pagination, errors, and status codes—so you can focus on your logic, not the boilerplate.

## Security by Default
Security is a first-class concern in Doppar. The framework is engineered to provide enterprise-grade protection out of the box, offering developers a secure foundation for applications ranging from microservices to full-scale enterprise systems.

### Core Security Features Overview
| Feature                     | Description                                                                   | Protection Against                                                      |
| --------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Model Properties Encryption | Automatic, transparent encryption of sensitive database fields at rest.       | Data breaches, unauthorized database access.                            |
| Stateless Authentication    | Lightweight, performant token-based authentication (Flarion).                 | Session hijacking, replay attacks, traditional session vulnerabilities. |
| CSRF Protection             | Automatically managed tokens for web routes.                                  | Cross-Site Request Forgery (CSRF).                                      |
| Input Validation            | Powerful, flexible, and strictly enforced request rules.                      | Injection attacks (SQLi, XSS), mass assignment, data corruption.        |
| Request Throttling          | Middleware-driven rate limiting for critical routes.                          | Denial of Service (DoS), brute force, API abuse.                        |
| Sensitive Input Exclusion   | Prevents sensitive fields (e.g., passwords) from being stored in the session. | Session exposure of sensitive user data.                                |
| Remember-Me Handling        | Secure and strict token generation and validation.                            | Persistent session hijacking.                                           |

From robust security features and flexible authentication to clean controller logic and performance-minded architecture, Doppar gives developers everything they need to build modern, production-grade APIs with confidence.

### Data-at-Rest Protection
Doppar provides a sophisticated and transparent way to encrypt cross-origin supported sensitive model attributes directly in the database.

## Build Something Great
Doppar is designed to get out of your way—so you can build faster and cleaner. From commands (pool) to rich Entity ORM tools, Doppar ensures your workflow is smooth and enjoyable.
