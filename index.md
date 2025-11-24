---
title: Doppar - The PHP Framework
description: Doppar The PHP Framework
meta:
  - name: keywords
    content: The PHP Framework, Where Performance Meets Aristocratic Syntax

layout: home

hero:
  name: Doppar
  text: Where Performance Meets Aristocratic Syntax
  tagline: Unlock the full power of your PHP applications with ODO, Doppar’s native templating engine, and Doppar AI
  actions:
    - theme: brand
      text: Why Doppar
      link: /versions/3.x/getting-started
    - theme: sponsor
      text: Get Started
      link: /versions/3.x/installation
    # - theme: alt
    #   text: Learn
    #   link: https://www.youtube.com/@doppar-3x

---

## ✨ Doppar AI Component
> Doppar AI Agents let you interact with any language model seamlessly — whether self-hosted or via OpenAI.
```php
// Self-Hosted
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\SelfHost;

$response = Agent::using(SelfHost::class)
    ->withHost('http://localhost:1234')
    ->model('local-model-name')
    ->prompt('Generate a PHP function to validate email')
    ->send();

// With OpenAI
use Doppar\AI\Agent;
use Doppar\AI\AgentFactory\Agent\OpenAI;

$response = Agent::using(OpenAI::class)
    ->withKey(env('OPENAI_API_KEY'))
    ->model('gpt-3.5-turbo')
    ->prompt('Explain Doppar PHP Framework')
    ->send();
```

## ✨ Routing
> With Doppar, a single line route definition gives you **full routing power** — HTTP method restriction, route naming, and middleware support — all **declaratively, elegantly, and without boilerplate**.
```php
#[Mapper(prefix: 'user', middleware: ['auth'])]
class UserController extends Controller
{
    #[Route(
        uri: '/{user}',                   // Route URL; {user} is the route parameter
        methods: ['GET'],                 // Allowed HTTP method
        name: 'user',                     // Named route
        middleware: ['is_admin'],         // Middleware applied
        rateLimit: 10,                    // Max requests allowed
        rateLimitDecay: 1                 // Decay period in minutes
    )]
    public function show(#[Model] ?User $user)
    {
        // Example endpoint: http://example.com/user/aliba@doppar.com
        // `$user` is automatically resolved by email (route-model binding)

        return $user; // Returns the User model instance
    }
}
```

## ✨ Binding Services to Abstraction
> Doppar allows you to inject dependencies directly at the method signature. By simply decorating a parameter, it immediately clear which concrete implementation is being used:
```php
class PostController extends Controller
{
    public function __construct(
        #[Bind(PostRepository::class)] readonly private PostRepositoryInterface $postRepository
    ) {}

    #[Route('/')]
    public function __invoke(
      #[Bind(AnotherConcrete::class)] AnotherAbstract $anotherRepository
    )
    {
      //
    }
}
```

## ✨ Automatic Transaction Wrapping
> Doppar allows you to automatically wrap a method (like __invoke) in a database transaction. By decorating the method with `#[Transaction]` attribute. This removes the traditional transaction boilerplate and collapses transaction management into a declarative, single line in your method signature:
```php
class PaymentController extends Controller
{
    #[Transaction]
    #[Route('payment', methods: ['POST'])]
    public function payment()
    {
      // Automatically wraps the payment method in a DB transaction
    }
}
```

## ✨ Model Hook
> Doppar’s Model Hooks provide an elegant way to tap into your models’ lifecycle events, allowing you to execute custom logic automatically during creation, updates, deletion, or booting.
```php
class User extends Model
{
    protected $hooks = [
        'after_updated' => [
            'handler' => App\Hooks\UserUpdatedHook::class,
            'when' => [self::class, 'isAdmin']
        ],
    ];

    public static function isAdmin(Model $model): bool
    {
      return (int) $model->role === 'admin';
    }
}

class UserUpdatedHook
{
    public function handle(Model $model): void
    {
        if ($model->isDirtyAttr('name')) {
            info("Name changed from {$model->getOriginal('name')} to {$model->name}");
        }
    }
}
```

## ✨ Queue
> The Doppar queue system is designed to handle background tasks efficiently with reliability and scalability in mind. Its feature set ensures smooth job processing, better performance, and full control over how tasks are executed.
```php
#[Queueable(tries: 3, retryAfter: 10, delayFor: 300, onQueue: 'email')]
class SendWelcomeEmailJob extends Job
{
    //
}
```

