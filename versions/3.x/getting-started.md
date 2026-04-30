---
title: Getting Started
description: Doppar Getting Started page
meta:
  - name: keywords
    content: Getting Started, Doppar Framework, PHP Framework, Doppar ORM, Doppar Entity Builder
---

## Why Doppar?

Most PHP frameworks make you negotiate. You get expressiveness, but you pay in performance. You get speed, but you inherit complexity. Doppar ends that negotiation.

Every layer of the framework — from routing to data access — is engineered for both beauty and throughput simultaneously. Repeated executions are memoized intelligently. Dependencies are minimal by design. The result is a framework that reads like prose and runs like a machine.

Entity ORM and Entity Builder are built entirely from Doppar's core — no external dependencies, no third-party overhead. Complex relationships, expressive queries, and high-performance data access, all native.

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

Every feature in Doppar exists because we ran into a wall with existing solutions. A dependency injection system that buried bindings in service providers far from where they were used. A task scheduler that still needed crontab, Supervisor, or systemd to actually run. An ORM that secretly pulled in a dozen packages just to function. A request object that did one thing — carry data — when it could do so much more.

Doppar was built to fix those walls. Not with workarounds, but with first-principles rethinking of what each feature should actually do. The result is a set of capabilities that don't exist anywhere else in the PHP ecosystem — not as plugins, not as packages, and not as optional add-ons. They're native, opinionated, and designed to work together.

What follows are the features that set Doppar apart.

## Frontend Ready
Doppar ships with a first-class frontend workflow built directly into the framework. Whether you prefer React, Vue, Svelte, or Vanilla JavaScript, Doppar can scaffold the client setup, wire your Odo layout, configure Vite, and prepare your assets for both development and production in one guided flow.

Instead of forcing you to stitch together build tools, entry files, layout integration, and deployment output by hand, Doppar handles the boilerplate for you. Your server-rendered foundation stays intact, while modern client-side interfaces layer in cleanly on top.

With `php pool frontend:install`, you can choose your preferred frontend stack, CSS setup, and TypeScript support, then let Doppar generate the structure, config, and integration automatically. The result is a frontend-ready PHP framework that feels cohesive from the first render to the final build.

## Ultra-Clean Syntax - Unmatched Clarity
Doppar is built around one core principle — clarity without compromise. Every class, method, and directive is designed to be instantly understandable and beautifully expressive. Doppar turns complex backend logic into readable, fluent, and elegant code that feels natural to write and effortless to maintain.

Unlike traditional PHP frameworks that trade simplicity for abstraction, Doppar delivers both — a syntax that’s catchy. In Doppar, everything is explicit, discoverable, and self-documenting. No hidden bindings. No magic facades. No framework guesswork.
```php
#[Route(uri: 'user', middleware: ['auth'])]
public function store(
    #[Bind(UserRepository::class)] UserRepositoryInterface $userRepositoryInterface
) {
    //
}
```

With attribute-based routing and inline dependency binding, Doppar makes intent crystal clear. Every dependency, every route, and every behavior is defined right where it belongs — in your code.

## Unmatched Dependency Injection
Doppar’s Service Container redefines how PHP frameworks handle dependency management.
It offers an unparalleled level of clarity, control, and flexibility, allowing you to inject, bind, and resolve services without hidden magic or external dependencies.

With Doppar, dependency injection is first-class, built directly into the core — not added as an afterthought. Whether through service providers, attribute-based bindings, or automatic resolution, the container adapts seamlessly to your application’s structure.

Doppar keeps your bindings visible and meaningful. No verbose configurations. No hidden service maps. Just pure, expressive code that tells you exactly what’s happening.
```php
#[Route(uri: 'user/store', methods: ['POST'])]
public function store(
  #[Bind(UserRepository::class)] UserRepositoryInterface $userRepository
) {
    // #[Bind] resolves UserRepositoryInterface to UserRepository
    // Explicitly and transparently.
}
```

