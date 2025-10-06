---
title: Controllers
description: Doppar controllers page
meta:
  - name: keywords
    content: controllers
---

## Controllers
### Introduction
Rather than defining all request-handling logic as closures in route files, you can use controller classes to organize related functionality. Controllers centralize request handling, making your code more structured and maintainable. For example, a UserController can manage user-related actions like displaying, creating, updating, and deleting users. By default, controllers are stored in the app/Http/Controllers directory.

## Create Controller
To quickly generate a new controller, you may run the `make:controller` Pool command. By default, all of the controllers for your application are stored in the `app/Http/Controllers` directory

```bash
php pool make:controller UserController
```

Let's take a look at an example of a basic controller. A controller may have any number of public methods which will respond to incoming HTTP requests:
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     */
    public function show(string $id)
    {
        return view('user.profile', [
            'user' => User::find($id)
        ]);
    }
}
```

Once you have written a controller class and method, you may define a route to the controller method like so:
```php
use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

## Single Action Controller
Doppar also support invokable controllers. You can call it single action controller also. To create a single action controller, need to pass the `--invokable` option before create a controller.
```bash
php pool make:controller ProductController --invokable
```
This command will generate an invokable controller in the `app\Http\Controllers` directory. The generated controller will look like this:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use App\Http\Controllers\Controller;

class ProductController extends Controller
{
    /**
     * Handle the incoming request.
     */
    public function __invoke(Request $request)
    {
        //
    }
}
```

When using an invokable controller, the route definition looks like this:
```php
Route::get('products', ProductController::class);
```

> Invokable Controllers are also resolved via the service container, so you may type-hint any dependencies you need within a Invokable Controllers's constructor or directly in `__invoke` method.

## Bundle Controller
Doppar also supports bundle controllers. These are controllers that contain all the standard CRUD (Create, Read, Update, Delete) methods in a single class.

To create a bundle controller, use the `--bundle` option when generating the controller:
```bash
php pool make:controller ProductController --bundle
```
This command will generate a controller in the `app/Http/Controllers` directory with all the typical bundle methods pre-defined.

Example Generated Controller
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use App\Http\Controllers\Controller;

class ProductController extends Controller
{
    /**
     * Display a listing of the items
     *
     * @return \Phaseolies\Http\Response
     */
    public function index()
    {
        //
    }

    /**
     * Show the form for creating a new item
     *
     * @return \Phaseolies\Http\Response
     */
    public function create()
    {
        //
    }

    /**
     * Create new item
     *
     * @param Request $request
     * @return \Phaseolies\Http\Response\RedirectResponse
     */
    public function store(Request $request)
    {
        //
    }


    /**
     * Display the specified item.
     *
     * @param int $id
     * @return \Phaseolies\Http\Response
     */
    public function show(int $id)
    {
        //
    }

    /**
     * Show the form for editing the specified item.
     *
     * @param int $id
     * @return \Phaseolies\Http\Response
     */
    public function edit(int $id)
    {
        //
    }

    /**
     * Update the specified item in storage.
     *
     * @param Request $request
     * @param int $id
     * @return \Phaseolies\Http\Response\RedirectResponse
     */
    public function update(Request $request, int $id)
    {
        //
    }

    /**
     * Remove the specified item from storage.
     *
     * @param int $id
     * @return \Phaseolies\Http\Response\RedirectResponse
     */
    public function delete(int $id)
    {
        //
    }
}
```

## API Bundle Controller
Doppar also supports api bundle controllers. These are controllers that contain all the standard CRUD (Create, Read, Update, Delete) methods in a single class.

To create a api bundle controller, use the `--api` option when generating the controller:
```bash
php pool make:controller ProductController --api
```
This command will generate a controller in the `app/Http/Controllers/API` directory with all the typical bundle methods pre-defined except `create` and `edit` method.