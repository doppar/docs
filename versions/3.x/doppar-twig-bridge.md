---
title: Twig Bridge
description: Integrate Twig templates into your Doppar applications.
meta:
  - name: keywords
    content: twig templates views doppar framework
---

## Twig Bridge

### Introduction

The Doppar Twig Bridge is an integration layer that allows applications built on the Doppar Framework to render views using Twig, a powerful and flexible templating engine. The bridge is intentionally minimalistic and non-intrusive so you can adopt Twig progressively without disrupting your existing Odo-based templates.

This document provides an in-depth explanation of how the bridge works, how to install it, and how to use it effectively in real-world Doppar applications.

## Installation

Installing the Twig Bridge is straightforward. From the root of your Doppar application, run:

```bash
composer require doppar/twig-bridge
```

---

## Register the service provider

In your application's `config/app.php`, add the Twig bridge service provider to the `providers` array:

```php
'providers' => [
    Doppar\TwigBridge\TwigServiceProvider::class,
],
```

There is no additional configuration file required. Once the provider is registered, Twig is ready to use.

---

## How it works

### Controller binding

The Doppar global `view()` helper resolves the main controller class from the container:

```php
view('some.view', [...]);
```

The Twig bridge service provider binds a custom controller that extends Doppar's base controller and decides, at render time, whether to use the original Odo engine or Twig.

### When Twig is used

Twig is **only** used when the view name ends with `.twig`.

- `view('home')` → **Odo** (existing Doppar templating engine)
- `view('home.html.twig')` → **Twig** (through the Twig bridge)

This allows you to adopt Twig incrementally without breaking existing views.

---

## Using Twig in a Doppar app

### Create a Twig view

Example file: `resources/views/hello.html.twig`

```twig
<h1>Hello {{ name }}</h1>
{{ dump(name) }}
```

### Return a Twig view from a controller

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

The Doppar Twig Bridge offers a flexible and efficient way to integrate Twig templates into Doppar applications. With minimal setup and complete backward compatibility.

You can adopt Twig incrementally, keep your existing Odo templates untouched, and enjoy the full power of Twig where you choose to use it.