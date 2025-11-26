---
title: Twig Bridge
description: Integrate Twig templates into your Doppar applications.
meta:
  - name: keywords
    content: twig templates views doppar framework
---

## Doppar Twig Bridge

### Introduction

The **Doppar Twig Bridge** provides a thin integration layer between **Twig** and the **Doppar Framework**.

Its goals are:

- **Keep the existing Doppar view system (Odo) working unchanged.**
- **Enable Twig only when you explicitly ask for it.**
- **Require virtually no configuration in your application.**

---

## Installation

From your Doppar application:

```bash
composer require doppar/twig-bridge:dev-main
composer dump-autoload
```

---

## Register the service provider

In your application's `config/app.php`, add the Twig bridge service provider to the `providers` array:

```php
'providers' => [
    App\Providers\AppServiceProvider::class,
    Doppar\TwigBridge\TwigServiceProvider::class,
],
```

There is no additional configuration file required. Once the provider is registered, Twig is ready to use.

---

## How it works

### 1. Controller binding

The Doppar global `view()` helper resolves the main controller class from the container:

```php
view('some.view', [...]);
```

The Twig bridge service provider binds a custom controller that extends Doppar's base controller and decides, at render time, whether to use the original Odo engine or Twig.

### 2. When Twig is used

Twig is **only** used when the view name ends with `.twig`.

- `view('home')` → **Odo** (existing Doppar templating engine)
- `view('home.html.twig')` → **Twig** (through the Twig bridge)

This allows you to adopt Twig incrementally without breaking existing views.

---

## Using Twig in a Doppar app

### 1. Create a Twig view

Example file: `resources/views/hello.html.twig`

```twig
<h1>Hello {{ name }}</h1>
{{ dump(name) }}
```

### 2. Return a Twig view from a controller

In one of your HTTP controllers:

```php
use Phaseolies\Http\Response;
use Phaseolies\Attributes\Route;

class WelcomeController
{
    #[Route(uri: '/', name: 'home')]
    public function welcome(): Response
    {
        return view('hello.html.twig', ['name' => 'MyName']);
    }
}
```