Localize your dependencies exactly where they are used. Doppar intelligently resolves classes without manual setup. It turns dependency management into an elegant, explicit part of your codebase. Doppar’s container is more than a dependency injector — it’s a clarity engine.

## Temporal Time Travel ORM
Most frameworks let you store data. Doppar lets you understand its history. With Doppar’s built-in Temporal ORM, every model becomes a time-aware entity. Every create, update, and delete operation is automatically captured as a full snapshot — giving you a complete, queryable timeline of your data with zero extra code.

At its core, the Temporal Time-Travel ORM does three things:

**Watches** — Every model marked with `#[Temporal]` automatically gets lifecycle hooks registered. After every create, update, or delete, a snapshot of the row is captured.

**Stores** — That snapshot — the full row as JSON, plus metadata — is written into a companion history table (`contracts_history`, `users_history`, etc.).

**Exposes** — A fluent time-travel API lives directly on the model. Query any past state, walk the full history, diff two points in time, rewind, restore.

No observer classes. No event listeners. No configuration files. One attribute, and it all works.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Database\Temporal\Attributes\Temporal;

#[Temporal]
class Contract extends Model
{
    protected $creatable = ['title', 'status', 'amount', 'client_id'];
}
```
Now run `php pool migrate:temporal` and that’s it.

What You Get Instantly
- **Complete audit history** — every change, every state, every moment
- **Time-travel queries** — fetch records exactly as they existed in the past
- **Diffing** — compare any two points in time
- **Undo / restore** — roll back data safely in one call
- **Actor tracking** — know exactly who made each change

Every change becomes part of a clean, structured timeline — built automatically.

## Frozen Services — Immutability at the Framework Level
Doppar introduces `#[Immutable]` — an attribute that enforces boot-time-only mutation on a service class. Once the application is fully booted, any attempt to modify an immutable service throws an `ImmutableViolationException` at runtime.
```php
#[Immutable]
class PaymentService
{
    use EnforcesImmutability;

    public string $gateway = 'stripe';
    public float  $taxRate  = 0.08;
}
```
During the boot phase, the service is fully configurable. Once booted, it becomes read-only. No other PHP framework has this.

## A Request Pipeline That Thinks
Doppar introduces a next-generation Request Object that goes far beyond simple input retrieval.
With fluent pipelines, inline validation, and declarative transformations, the Doppar Request turns raw input handling into a clean, expressive, and composable workflow.

Instead of juggling multiple validation layers or helper functions, Doppar lets you filter, transform, and ensure your data — all within a single, elegant chain.
```php
$data = $request
    ->pipeInputs([
        'title' => fn($v) => ucfirst(trim($v)),
        'tags'  => fn($v) => is_string($v) ? explode(',', $v) : $v,
    ])
    ->contextual(fn($data) => [
        'slug' => Str::slug($data['title']),
    ])
    ->ensure('slug', fn($slug) => strlen($slug) > 0)
    ->cleanse()
    ->nullifyBlanks()
    ->only('title', 'slug', 'tags');

Post::create($data);
```

This design makes Doppar’s Request object not just a data carrier — but a powerful input processing engine.
It gives you the clarity of functional pipelines with the simplicity of modern PHP

## Dual-Mode Task Scheduling
Doppar provides a powerful dual-mode scheduling engine that lets you run tasks the way your application needs — either through traditional cron or a real-time daemon loop. This flexibility makes Doppar suitable for everything from standard automation to high-frequency, second-based operations.

`Standard Mode` — triggered by a single cron entry, runs due tasks every minute. Perfect for lightweight automation, low-resource usage, and everyday jobs.
```bash
php pool cron:run
```

`Daemon Mode` — a continuous, real-time scheduling loop. Runs tasks at second-level granularity. No cron entry required. Perfect for monitoring pipelines, bots, IoT automation, and high-frequency tasks.
```bash
php pool cron:run --daemon
```

No other PHP framework offers second-level scheduling without external process managers.

