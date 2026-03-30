---
title: Starter Kits
description: Doppar starter kits page
meta:
  - name: keywords
    content: doppar-starter-kits
---

## Starter kits
To give you a head start building your new Doppar application, we are happy to offer application starter kits. These starter kits give you a head start on building your next Doppar application, and include the layouts, and extending views you need to start developmen.

This is the base layout file that your pages will extend. It defines the overall structure of your HTML document, including areas for dynamic title, styles, content, and scripts.

### Base Layout
This is the base layout file that your pages will extend. It defines the overall HTML document structure including areas for dynamic title, styles, content, and scripts. Notice that styles and scripts use `#slot` — this means any view or partial anywhere in the page can inject assets directly into the correct location in the document without needing to pass them up through the template hierarchy manually.

```html
[[-- resources/views/layouts/app.odo.php --]]
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>#yield('title') — [[ config('app.name') ]]</title>

    [[-- Global styles --]]
    <link rel="stylesheet" href="/css/app.css">

    [[-- Page and component specific styles injected here --]]
    #slot('styles')
</head>
<body>
    <div class="container mt-4">
        <div class="row justify-content-center">
            <div class="col-md-8">
                #yield('content')
            </div>
        </div>
    </div>

    [[-- Global scripts --]]
    <script src="/js/app.js"></script>

    [[-- Page and component specific scripts injected here --]]
    #slot('scripts')
</body>
</html>
```

###  Extending the Layout
Here is a view file that extends the base layout and fills in the dynamic sections. Page-specific styles and scripts are pushed into their respective slots using `#inject` so they land in the correct place in the document — inside `<head>` for styles and before `</body>` for scripts — regardless of where in the template they are declared.

```html
[[-- resources/views/dashboard.odo.php --]]
#extends('layouts.app')

[[-- Page title --]]
#section('title')
    Dashboard
#endsection

[[-- Inject page-specific styles into the layout's #slot('styles') --]]
#inject('styles')
    <link rel="stylesheet" href="/css/dashboard.css">
#endinject

[[-- Page content --]]
#section('content')
    <div class="card shadow-lg mb-4">
        <div class="card-header bg-primary text-white">
            <h5 class="mb-0">Dashboard</h5>
        </div>
        <div class="card-body">
            <p class="fw-bold fs-5">
                Welcome back, [[ Auth::user()->name ]]
            </p>
        </div>
    </div>

    #include('partials.stats')
    #include('partials.recent-activity')
#endsection

[[-- Inject page-specific scripts into the layout's #slot('scripts') --]]
#inject('scripts')
    <script src="/js/dashboard.js"></script>
#endinject
```

### Injecting from Partials
One of the key advantages of `#slot` and `#inject` over the older `#section` / `#append` approach is that any partial included anywhere in the page can push its own assets directly into the layout's slots — no matter how deeply nested the partial is.

```html
[[-- resources/views/partials/stats.odo.php --]]
#inject('scripts')
    <script src="/js/stats-widget.js"></script>
#endinject

<div class="row">
    <div class="col-md-3">
        <div class="card text-white bg-primary mb-3">
            <div class="card-body">
                <h5 class="card-title">Total Users</h5>
                <p class="card-text display-6">[[ $stats->totalUsers ]]</p>
            </div>
        </div>
    </div>
</div>
```

The final rendered document collects all injected content from the page, its sections, and all its partials and outputs everything in the correct location:
```html
<head>
    ...
    <link rel="stylesheet" href="/css/app.css">

    [[-- collected from all #inject('styles') calls --]]
    <link rel="stylesheet" href="/css/dashboard.css">
    <link rel="stylesheet" href="/css/activity-feed.css">
</head>
<body>
    ...
    <script src="/js/app.js"></script>

    [[-- collected from all #inject('scripts') calls --]]
    <script src="/js/dashboard.js"></script>
    <script src="/js/stats-widget.js"></script>
    <script src="/js/activity-feed.js"></script>
</body>
```
