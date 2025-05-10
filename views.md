## Views
### Introduction
In Doppar, it's not practical to return entire HTML document strings directly from your routes and controllers. Fortunately, views offer a clean and structured way to manage your application's UI by keeping HTML in separate files.

Views help separate your application's logic from its presentation layer, improving maintainability and readability. In Doppar, views are typically stored in the `resources/views` directory. The templating system in Doppar allows you to create dynamic and reusable UI components efficiently. A basic view file might look like this:
```html
<!-- View stored in resources/views/greeting.blade.php -->
<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Since this view is stored at `resources/views/greeting.blade.php`, we may return it using the global view helper like so:

```php
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

### Passing Data to Views
As you saw in the previous examples, you may pass an array of data to views to make that data available to the view:
```php
return view('greetings', ['name' => 'Mahedi']);
```

### Optimizing Views
In Doppar Framework, Blade template views are compiled on demand. When a request is made to render a view, Doppar checks if a compiled version exists. If the compiled view is missing or outdated compared to its source, Doppar will recompile it dynamically.

However, compiling views at runtime introduces a minor performance overhead. To optimize performance, Doppar provides the view:cache command, which precompiles all views ahead of time. This can be particularly useful during deployment.

```bash
php pool view:cache
```

You may use the view:clear command to clear the view cache:
```bash
php pool view:clear
```
