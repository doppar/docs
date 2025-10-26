---
title: Getting Started
description: Doppar Getting Started page
meta:
  - name: keywords
    content: Getting Started, Doppar Framework, PHP Framework, Doppar ORM, Doppar Entity Builder
---

## Why Doppar?

As developers, we constantly seek harmony in our craft — the balance between elegance and performance. Often, we find one at the cost of the other: elegant syntax that slows us down, or high performance wrapped in complexity.

Doppar brings both worlds together. It offers aristocratic elegance in syntax and uncompromising performance under the hood — a framework designed for developers who value both beauty and speed.

Doppar is engineered for speed — every repeated execution is intelligently memoized, ensuring results are delivered instantly without unnecessary reprocessing. With minimal reliance on third-party libraries and most features built directly into the core, you get lightning-fast performance right out of the box. No unnecessary bloat—just clean, efficient execution

Doppar `Entity ORM` and `Entity Builder` Built entirely from core with zero external dependencies, Doppar delivers a powerful and expressive `Entity ORM` and `Entity Builder` system. Manage complex relationships with ease—no third-party packages required.

Whether you're a seasoned PHP developer or just diving in, Doppar makes it easy to build powerful applications quickly and cleanly.

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

## Unmatched Request Object
Doppar introduces a next-generation Request Object that goes far beyond simple input retrieval.
With fluent pipelines, inline validation, and declarative transformations, the Doppar Request turns raw input handling into a clean, expressive, and composable workflow.

Instead of juggling multiple validation layers or helper functions, Doppar lets you filter, transform, and ensure your data — all within a single, elegant chain.
```php
$data = $request
    ->pipeInputs([
        'title' => fn($v) => ucfirst(trim($v)),
        'tags' => fn($v) => is_string($v) ? explode(',', $v) : $v
    ])
    ->contextual(fn($data) => [
        'slug' => Str::slug($data['title'])
    ])
    ->ensure('slug', fn($slug) => strlen($slug) > 0)
    ->only('title','slug');

Post::create($data);
```

This design makes Doppar’s Request object not just a data carrier — but a powerful input processing engine.
It gives you the clarity of functional pipelines with the simplicity of modern PHP

## Doppar Benchmark: High-Concurrency Performance Test
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

![Doppar Entity Builder and Entity ORM](/doppar-entity-builder.png)

## Entity ORM
Entity ORM is a modern, intuitive, and high-performance Object-Relational Mapper designed to make database interactions seamless and efficient. Each database table is represented by a dedicated Data Model, giving you a clean, object-oriented interface for querying, inserting, updating, and deleting records — all without writing raw SQL.

Beyond simple CRUD operations, Entity ORM empowers you to manage complex data relationships with ease. Features like eager loading, nested relationships, and relationship-based filtering make it effortless to work with related data while eliminating repetitive boilerplate code.

Whether you’re handling straightforward tables or building intricate relational architectures, Entity ORM keeps your workflow smooth, expressive, and maintainable — turning database interactions into a natural, enjoyable part of your development process.

Crafted entirely within Doppar’s core, Entity ORM delivers pure, dependency-free performance for clean, reliable, and scalable data handling.

## Entity Builder
Entity Builder is a powerful, flexible, and model-free query builder built for developers who want full control over database operations — without relying on predefined models.

With Entity Builder, you can construct, execute, and manage even the most complex SQL queries through a clean, expressive, and chainable interface. From fetching and filtering to joins, inserts, and updates, Entity Builder translates raw SQL power into elegant, object-oriented syntax that feels intuitive and effortless.

While Entity ORM excels in model-driven development, Entity Builder shines in dynamic scenarios — such as data analysis, raw manipulation, or rapid prototyping — letting you query any table directly. It supports nearly all ORM-level query capabilities (minus relationship-based operations) while maintaining the same clarity, consistency, and speed.

Fully integrated into Doppar’s ecosystem, Entity Builder runs without any third-party dependencies, ensuring unmatched reliability and performance across all data-driven applications.

With Doppar Entity Builder, you get the freedom of SQL with the clarity, safety, and precision of Doppar.

## JIT Template Compilation
Doppar brings advanced `Just-in-Time (JIT)` compilation to the Blade template engine — a performance-focused feature that transforms how views are compiled and rendered at runtime.

### What is JIT Optimization?
In Doppar, JIT optimization refers to the process of dynamically analyzing and transforming compiled Blade templates at the moment they're needed — just before rendering. Instead of relying solely on precompiled static views, JIT inspects templates during request execution and applies smart, runtime optimizations to reduce rendering cost and speed up response times.