## ✨ The Magic of Parameter-Level Hydration
> Traditionally, a controller action receiving a POST request to create a user. With Doppar’s #[BindPayload], this entire setup collapses into a declarative, single line in your method signature:
```php
public function store(Request $request) {
    // 1. Validate the request data.
    // 2. Extract the data.
    // 3. Create a new User object and fill it manually.
    $user = new User();
    $user->name = $request->input('name');
    // ... many more lines of mapping
    $user->save();
}

// With #[BindPayload] Attribute
public function store(
    #[BindPayload] User $user // The $user object is fully ready!
) {
    return User::createFromModel($user);
}
```

## ✨ Entity ORM
> Doppar Entity ORM functionality with expressive reusable query, selective fields, and embedded relationships, all while keeping code clean and readable delivering pure, high-performance data handling with zero external overhead.
```php
public function __active(Builder $query): Builder
{
    return $query->whereStatus(true);
}

Post::active()
    ->present('comments.reply', function ($query) {
        $query->where('approved', true);
    })
    ->search(
        attributes: [
            'title',
            'user.name',
            'category.name',
            'tags.name',
            'comments.body',
            'comments.reply.body',
        ],
        searchTerm: $request->search
    )
    ->embed(
        relations: [
            'category:id,name',
            'user:id,name',
            'tags',
        ]
    )
    ->embedCount([
        'tags',
        'comments.reply' => fn($q) => $q->where('approved', true),
    ])
    ->paginate(perPage: 10);
```

<div class="why-doppar">
  <div class="hero-header">
    <h1 class="hero-title">
      <span class="gradient-text">Why Doppar?</span>
      <div class="title-underline"></div>
    </h1>
    <p class="hero-subtitle">Doppar is engineered for speed — every repeated execution is intelligently memoized, ensuring results are delivered instantly without unnecessary reprocessing.</p>
  </div>
  <div class="features-container">
    <div class="feature-card performance">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">Blazing Fast Performance</h3>
        </div>
        <p class="feature-description">Doppar’s highly optimized core is engineered for maximum speed—delivering near-instant response times, low latency, and seamless execution even under heavy load.</p>
        <ul class="feature-list">
          <li><span class="list-icon">⚡</span> Lightweight architecture</li>
          <li><span class="list-icon">📉</span> Minimal overhead</li>
          <li><span class="list-icon">🔄</span> Efficient resource usage</li>
        </ul>
      </div>
    </div>
    <div class="feature-card architecture">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">ODO – The Doppar Templating Engine</h3>
        </div>
        <p class="feature-description">
          Doppar introduces <strong>ODO</strong>, a clean, configurable, and framework-native templating engine.
          ODO delivers a modern, lightweight, syntax-focused approach to templating.
        </p>
        <ul class="feature-list">
          <li><span class="list-icon">#️⃣</span> Fast, minimal, framework-native template compiler</li>
          <li><span class="list-icon">💉</span> Fully customizable syntax via <code>config/odo.php</code></li>
          <li><span class="list-icon">🧩</span> Clean directive with 100% configurable</li>
        </ul>
      </div>
    </div>
    <div class="feature-card scaling">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">Entity ORM</h3>
        </div>
        <p class="feature-description">Crafted entirely within Doppar's core, this ORM eliminates all third-party dependencies—delivering pure, high-performance data handling with zero external overhead.</p>
        <ul class="feature-list">
            <li><span class="list-icon">🔌</span> Fully core-powered ORM</li>
            <li><span class="list-icon">🧠</span> Expressive syntax</li>
            <li><span class="list-icon">🚀</span> High performance queries</li>
        </ul>
      </div>
    </div>
    <div class="feature-card scaling">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">Hook-Driven Model Architecture</h3>
        </div>
        <p class="feature-description">Doppar’s model hooks goes beyond and a cleaner, more flexible lifecycle API with support for inline, class-based, and conditional hooks directly in the ORM core.</p>
        <ul class="feature-list">
            <li><span class="list-icon">🔌</span> Hook into key model stages: boot, update, etc</li>
            <li><span class="list-icon">🧠</span> Inline, class-based, and conditional logic</li>
            <li><span class="list-icon">🚀</span> Clean, testable architecture with zero clutter</li>
        </ul>
      </div>
    </div>
    <div class="feature-card scaling">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">Task Schedule</h3>
        </div>
        <p class="feature-description">Doppar runs tasks with second-level accuracy, enabling real-time automation, monitoring. Perfect for applications that cannot wait for the next minute.</p>
        <ul class="feature-list">
            <li><span class="list-icon">🔔</span> Smart dual-mode engine (second and minute)</li>
            <li><span class="list-icon">🎉</span> Doppar’s built-in daemon for continuous execution</li>
            <li><span class="list-icon">🚀</span> If a task fails, Doppar's daemon continues running without dying</li>
        </ul>
      </div>
    </div>
    <div class="feature-card scaling">
      <div class="card-decoration"></div>
      <div class="card-content">
        <div class="feature-header">
          <h3 class="gradient-text">Doppar Queue</h3>
        </div>
        <p class="feature-description">The Doppar Framework queue system is designed to handle background tasks efficiently with reliability and scalability in mind</p>
        <ul class="feature-list">
            <li><span class="list-icon">🧩</span> Organize jobs by priority and type</li>
            <li><span class="list-icon">📌</span> Configurable retry attempts with delays</li>
            <li><span class="list-icon">📈</span> Schedule jobs for future execution</li>
        </ul>
      </div>
    </div>
  </div>
