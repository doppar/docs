  - [Installation](#installation)
  - [Requirements](#requirements)
  - [Environment Setup](#environment-setup)
  - [Creating an Application](#creating-an-application)
    - [Initial Configuration](#initial-configuration)
    - [Databases and Migrations](#databases-and-migrations)
  - [IDE Support](#ide-support)
  - [What's Next?](#whats-next)

<a name="installation"></a>
# ⚙️ Installation

Before you can start building with **Doppar**, ensure your local development environment is properly configured. This guide will walk you through installing the necessary dependencies and creating your first Doppar application.

<a name="requirements"></a>
## 🛠 Requirements

To get started with Doppar, you’ll need:

- **PHP 8.3 or higher**
- **Composer** (PHP dependency manager)

> Doppar is built on modern PHP features and requires **PHP 8.3+** to run correctly. Please make sure you're running the correct version before proceeding.

<a name="environment-setup"></a>
## 💻 Environment Setup

If you don’t already have **PHP** and **Composer** installed, you can quickly set them up using the platform-specific instructions below.

### macOS

```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/mac/8.3)"
```

### Windows (PowerShell)

```bash
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://php.new/install/windows/8.3'))
```
### Linux
```bash
/bin/bash -c "$(curl -fsSL https://php.new/install/linux/8.3)"
```
These scripts will install PHP 8.3 along with Composer in a reliable and clean manner. Always verify installation success by running:
```bash
php -v
composer -v
```

<a name="creating-an-application"></a>
### Creating an Application

After you have installed PHP, Composer, you're ready to create a new Doppar application. Run the following command to create a new Doppar application:
```bash
composer create-project doppar/doppar example-app
```

Once the application has been created, you can start Doppar's local development server like
```php
php pool start
```

<a name="initial-configuration"></a>
### Initial Configuration
All configuration settings for the Doppar framework are located in the **config** directory at the root of your application. Each file in this directory is neatly organized by responsibility and fully commented, making it easy to understand and customize the behavior of your application.

Doppar is designed to work with minimal configuration right out of the box. You can begin building your application immediately after installation. However, it's a good idea to explore the `config/app.php` file, as it contains several key settings such as the application name, environment, url, timezone, and locale—which you may want to tailor to match your project’s requirements.

Feel free to browse through the configuration files and adjust the settings to suit your development and deployment needs. With Doppar, flexibility and clarity in configuration are built-in.

### Environment-Based Configuration
In Doppar, many configuration values are designed to adapt depending on the environment in which your application is running—whether it's a local development machine, staging server, or a live production system.

To support this flexibility, Doppar uses a **.env** file located at the root of your project. This file allows you to define environment-specific settings such as the database connection, application URL, debug mode, mail credentials, and more.

Each environment (local, production, etc.) can have its own **.env** file with unique values. This approach makes it easy to manage different configurations without modifying your core configuration files.

 > ⚠️ Important: Never commit your .env file to version control.
Environment files often contain sensitive information like API keys, database passwords, and encryption secrets. Keeping them out of your repository helps protect your application from security breaches.

By separating environment settings from your codebase, Doppar ensures a clean, secure, and maintainable way to manage configuration across different deployment stages.

<a name="databases-and-migrations"></a>
### 🗄️ Databases & Migrations
Once your Doppar application is up and running, you’ll likely want to connect it to a database to start storing and retrieving data. By default, Doppar is preconfigured to use MySQL, offering a lightweight and fast setup.

If you prefer to work with a more robust database like MySQL or PostgreSQL, Doppar makes it easy to switch. Simply update the relevant **DB_*** variables in your **.env** file to match your database credentials:
```bash
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=doppar
DB_USERNAME=root
DB_PASSWORD=
```
Once your **.env** file is configured, you can create and modify your database schema using Doppar’s powerful migration system. Migrations let you version-control your database changes and keep your schema consistent across environments.

To run migrations:
```bash
php pool migrate
```
> 💡 Doppar migrations are stored in the **database/migrations directory** You can create a new migration with:

<a name="ide-support"></a>
### IDE Support
Doppar works seamlessly with any code editor or IDE, so you're free to choose the development environment that best fits your workflow.

However, to get the most out of Doppar’s developer experience, we recommend using an IDE with strong PHP and framework-specific support. Here are a few popular choices:
- PhpStorm offers outstanding PHP intelligence and Doppar compatibility right out of the box
- Visual Studio Code is a lightweight and powerful editor, widely used in the PHP community

Whether you're using Vim, Sublime Text, Emacs, or another editor, Doppar keeps things simple. All configuration, routing, and services follow clean, readable standards that don’t lock you into a specific IDE. No matter what tools you prefer, Doppar is built to offer a fast, intuitive, and flexible development experience—enhanced by editor support but never dependent on it.

<a name="whats-next"></a>
### What's Next?
Now that you have created your Doppar application, you may be wondering what to learn next. First, we strongly recommend becoming familiar with how Doppar works by reading the following documentation:

- Request Lifecycle
- Configuration
- Directory Structure
- Frontend
- Service Container
- Facades

How you want to use Doppar will also dictate the next steps on your journey.