### Why JIT Makes Doppar Exceptional
JIT is enabled by default `(BLADE_JIT_ENABLED=true)` and you can configure it from `.env` and comes with an adjustable optimization level:
| Level | Description                       | Features Included                                                                   |
| ----- | --------------------------------- | ----------------------------------------------------------------------------------- |
| 0     | Disabled                          | No runtime optimization, standard Blade behavior                                    |
| 1     | Basic Optimization                | Whitespace cleanup, faster control structures, echo consolidation                   |
| 2     | Aggressive Optimization (default) | Inlines small templates, simplifies nested loops, lazy-loads components dynamically |

### What JIT Does Under the Hood
When a view is compiled, Doppar’s JIT engine intelligently refines the resulting PHP code using techniques like:

<ul class="doppar-features">
  <li><strong>Whitespace Reduction</strong>: Compresses unnecessary spacing to reduce file size and improve parsing speed.</li>
  <li><strong>Optimized Control Structures</strong>: Transforms verbose `@if`, `@foreach`, and `@else` blocks into efficient native PHP syntax, leading to cleaner and faster execution.</li>
  <li><strong>Echo Optimization</strong>: Combines adjacent `{{ }}` outputs into single expressions, significantly reducing PHP overhead during rendering.</li>
  <li><strong>Blade Loop Simplification</strong>: Detects and optimizes common loop constructs (like `@foreach`) for faster iteration and cleaner compiled output, improving rendering efficiency.</li>
  <li><strong>Inline Small Templates</strong>: Automatically inlines `@include()` directives for small views (under 500 bytes). This eliminates disk I/O for these components, speeding up overall load times.</li>
  <li><strong>Lazy Component Loading</strong>: Converts Blade components into shared, in-memory objects that render only once per request. This intelligent caching mechanism drastically reduces duplication and improves performance for frequently used components..</li>
</ul>

### Why It Matters
Doppar’s JIT system is designed to improve the real-world performance of your Blade templates, especially:

- On high-traffic sites where rendering efficiency matters
- When using reusable components across pages
- For developers who want to keep using Blade’s elegance without sacrificing speed

Doppar doesn’t just compile Blade — it understands it, improves it, and runs it faster than ever.

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
| **Views** | JIT-powered Blade templates with custom control capabilities. |
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

To prevent abuse and improve performance, Doppar provides built-in rate limiting. You can apply rate limits globally, per user, per route, or based on IP. With simple configuration, you can:

  - Protect against brute force attacks
  - Control load on your infrastructure
  - Improve the fairness of resource distribution

Rate limiting is managed via middleware, and it's fully customizable. You can load it using attribute based routing system or annotation based or even more file based routing system.

### Security by Default

API security is a first-class concern in Doppar. The framework includes:

  - Lightweight, stateless authentication
  - CSRF protection for web routes
  - Cross-Origin Model properties encryption
  - Input validation using powerful and flexible rules
  - Request throttling
  - Header-based authentication
  - Encryption and decryption utilities
  - Secure and strict remember-me token handling.

In short, **Doppar isn't just capable of building APIs. it’s engineered for it**. From robust security features and flexible authentication to clean controller logic and performance-minded architecture, Doppar gives developers everything they need to build modern, production-grade APIs with confidence.

### Intelligent Rate Limiting
Prevent abuse and ensure fair usage with Doppar’s advanced rate-limiting features. Configure request thresholds per endpoint, IP, or user to protect your backend from DDoS attacks, brute-force attempts, and excessive API calls. Dynamic rate-limiting rules adapt to traffic patterns, ensuring optimal performance while maintaining service availability for legitimate users.

### Designed to Scale

Whether you're handling **thousands of requests per minute** or operating within a **microservices architecture**, Doppar is ready. Its modular approach to routing, service containers, and middleware allows you to keep your API logic clean, maintainable, and testable.

### Contribute or Build Packages
One of Doppar’s greatest strengths is its **modular and extensible architecture**, which allows developers to **extend the framework by building custom packages** or integrating community-contributed modules. Whether you're adding new functionality, creating reusable components, or enhancing core features, **Doppar’s package development tools are designed to make the process seamless and maintainable**.

Doppar enables you to encapsulate features into packages that can be easily reused across multiple projects. This is especially useful for:

By following Doppar’s conventions, your packages can hook into the application's **service container**, **routing**, **middleware**, and more—just like core framework components. This means you can package your features professionally—with the same structure and power as native Doppar components.

Doppar makes package development not just possible, but **enjoyable and powerful**. Whether you're solving a problem for your own project or building tools for the wider community, **Doppar’s extensibility helps you do it cleanly, efficiently, and in a scalable way**.

## Build Something Great
Doppar is designed to get out of your way—so you can build faster and cleaner. From commands (pool) to rich Entity ORM tools, Doppar ensures your workflow is smooth and enjoyable.