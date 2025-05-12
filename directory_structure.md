## Directory Structure
### Introduction
Doppar Framework is a lightweight, modern PHP application framework inspired by Laravel’s architectural elegance and modular design principles. It aims to provide a clean, expressive, and scalable codebase that accelerates web application development while remaining easy to understand and extend. Whether you're building RESTful APIs, full-stack web applications, or microservices, Doppar offers the foundation and flexibility you need.

The framework is built around familiar conventions and emphasizes simplicity without compromising on power. It features an MVC (Model-View-Controller) architecture, a powerful routing system, middleware support, a service provider-based bootstrapping mechanism, and organized directory structures. Additionally, it integrates seamlessly with Composer for dependency management and supports environment-based configuration via .env files.

Doppar promotes clean code, testability, and maintainability, offering essential development tools such as migrations, seeders, and a built-in command-line interface for managing application logic. It encourages developers to follow modern PHP standards such as PSR-4 autoloading, namespacing, and dependency injection.

Whether you're a seasoned PHP developer coming from Laravel or a newcomer looking for a structured framework to start with, Doppar provides the right balance between simplicity and capability. Below is a comprehensive guide to the Doppar framework’s directory structure to help you understand where everything fits.

Take a look at default folder structure
```markdown
└── 📁doppar-framework
    └── 📁app
        └── 📁Http
            └── 📁Controllers
            └── Kernel.php
            └── 📁Middleware
        └── 📁Models
        └── 📁Providers
    └── 📁bootstrap
    └── 📁config
    └── 📁lang
    └── 📁database
        └── 📁migrations
        └── 📁seeds
    └── 📁public
    └── 📁resources
        └── 📁views
    └── 📁routes
    └── 📁storage
        └── 📁app
            └── 📁public
        └── 📁cache
        └── 📁logs
        └── 📁sessions
    └── 📁tests
    └── .env
    └── .env.example
    └── .gitignore
    └── composer.json
    └── composer.lock
    └── pool
```

### Directory Structure
The Doppar framework's default directory structure is organized as follows:

### app/
The `app` directory contains the core logic of your application. It follows a modular and domain-driven design by default.
### app/Http/
This folder handles all HTTP-related functionality like routing, middleware, and controllers.

### app/Http/Controllers/
This is where all controller classes are defined. Controllers handle user requests, process business logic (or delegate it), and return responses—usually in the form of views or JSON.

### app/Http/Middleware/
Middleware classes are stored here. Middleware allows you to filter or modify HTTP requests entering your application, such as authentication, CORS, logging, etc.

### app/Http/Kernel.php
This is the core HTTP Kernel that registers global and route-specific middleware and handles the incoming HTTP request lifecycle.

### app/Models/
This directory contains your application's Eloquent-style models or custom ORM classes. These models represent your data schema and encapsulate logic for interacting with your database.

### app/Providers/
Application service providers live in this directory. They are responsible for bootstrapping various components of your application like binding services into the container, event listeners, etc

### bootstrap/
This folder contains files responsible for initializing the framework. The `app.php` file located here is loaded first and sets up the application environment, services, and configuration.

### config/
All of your configuration files live here. Configuration settings such as database credentials, mail, logging, and application-specific settings are managed in this directory, and are accessible globally throughout the app.

### lang/
Localization and language files are stored here. This enables multi-language support for your application, including validation messages and UI labels.

### database/
This directory holds all database-related resources.

### database/migrations/
Migration files that define your database schema changes (create, update, delete tables/columns).

### database/seeds/
Database seeders allow you to populate your database with sample or testing data.

### public/
The public directory is the web root of your application. All user-accessible content such as images, JavaScript, CSS files, and the main `index.php` live here. This file acts as the entry point for all web requests and boots the application.

### resources/
This directory stores frontend-facing resources and templates.

### resources/views/
Contains view files, such as Blade or native PHP templates, which are returned from controllers and rendered as HTML.

### routes/
Route definitions for the web and API are stored here. You can define closures or controller-based routes inside this folder.

### storage/
The storage directory is used to store logs, compiled views, cache files, and session data.

### storage/app/public/
Stores user-uploaded files or other content meant to be publicly accessible. A symbolic link from `public/storage` points here.

### .env
The environment file contains sensitive configuration variables such as database passwords, app mode, API keys, etc. It’s loaded into the application at runtime and should not be committed to version control.

### pool
It contains custom CLI (Command Line Interface) commands used to interact with and manage the application. The commands defined in pool are executed via the Doppar CLI tool, providing a powerful and extensible interface for automating development.