### One Scheduler, Two Execution Styles
Whether you need basic automation or high-frequency task execution, Doppar gives you both — fully integrated into one cohesive scheduling system.

- Cron mode → simple, dependable, minimal
- Daemon mode → fast, reactive, always running

Choose the mode that fits your application, or combine both for maximum power.

## A Native Cron Daemon
Doppar includes a high-performance Cron Daemon, built directly into the framework.No server configuration. No crontab. No Supervisor. No systemd.

Just run:
```bash
php pool cron:daemon start
```
And boom, Doppar launches its own background process capable of executing tasks every second. You don’t need to edit `/etc/crontab`. You don’t need to use Supervisor. You don’t need systemd services. Doppar manages everything internally:
```bash
php pool cron:daemon start
php pool cron:daemon stop
php pool cron:daemon restart
php pool cron:daemon status
```
No crontab. No Supervisor. No systemd. Doppar launches its own managed background process, capable of executing tasks every second, handling PID tracking, overlap prevention, logging, error recovery, and graceful SIGTERM/SIGINT shutdown — all internally.

This means your scheduling behavior is identical in local, staging, and production, without touching a single server config file. Your entire scheduling system lives in your codebase, version-controlled, portable, and fully framework-native.

## Doppar Queue Component
Doppar's queue system goes beyond dispatching individual jobs. With `Drain::conduct()`, you can compose multi-step job pipelines with chained execution, error isolation, and completion callbacks:
```php
Drain::conduct([
    new DownloadFileJob($url),
    new ProcessFileJob($path),
    new UploadResultJob($file),
])
->then(fn() => Log::info('Pipeline complete'))
->catch(fn($job, $ex, $index) => Log::error("Step {$index} failed: ", [
    'job' => $job,
    'exception' => $ex,
    'index' => $index,
]))
->dispatch();
```

Job behavior is configured declaratively at the class level using #[Queueable]:
```php
#[Queueable(tries: 3, retryAfter: 10, delayFor: 300, onQueue: 'email')]
class SendWelcomeEmailJob extends Job
{
    // Retry logic, delay, and queue assignment — defined once, here.
}
```

## Model Hooks — Lifecycle Events as Methods
Doppar replaces observer classes and external event listeners with `#[Hook]` attributes directly on your model methods. Model lifecycle behavior lives in the model itself — no separate file, no registration ceremony:
```php
#[Hook('before_created')]
public function generateSlug(): void
{
    $this->slug = str()->slug($this->title);
}

#[Hook('after_updated')]
public function invalidateCache(): void
{
    Cache::delete("post:{$this->id}");
}
```
Available hooks: `before_created`, `after_created`, `before_updated`, `after_updated`, `before_deleted`, `after_deleted`. Everything stays co-located, readable, and maintainable.

## Real-Time WebSockets with Doppar Airbend
Doppar ships a full WebSocket broadcasting system — public channels, private channels, presence channels, whispers, and real-time metrics — all built in.
```javascript
// Client-side
const channel = airbender.channel('orders');
channel.listen('OrderShippedEvent', (data) => {
    console.log('Order shipped:', data.orderId);
});

// Presence channel
const presence = airbender.join('team.42');
presence.here(members => updateOnlineList(members));
presence.joining(user => addToList(user));
presence.leaving(user => removeFromList(user));
```

## API Presenter — Structured, Lazy, Composable
Doppar's API Presenter gives you a clean, expressive layer for shaping your API responses. Exclude fields, lazy-load relationships, and paginate — all in one fluent chain:
```php
UserPresenter::bundle(User::oldest('id')->paginate(20))
    ->except('password', 'remember_token')
    ->lazy()
    ->toPaginatedResponse();
```

## Bloom Filters — Built In
Doppar ships native Bloom filter support with MD5 and Murmur hashing strategies over Redis. Probabilistic existence checks — no external package:
```php
Bloom::key('seen_emails')->add($email);

if (Bloom::key('seen_emails')->has('random@other.com')) {
    // Probably seen before — skip expensive DB lookup
}
```

