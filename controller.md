## Controllers
Rather than defining all request-handling logic as closures in route files, you can use controller classes to organize related functionality. Controllers centralize request handling, making your code more structured and maintainable. For example, a UserController can manage user-related actions like displaying, creating, updating, and deleting users. By default, controllers are stored in the app/Http/Controllers directory.

To quickly generate a new controller, you may run the `make:controller` Pool command. By default, all of the controllers for your application are stored in the `app/Http/Controllers` directory

```php
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

### Invokable Controllers
Doppar also support invokable controllers. You can call it single action controller also. To create a single action controller, need to pass the `--invok` option before create a controller.
```php
php pool make:controller ProductController --invok
```
Now this command will create a invokable controller for you. For invokable controller, route defination will be like
```php
Route::get('products', ProductController::class);
```

> Invokable Controllers are also resolved via the service container, so you may type-hint any dependencies you need within a Invokable Controllers's constructor.

