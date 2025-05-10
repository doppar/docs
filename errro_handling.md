## Error Handling
### HTTP Exceptions
To handle error showing in browser is important. Now assume you want to redirect user a 404 page or showing any HTTP status page error. You can use `Phaseolies\Support\Facades\Abort` class or `abort` global helper function.
```php
use Phaseolies\Support\Facades\Auth;

Abort::abort(404);
```

Using abort() global helper function
```php
abort(404);
```
This will show the user 404 error page. You can also use `abort_if` to add extra condition.

```php
abort_if(auth()->id() === 1, 404);
```

### abort() with custom message and headers
You can pass your custom error message and headers as the second and third arguments in abort() function as follows
```php
abort(500, 'Server Error' , ['Header Key' => 'Header Value']);
```

This will throw `500` errors with `Server Error` client message including your providing headers.

### Custom HTTP Error Pages
If you want to use a custom error page, you need to create your error view file inside the `resources/views/errors` directory. For example, to create a custom 404 error page, you would name the file `404.blade.php`, where `404` represents the HTTP status code. Once created, Doppar will automatically use your custom error page for the corresponding status code.