## Doppar AI Component
Doppar AI lets you run powerful AI models locally in PHP, combining TransformersPHP for on-device inference and Symfony AI Agent for a clean developer workflow. Choose any supported Hugging Face model, and call it directly from your controllers using the Pipeline API.

```php
use Doppar\AI\Pipeline;
use Doppar\AI\Enum\TaskEnum;

$result = Pipeline::execute(
    task: TaskEnum::SENTIMENT_ANALYSIS,
    data: 'I absolutely love Doppar.'
);
```
Output:
```php
[
    'label' => 'POSITIVE',
    'score' => 0.9998
]
```
On first run the model is downloaded automatically and then cached in `storage/app/transformers`, giving you fast, fully self-hosted AI responses.

Agent API — connect to any major AI provider with a fluent, consistent interface:

```php
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$stream = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->prompt('Explain quantum computing in simple terms')
    ->withStreaming()
    ->send();

foreach ($stream as $chunk) {
    echo $chunk;
    flush();
}
```
Supports `OpenAI`, `Gemini`, `Claude`, `OpenRouter`, and `self-hosted` models — all through the same clean API.

## Entity Builder and Entity ORM
Doppar introduces two powerful, fully native systems — Entity ORM and Entity Builder — built entirely from the core with zero external dependencies. Together, they redefine how developers interact with databases by combining expressive syntax, high performance, and total control.

![Doppar Entity Builder and Entity ORM](/assets/img/doppar-entity-builder.png)

## Entity ORM
Entity ORM is a modern, intuitive, and high-performance Object-Relational Mapper designed to make database interactions seamless and efficient. Each database table is represented by a dedicated Data Model, giving you a clean, object-oriented interface for querying, inserting, updating, and deleting records — all without writing raw SQL.

```php
// Define a Query Binding inside Post Model
public function __published($query)
{
    return $query->where('status', 'published');
}

// Usage
$posts = Post::published()->get();
```

Relationship make sense including relational column search
```php
Post::active()
    ->present('comments.reply', fn($q) => $q->where('approved', true))
    ->search(attributes: ['title', 'user.name', 'tags.name'], searchTerm: $request->search)
    ->embed(relations: ['category:id,name', 'user:id,name', 'tags'])
    ->embedCount(['tags', 'comments.reply' => fn($q) => $q->where('approved', true)])
    ->paginate(perPage: 10);
```

Crafted entirely within Doppar’s core, Entity ORM delivers pure, dependency-free performance for clean, reliable, and scalable data handling.

## Entity Builder
Entity Builder is a powerful, flexible, and model-free query builder built for developers who want full control over database operations — without relying on predefined models.

With Entity Builder, you can construct, execute, and manage even the most complex SQL queries through a clean, expressive, and chainable interface. From fetching and filtering to joins, inserts, and updates, Entity Builder translates raw SQL power into elegant, object-oriented syntax that feels intuitive and effortless.

```php
db()->bucket('post')
    ->if($request->input('search'),
        // If search is provided, filter by title
        fn($q) => $q->where('title', 'LIKE', "%{$request->search}%"),

        // If no search is provided, filter by featured status
        fn($q) => $q->where('is_featured', true)
    )
    ->get();
```

With Doppar Entity Builder, you get the freedom of SQL with the clarity, safety, and precision of Doppar.

## ODO — A Configurable Templating Engine
Doppar includes ODO, a lightweight templating engine built exclusively for Doppar. Unlike Blade or Twig, every part of ODO's syntax is configurable — directives, echo delimiters, raw output markers, comment syntax — all via `config/odo.php`. You define how your templates look.

## Core Concepts
Doppar brings together modern PHP practices and elegant simplicity — empowering developers to build with grace and precision.

