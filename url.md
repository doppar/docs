## URL Generation
Doppar provides several helpers to assist you in generating URLs for your application. These helpers are primarily helpful when building links in your templates, or when generating redirect responses to another part of your application. The most basic URL generation will be like assume we want to generate url using route name.

### Generating URLs
Doppar provides a convenient url helper function that can be used to generate full URLs within your application. This helper automatically uses the scheme (HTTP or HTTPS) and host from the current request context.
```php
use App\Models\Post;

$post = Post::find(1);

echo url("/posts/{$post->id}");

// http://example.com/posts/1
```

You can use `to()` function to generate url like
```php

use Phaseolies\Support\Facades\URL;

URL::to('products')->make(); // using URL facades
url()->to('products')->make(); // using helper function

// http://example.com/products
```

To generate a URL with query string parameters, you may use the query method:
```php
return url()->to('products', ['category' => 'electronics', 'page' => 2])->make();

// http://example.com/products?category=electronics&page=2
```

### Secure URL
To generate a secure (HTTPS) URL, Doppar provides support through the UrlGenerator class. By enabling the secure flag, all generated URLs will use the `https://` scheme:

```php
url()->to('checkout')->make();

// https://example.com/checkout
```

### With Fragment
You can append a fragment (hash) to the end of a URL using `withFragment()`:
```php
url()->to('about')->withFragment('team')->make();

// http://localhost:8000/about#team
```

### Signed URL
Generate a temporary, signed URL (useful for secure actions like downloads or password resets):
```php
url()->signed('download', ['file' => 'report.pdf'], 3600);

// http://example.com/download?file=report.pdf&expires=...&signature=...
```
You can also create this above URL using
```php
url()->to('/download')
    ->withQuery(['file' => 'report.pdf'])
    ->withSignature(3600)
    ->withFragment('about')
    ->make();

// output
http://localhost:8000/download?file=report.pdf&expires=1742401435&signature=363ef0e47fd9fca7197882490ee8f4c132df6b9b6e9e0041ac0df5c31cc349d3#about
```

### Route Generation
Use named routes to generate URLs dynamically:
```php
url()->route('user.profile', ['id' => 123]);

// http://example.com/users/123/profile
```

### Current URL
Retrieve the full URL of the current request:
```php
url()->current();

// http://example.com/current-path
```

### Full URL with Query
Get the full current URL including query parameters:
```php
url()->full();

// http://example.com/current-path?with=query
```

### Changing Security Setting
Toggle between HTTP and HTTPS programmatically:
```php
url()->to('login')->setSecure(true)->make();

// Output: https://example.com/login
```

### URL Validation
Check if a given string is a valid URL:
```php
url()->isValid('https://valid.com'); // true
url()->isValid('invalid url');       // false
```

### Accessing public assets
Doppar provides enqueue() method to access your public assstes like
```php
url()->enqueue('/assets/example.png');

// http://example.com/assets/example.png
```

### Get Base URL
To get base URL, you can call `base()` method like
```php
url()->base();

// http://example.com
```

### URL Facades
You can generate all the above URL using `Phaseolies\Support\Facades\URL` facades. Doppar provides both ways to generate custom URL. Let's see an example of generating URL using URL facades.
```php
use Phaseolies\Support\Facades\URL;

URL::to('/download')
    ->withQuery(['file' => 'report.pdf'])
    ->withSignature(3600)
    ->withFragment('about')
    ->make();
```
This fluent API makes it easy to build complex URLs cleanly and consistently.