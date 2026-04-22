---
title: Your First App
description: Build your first Doppar application from scratch
meta:
  - name: keywords
    content: first app, doppar tutorial, getting started
---

- [Overview](#overview)
- [What We're Building](#what-were-building)
- [Setting Up the Project](#setting-up-the-project)
- [Defining Routes](#defining-routes)
  - [Web Routes](#web-routes)
  - [Attribute-Based Routes](#attribute-based-routes)
- [Creating a Controller](#creating-a-controller)
- [Working with Views](#working-with-views)
- [Handling Form Submissions](#handling-form-submissions)
- [Returning JSON Responses](#returning-json-responses)
- [What's Next?](#whats-next)


## Overview

Now that you've installed Doppar and configured your environment, it's time to build something real. This guide walks you through creating a simple task manager application — covering routing, controllers, views, form handling, and responses from the ground up.

By the end of this guide, you'll have a solid understanding of how the core pieces of Doppar fit together.

## What We're Building

We'll build a minimal **Task Manager** that allows users to:

- View a list of tasks
- Submit a new task via a form
- Return a JSON response for API consumers

This covers the most common patterns you'll encounter in any Doppar application.

## Setting Up the Project

If you haven't already created your Doppar application, run:

```bash
composer create-project doppar/doppar task-manager
```

Navigate into your project and start the local development server:

```bash
cd task-manager

php pool server:start
```

Your application will be available at `http://localhost:8000`.

> Make sure your `.env` file is properly configured before proceeding. At minimum, verify your `APP_URL`, `APP_ENV`, and database settings are correct.

## Defining Routes
Doppar provides defining routes directly above controller methods using `#[Route]` attributes. This keeps your route definitions close to the logic they control:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;

class TaskController extends Controller
{
    #[Route(uri: 'tasks', name: 'tasks.index')]
    public function index()
    {
        //
    }

    #[Route(uri: 'tasks', methods: ['POST'], name: 'tasks.store')]
    public function store()
    {
        //
    }

    #[Route(uri: 'tasks/api', name: 'tasks.api')]
    public function apiList()
    {
        //
    }
}
```

Both approaches are fully supported. Use whichever style suits your project's conventions.

## Creating a Controller
Doppar provides `make:controller` command to create a new controller. Run below command to create `TaskController`.
```bash
php pool make:controller TaskController
```

Controllers live in the `app/Http/Controllers` directory. Create a new `TaskController`:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use Phaseolies\Http\Response;

class TaskController extends Controller
{
    /**
     * In-memory task list for demonstration purposes.
     */
    private array $tasks = [
        ['id' => 1, 'title' => 'Install Doppar', 'done' => true],
        ['id' => 2, 'title' => 'Read the documentation', 'done' => false],
        ['id' => 3, 'title' => 'Build something awesome', 'done' => false],
    ];

    /**
     * Display the list of tasks.
     */
    public function index()
    {
        return view('tasks.index', [
            'tasks' => $this->tasks,
        ]);
    }

    /**
     * Handle a new task submission.
     */
    public function store(Request $request)
    {
        $title = $request->input('title');

        // In a real app, you'd persist this to a database.
        // For now, we'll redirect back with a success message.

        return redirect()->route('tasks.index')
            ->with('success', 'Task "' . $title . '" was added successfully!');
    }

    /**
     * Return the task list as a JSON response.
     */
    public function apiList()
    {
        return response()->json([
            'data'  => $this->tasks,
            'total' => count($this->tasks),
        ]);
    }
}
```

Doppar's service container automatically resolves and injects the `Request` instance into your controller method — no manual wiring needed.

## Working with Views

Views are stored in the `resources/views` directory. Doppar uses the **Odo** templating engine, with files ending in `.odo.php`.

Create the directory and view file at `resources/views/tasks/index.odo.php`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Task Manager</title>
</head>
<body>

    <h1>My Tasks</h1>

    #if (session()->has('success'))
        <div class="alert alert-success">
            [[ session()->pull('success') ]]
        </div>
    #endif

    <ul>
        #foreach ($tasks as $task)
            <li>
                [[ $task['title'] ]]
                #if ($task['done'])
                    <em>(done)</em>
                #endif
            </li>
        #endforeach
    </ul>

    <hr>

    <h2>Add a New Task</h2>

    <form method="POST" action="[[ route('tasks.store') ]]">
        #csrf
        <input type="text" name="title" placeholder="Task title" required>
        <button type="submit">Add Task</button>
    </form>

</body>
</html>
```

A few things to note:

- `#csrf` generates the required CSRF token field. All `POST`, `PUT`, `PATCH`, and `DELETE` form submissions must include this token, or Doppar will reject the request.
- `[[ $variable ]]` outputs escaped data — equivalent to `echo htmlspecialchars($variable)`.
- `route('tasks.store')` generates the URL for the named route, keeping your templates decoupled from hardcoded paths.

## Handling Form Submissions

When the form above is submitted, Doppar routes the `POST /tasks` request to `TaskController@store`. Inside that method, you can access all submitted input through the `Request` object:

```php
public function store(Request $request)
{
    $title = $request->input('title');

    return redirect()->route('tasks.index')
        ->with('success', 'Task "' . $title . '" was added successfully!');
}
```

After saving the task, we redirect the user back to the task list and flash a success message to the session. The view then reads that message using `session()->pull('success')` and displays it to the user.

<div class="doppar-alert doppar-alert-danger">
Never trust raw user input. In production applications, always validate and sanitize data before persisting it. Doppar provides a robust validation system — refer to the validation documentation for details.
</div>

## Returning JSON Responses

For API consumers, the `apiList` method returns the task list as a proper JSON response:

```php
public function apiList()
{
    return response()->json([
        'data'  => $this->tasks,
        'total' => count($this->tasks),
    ]);
}
```

Visiting `http://localhost:8000/tasks/api` in your browser or a REST client will return:

```json
{
  "data": [
    { "id": 1, "title": "Install Doppar", "done": true },
    { "id": 2, "title": "Read the documentation", "done": false },
    { "id": 3, "title": "Build something awesome", "done": false }
  ],
  "total": 3
}
```

The `response()->json()` method automatically sets the `Content-Type` header to `application/json` and serializes your data. You can also include a status code as the second argument:

```php
return response()->json(['message' => 'Created'], 201);
```

## What's Next?

You've now built a working Doppar application covering the most essential concepts. Here are some recommended next steps to deepen your understanding:

- [Routing](/versions/3.x/routing) — Explore named routes, route groups, middleware, and resource bundles
- [Request Lifecycle](/versions/3.x/request-lifecycle) — Understand how Doppar processes an incoming request end-to-end
- [Responses](/versions/3.x/responses) — Learn about redirects, file downloads, streaming, and response caching
- [Service Container](/versions/3.x/service-container) — Discover how Doppar manages dependencies and injects them automatically
- [Middleware](/versions/3.x/middleware) — Protect routes and filter requests with reusable middleware layers
- [Database & Migrations](/versions/3.x/migrations) — Connect to a real database and version-control your schema

How far you take Doppar depends on what you're building — but the foundation you've set up here will carry you the entire way.