| Feature | Description |
|----------|-------------|
| **Service Container** | On-demand service loading with optional smart provider resolution. |
| **Service Provider** | Cleanly separate concerns and manage dependencies with ease. |
| **Routing** | Supports both attribute-based and file-based routes. |
| **Model Hook** | Flexible approaches to handle Entity model events. |
| **Request** | An unparalleled Request object built for clarity and power. |
| **Middleware** | Global, web or route-specific; easily configurable in multiple ways. |
| **Controllers** | Structured request-response handling with intuitive flow. |
| **API Presenter** | A clean, structured boilerplate for API responses. |
| **Views** | Odo templates with custom control capabilities. |
| **Entity ORM** | A core-built, dependency-free Entity ORM for pure performance. |
| **Security** | CSRF protection, sessions, cookies, and validation out of the box. |
| **Auth** | Authentication, encryption, and annotation-based rate limiting. |
| **Utilities** | Built-in mail, file uploads, and caching tools. |
| **Atomic Lock** | Owner-based locking, Blocking and non-blocking modes with TTL. |
| **CLI** | A developer console for streamlined task execution. |
| **Localization** | Effortless support for multi-language applications. |

## API Ready
With API authentication, rate limiting, and JSON-first controllers, Doppar is ready for your next backend. Doppar is built from the ground up with API development in mind. Whether you're creating RESTful services, backend systems for mobile apps, or headless applications for modern frontend frameworks like Vue, React, or Angular—Doppar provides all the tools you need to build secure, scalable, and high-performance APIs.

###  JSON-First Philosophy

At the core of Doppar’s API readiness is its JSON-first controller structure. Responses are standardized, consistent, and optimized for API consumption. Doppar controllers can be easily configured to return JSON by default, with helpers to format responses, handle pagination, errors, and status codes—so you can focus on your logic, not the boilerplate.

### API Authentication

Doppar includes a flexible and secure API authentication system using **Doppar flarion**, supporting token-based authentication out of the box.

### Rate Limiting
Apply rate limits directly in your route definition — no middleware registration, no config files:
```php
#[Route(uri: 'api/login', methods: ['POST'], rateLimit: 10, rateLimitDecay: 1)]
public function login() { }

// Or with a PHP attribute
#[Throttle(limit: 60, decay: 1)]
public function apiEndpoint() { }

// Or with a DocBlock annotation
/** @RateLimit(limit=100, decay=60) */
public function search() { }
```

Three different styles, one consistent behavior.

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

### Implementation with `Encryptable` Contract
To enable encryption for a model, it must implement the `Phaseolies\Support\Contracts\Encryptable` interface and define the fields to be encrypted in the `getEncryptedProperties()` method.
```php
namespace App\Models;

use Phaseolies\Database\Entity\Model;
use Phaseolies\Support\Contracts\Encryptable;

class User extends Model implements Encryptable
{
    /**
     * Return an array of model attributes that should be encrypted
     */
    public function getEncryptedProperties(): array
    {
        return [
            'email', // This field will be encrypted
        ];
    }
}
```

When a model is serialized (e.g., converted to JSON for an API response or logging), the encrypted value is preserved unless you explicitly decrypt it, ensuring sensitive data is not accidentally exposed.
```php
{
    "id": 1,
    "name": "Aliba",
    "email": "T0RvMnZqWUIzVWhURkNKdWZSN0ZPaDZvN3g2M0o0L21nUTZ1",
    // ...
}
```

## API Security
Doppar's Flarion package provides a lightweight, stateless personal access token (PAT) system, ideal for APIs, mobile apps, and third-party integrations.

### Secure Token Lifecycle
Tokens are hashed using `HMAC-SHA256` with the application key to prevent brute-force or rainbow table attacks. Only the hashed lookup and metadata (user ID, abilities, expiration) are stored in the database. Raw tokens are never persisted

