---
title: MVC Architecture
description: Doppar MVC Architecture page
meta:
  - name: keywords
    content: MVC, Controllers, Models, Views
---

## Introduction

Doppar follows the Model-View-Controller (MVC) architectural pattern, a widely accepted standard in web application development. This design separates the application logic into three distinct components: `Model`, `View`, and `Controller`.

The Model is responsible for managing the data, business rules, and interactions with the database. The View handles the presentation layer, rendering content for users based on the data it receives. The Controller acts as a mediator, processing user input, calling the appropriate model logic, and returning the corresponding view. This separation of concerns improves code organization and makes the application easier to maintain and scale.

By isolating business logic from UI and request handling, MVC enhances testability and collaboration among developers. Doppar’s MVC implementation ensures a clean workflow for building robust and responsive web applications. It encourages reusable components and facilitates rapid development while maintaining structure.

This architecture divides the application into three interconnected components, each with a distinct responsibility:

- **Model:** Manages the data and business logic.
- **View:** Handles presentation and UI rendering.
- **Controller:** Interacts with the request, processes input, and returns a response.

This separation promotes organized, testable, and maintainable code.

Here's a simple MVC Request Lifecycle Diagram that outlines how a typical request flows through an MVC-based application like Doppar:

```
        +-------------------+       +--------------------+
        |   User (Browser)  |       |    HTTP Request    |
        |  Sends a Request  |       | (e.g. URL, form)   |
        +---------+---------+       +---------+----------+
                  \                       /
                   \                     /
                    v                   v
        +-------------------+       +--------------------+
        |    Controller     |       |      Model         |
        | - Handles input   |       | - Business logic   |
        | - Calls model     |       | - Data access      |
        +---------+---------+       +---------+----------+
                  \                       /
                   \                     /
                    v                   v
                 +---------------------------+
                 |           View            |
                 |  - Renders response (UI)  |
                 |  - Returns HTML/JSON/XML  |
                 +-------------+-------------+
                               |
                               v
                 +---------------------------+
                 |   User receives response  |
                 +---------------------------+
```

This layout clearly shows parallel components and how they converge to the final view sent back to the user.

## Controllers

Controllers in Doppar act as the entry point for handling HTTP requests. They receive input from the user, delegate it to the appropriate model, and return a view or JSON response.

Controllers are stored in the `App\Http\Controllers` namespace by default.

Basic Controller Example

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use Phaseolies\Utilities\Attributes\Route;

class PostController extends Controller
{
    #[Route(uri: 'posts')]
    public function index()
    {
        return view('posts.index');
    }

    #[Route(uri: 'posts', method: 'POST')]
    public function store(Request $request)
    {
        //
    }
}
```

## Models

Models in Doppar represent your application's data structure and interact directly with the database. They typically reside in the `App\Models` namespace.

Models should encapsulate business logic, relationships, and rules for a single entity.

Basic Model Example

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    //
}
```

## Views

Views are responsible for presenting data to the user. Doppar uses its native `odo` templating engine for rendering views with clean, readable syntax.

Views are typically stored in the `resources/views` directory.

Returning a View from Controller

```php
return view('posts.index', ['posts' => $posts]);
```

The MVC pattern in Doppar helps you write clean, well-organized, and scalable web applications. Whether you're building a simple blog or a complex web app, Doppar's MVC implementation gives you structure and flexibility.