</div>

&nbsp;
<div>
<div class="hero-header">
    <h1 class="hero-title">
      <span class="gradient-text">Doppar Offers</span>
      <div class="title-underline"></div>
    </h1>
    <p class="hero-subtitle">Doppar offers a rich set of features out of the box — from a fast, core-powered ORM to built-in routing, validation, and concurrency controls. Everything you need to build modern, high-performance applications with zero unnecessary dependencies.</p>
  </div>

  <div class="features-grid">
    <div class="feature-card" data-feature="container">
      <div class="card-content">
        <h3>Service Container</h3>
        <div class="feature-list">
          <span>Automatic resolution</span>
          <span>Contextual bindings</span>
          <span>Singleton/Transient</span>
        </div>
        <a href="/versions/3.x/service-container" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="middleware">
      <div class="card-content">
        <h3>Middleware</h3>
        <div class="feature-list">
          <span>Throttle handling</span>
          <span>Global/Route-specific</span>
          <span>Lightweight pipeline</span>
        </div>
        <a href="/versions/3.x/middleware" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="authentication">
      <div class="card-content">
        <h3>API Authentication</h3>
        <div class="feature-list">
          <span>Stateless Auth</span>
          <span>Scoped permissions</span>
          <span>Revocable tokens</span>
        </div>
        <a href="/versions/3.x/doppar-flarion" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="limiter">
      <div class="card-content">
        <h3>Rate Limiter</h3>
        <div class="feature-list">
          <span>Dynamic throttling</span>
          <span>Redis/File backends</span>
          <span>Custom thresholds</span>
        </div>
        <a href="/versions/3.x/rate-limiting" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="cache">
      <div class="card-content">
        <h3>Cache</h3>
        <div class="feature-list">
          <span>Multi-driver</span>
          <span>Tag invalidation</span>
          <span>Auto revalidation</span>
        </div>
        <a href="/versions/3.x/caching" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="packages">
      <div class="card-content">
        <h3>Task Schedule</h3>
        <div class="feature-list">
          <span>Exclude date</span>
          <span>Throttle cron handler</span>
          <span>Cron with retry</span>
        </div>
        <a href="/versions/3.x/package-development" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="packages">
      <div class="card-content">
        <h3>Pool Console</h3>
        <div class="feature-list">
          <span>Dev Acceleration</span>
          <span>Class Generation</span>
          <span>Task Scheduling</span>
        </div>
        <a href="/versions/3.x/package-development" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="packages">
      <div class="card-content">
        <h3>Processes</h3>
        <div class="feature-list">
          <span>Concurrency</span>
          <span>Async Processes</span>
          <span>Command Injection</span>
        </div>
        <a href="/versions/3.x/package-development" class="feature-link">Learn More →</a>
      </div>
    </div>
    <div class="feature-card" data-feature="doppar-axios">
      <div class="card-content">
        <h3>Doppar Axios</h3>
        <div class="feature-list">
          <span>Fluent HTTP Client</span>
          <span>Promise-like Syntax</span>
          <span>Built-in Retry & Timeout</span>
        </div>
        <a href="/versions/3.x/doppar-axios" class="feature-link">Learn More →</a>
      </div>
    </div>
  </div>
</div>
&nbsp;