| Stage          | Security Mechanism                 | Description                                                                                                                   |
| -------------- | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Generation     | CSPRNG (bin2hex(random_bytes(40))) | Generates a cryptographically strong, unique, and unpredictable raw token.                                                    |
| Storage        | HMAC-SHA256 Hashing                | The raw token is never stored. Instead, a secure lookup hash is created using the application key and stored in the database. |
| Validation     | HMAC Verification                  | Incoming tokens are hashed and checked against the stored lookup hash for a fast and secure verification process.             |
| Access Control | Scoped Abilities                   | Each token is issued with a specific array of permissions (abilities), strictly limiting its power.                           |
| Expiration     | Configurable Lifespan              | Tokens can be configured to automatically expire, minimizing the window for a potential exploit.                              |

This architecture ensures that even if the database is breached, the raw access tokens are never compromised, making the system fast, simple, and enterprise-grade secure.

## Sensitive Input Exclusions
Doppar prevents sensitive information from persisting beyond the immediate request by defining a list of fields that are never stored in the session or debug context:

```php
// config/app.php excerpt
"exclude_sensitive_input" => [
    'password',
    '_insight_redirect_chain' // Example: for internal framework tooling
],
```

## Request Throttling
Doppar uses flexible middleware to protect routes from abuse and DoS attacks. The throttle middleware allows for fine-grained control over request limits.

Example Route Throttling:
```php
#[Route(uri: 'login', rateLimit: 10, rateLimitDecay: 1)]
public function login()
{
    //
}
```
If the limit is exceeded, the client receives an `HTTP 429 Too Many Requests` response, protecting the server resources. This prevent abuse and ensure fair usage with Doppar’s advanced rate-limiting features. 

Configure request thresholds per endpoint, IP, or user to protect your backend from DDoS attacks, brute-force attempts, and excessive API calls. Dynamic rate-limiting rules adapt to traffic patterns, ensuring optimal performance while maintaining service availability for legitimate users.

## Input Validation
Doppar provides a powerful and flexible input validation system to ensure incoming request data is always safe and properly formatted. Automatically validates and cleans request data using the `sanitize()` method. Further modifies or transforms validated inputs via `pipeInputs()` like method. Built-in protection for web routes to prevent Cross-Site Request Forgery.

Also model `$creatable` property to explicitly whitelist attributes that can be set in bulk operations, preventing unauthorized updates.

Example:
```php
$request->sanitize([
    'name' => 'required|min:2|max:20',
    'email' => 'required|email|unique:users|min:2|max:100',
    'password' => 'required|min:2|max:20',
    'confirm_password' => 'required|same_as:password',
]);

$payload = $request
    ->pipeInputs([
        'email' => fn($input) => strtolower(trim($input)),
        'password' => fn($input) => bcrypt($input)
    ])
    ->only('name', 'email', 'password');

User::create($payload);
```

### Designed to Scale

Whether you're handling **thousands of requests per minute** or operating within a **microservices architecture**, Doppar is ready. Its modular approach to routing, service containers, and middleware allows you to keep your API logic clean, maintainable, and testable.

### Contribute or Build Packages
One of Doppar’s greatest strengths is its **modular and extensible architecture**, which allows developers to **extend the framework by building custom packages** or integrating community-contributed modules. Whether you're adding new functionality, creating reusable components, or enhancing core features, **Doppar’s package development tools are designed to make the process seamless and maintainable**.

Doppar enables you to encapsulate features into packages that can be easily reused across multiple projects. This is especially useful for:

By following Doppar’s conventions, your packages can hook into the application's **service container**, **routing**, **middleware**, and more—just like core framework components. This means you can package your features professionally—with the same structure and power as native Doppar components.

Doppar makes package development not just possible, but **enjoyable and powerful**. Whether you're solving a problem for your own project or building tools for the wider community, **Doppar’s extensibility helps you do it cleanly, efficiently, and in a scalable way**.

## Build Something Great
Doppar is designed to get out of your way—so you can build faster and cleaner. From commands (pool) to rich Entity ORM tools, Doppar ensures your workflow is smooth and enjoyable.
