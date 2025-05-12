## Starter kits
To give you a head start building your new Doppar application, we are happy to offer application starter kits. These starter kits give you a head start on building your next Doppar application, and include the layouts, and extending views you need to start developmen.

This is the base layout file that your pages will extend. It defines the overall structure of your HTML document, including areas for dynamic title, styles, content, and scripts.

```html
layout/app.blade.php
<!DOCTYPE html>
<html lang="en">
    <head
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>@yield('title')</title>
        @section('style')
          // This for loading ondemand page css
        @show
    </head>
    <body>
        <div class="container mt-4">
            <div class="row justify-content-center">
                <div class="col-md-8">
                    @yield('content')
                </div>
            </div>
        </div>
        @section('script')
          // This for loading ondemand page script
        @show
    </body>
</html>
```

###  Extending the Layout
Here is a view file that extends the base layout and fills in the dynamic sections: title, styles, content, and scripts.
```php
@extends('layouts.app')
@section('title') Home Page
@section('style')
 // Load css
@append
@section('content')
    <div class="card shadow-lg mb-4">
        <div class="card-header bg-primary text-white">
            <h5 class="mb-0">Dashboard</h5>
        </div>
        <div class="card-body">
            <p class="fw-bold fs-5">
                Howdy
            </p>
        </div>
    </div>
@endsection
@section('script')
 // Load script
@append
```