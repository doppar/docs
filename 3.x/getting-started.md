---
title: Getting Started
description: Doppar Getting Started page
meta:
  - name: keywords
    content: Getting Started
---

## Why Doppar?

Doppar is engineered for speed — every repeated execution is intelligently memoized, ensuring results are delivered instantly without unnecessary reprocessing. With minimal reliance on third-party libraries and most features built directly into the core, you get lightning-fast performance right out of the box. No unnecessary bloat—just clean, efficient execution

Doppar ORM Built entirely from core with zero external dependencies, Doppar delivers a powerful and expressive ORM system. Manage complex relationships with ease—no third-party packages required.

Whether you're a seasoned PHP developer or just diving in, Doppar makes it easy to build powerful applications quickly and cleanly.

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

### Key highlights:
- **Simplicity with Power** Intuitive architecture that stays out of your way.
- **Feature-Based Development** Organize your application by features, not layers.
- **Just-in-Time (JIT)** Advanced JIT compilation for the Blade template engine.
- **Performance First** Lightweight and efficient, designed for speed without compromise.
- **Scalable & Modular** Perfect for projects of all sizes—from microservices to full-scale.
- **API Presenter Bundle** Fully internal, zero-config API Presenter. no overrides required.
- **Two-Factor Authentication (TOTP)** Industry-standard TOTP Authentication.
- **Rich Ecosystem** Eloquent ORM, Routing, Middleware, Pool Console, Caching and more.
- **Developer-Focused** Simplifies development with thoughtful conventions.

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
<p>Doppar brings together modern PHP practices and simplicity. Here's a taste of what’s available:</p>

- **Routing**: Parameters and naming
- **Middleware**: Global & route-specific
- **Controllers**: Structured request/response handling
- **Views**: Blade-like templates with asset bundling
- **ORM**: Built entirely from core with zero external dependencies
- **Security**: CSRF Protection, sessions, cookies, and validation
- **Auth**: Authentication, encryption, rate limiting
- **Utilities**: Mail, file uploads, caching
- **CLI**: Console (pool) for development tasks
- **Localization**: Helpers and more

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

Rate limiting is managed via middleware, and it's fully customizable for different route groups (e.g., public vs. authenticated API).

### Security by Default

API security is a first-class concern in Doppar. The framework includes:

  - CSRF protection for web routes
  - Input validation using powerful and flexible rules
  - Request throttling
  - Header-based authentication
  - Encryption and decryption utilities

In short, **Doppar isn't just capable of building APIs—it’s engineered for it**. From robust security features and flexible authentication to clean controller logic and performance-minded architecture, Doppar gives developers everything they need to build modern, production-grade APIs with confidence.

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
Doppar is designed to get out of your way—so you can build faster and cleaner. From commands (pool) to rich Eloquent ORM tools, Doppar ensures your workflow is smooth and enjoyable.