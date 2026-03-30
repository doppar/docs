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

## What Makes Doppar Different
Building a framework is easy. Building one that makes you genuinely rethink how PHP should feel — that's harder.

Every feature in Doppar exists because we ran into a wall with existing solutions. A dependency injection system that buried bindings in service providers far from where they were used. A task scheduler that still needed crontab, Supervisor, or systemd to actually run. An ORM that secretly pulled in a dozen packages just to function. A request object that did one thing — carry data — when it could do so much more.

Doppar was built to fix those walls. Not with workarounds, but with first-principles rethinking of what each feature should actually do. The result is a set of capabilities that don't exist anywhere else in the PHP ecosystem — not as plugins, not as packages, and not as optional add-ons. They're native, opinionated, and designed to work together.

What follows are the features that set Doppar apart.

## Ultra-Clean Syntax - Unmatched Clarity
Doppar is built around one core principle — clarity without compromise. Every class, method, and directive is designed to be instantly understandable and beautifully expressive. Doppar turns complex backend logic into readable, fluent, and elegant code that feels natural to write and effortless to maintain.

Unlike traditional PHP frameworks that trade simplicity for abstraction, Doppar delivers both — a syntax that’s catchy. In Doppar, everything is explicit, discoverable, and self-documenting. No hidden bindings. No magic facades. No framework guesswork.
```php
#[Route(uri: 'user/store', methods: ['POST'], middleware: ['auth'])]
public function store(
    #[Bind(YourConcrete::class)] YourAbstraction $lala
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

## Doppar Benchmark - High-Concurrency Performance Test
We stress-tested Doppar under extreme concurrency to evaluate its throughput, latency, and stability on a real database-backed endpoint. The results speak for themselves. We compared request handling and latency of Doppar under high concurrency with a database-backed endpoint.

### Test Setup
- **Load:** 50,000 requests
- **Concurrency:** 1,000 simultaneous requests
- **Endpoint:** `/tags` (database-backed endpoint to fetch `tags`)
- **Tool:** ApacheBench (ab)
- **Metrics Measured:** Requests/sec, latency distribution, max latency, response size, error rate

### Benchmark Results

| Metric                     | **Doppar**                | **Notes**                                               |
| -------------------------- | ------------------------- | ------------------------------------------------------- |
| **Total Requests**         | 50,000                    | Total requests completed during the test run            |
| **Failed Requests**        | 0                         | 100% success rate, no dropped or errored responses      |
| **Requests/sec (RPS)**     | **318.5 req/s**           | Sustained throughput under 1,000 concurrent users       |
| **Avg Latency**            | \~3.1s (3100 ms)          | Mean response time across all requests                  |
| **Median Latency (P50)**   | \~2.7s (2703 ms)          | Half of all requests completed at or below this latency |
| **P75 Latency**            | \~3.6s                    | 75% of requests completed within this time              |
| **P95 Latency**            | \~4.8s                    | 95% of requests completed within this time              |
| **P99 Latency**            | \~6.2s                    | Only 1% of requests exceeded this latency               |
| **Max Latency**            | \~7.9s                    | Slowest observed request                                |
| **Response Size**          | 1083 bytes                | Size of a single JSON payload returned from `/tags`     |
| **Concurrency Level**      | 1,000 connections         | Simultaneous open requests during the test              |
| **Total Data Transferred** | \~54 MB (50,000 × 1083 B) | Useful for bandwidth & scaling considerations           |

### Key Takeaways

- **High Throughput**
  - Sustained 318+ requests per second with 1,000 concurrent users — a level that pushes most frameworks to their limits.
- **Stable Latency Under Load**
  - Median response time remained below 3 seconds, even when hitting the database under extreme concurrency.
- **Zero Failures**
  - Across 50,000 requests, Doppar maintained a 100% success rate with no dropped or failed connections.

### Why Doppar Stands Out

Handling thousands of concurrent connections while keeping performance predictable is notoriously difficult. Most PHP frameworks show significant degradation in these conditions — but Doppar was engineered for concurrency from the ground up.

### What Sets Doppar Apart

- `7–8× higher throughput` than typical PHP frameworks in comparable benchmarks
- `Predictable latency` that keeps user experience smooth, even under peak traffic
- `Optimized database access`, making it ideal for data-intensive applications

👉 If you’re building systems that demand real scalability, high concurrency, and reliable performance, Doppar is ready to power them.

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