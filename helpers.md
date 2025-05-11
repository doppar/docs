## Helpers
### Introduction
Doppar offers a rich set of global helper functions designed to streamline common tasks throughout your application. These helpers encapsulate repetitive logic and provide a more expressive, readable syntax for tasks like working with arrays, strings, URLs, paths, and more.

While many of these functions are used internally by the framework itself, they are fully accessible within your application and can significantly simplify your code when used appropriately.

From accessing configuration values to generating routes and handling authentication, Doppar's helpers make your development workflow faster and cleaner.

### env()
The `env()` function in Doppar is used to retrieve environment variables from the application's configuration. It allows you to define environment-specific settings in a `.env` file and access them throughout your application. If the specified variable is not found, you can provide a default value as a fallback.
```php
$databaseHost = env('DB_HOST', 'localhost');
```
In this example, `DB_HOST` is fetched from the environment file, and if it's not set, 'localhost' is returned as the default.

### app()
The `app()` function provides access to the service container instance in thouht it return the Application instance of Doppar. It can be used to resolve dependencies, retrieve bound services, or access the container itself.
```php
$app = app(); // This returns the global Application instance.
```
But if your pass a class like

```php
$logger = app(Logger::class); // This fetches an instance of Logger.
```

You can Resolve a service with parameters as well

```php
$service = app(MyService::class, ['param1' => 'value']);
```
This retrieves `MyService` while passing custom parameters. This function simplifies dependency injection, making it easier to manage and access services within the application.

### resolve()
The `resolve()` function is a global helper provided by Doppar to retrieve an instance of a class or service from the service container. It’s a convenient shorthand for calling `app()->make()`.

#### Signature
```php
resolve(string $name, array $parameters = []): mixed
```

#### Basic Example
```php
use App\Services\PaymentService;

$paymentService = resolve(PaymentService::class);
$paymentService->charge(100);
```
Here, PaymentService will be automatically instantiated by the container, and any dependencies it needs will be resolved and injected.

#### Example With Parameters
If your class constructor requires parameters that aren’t container-resolvable, you can pass them manually:
```php
use App\Services\ReportService;

$reportService = resolve(ReportService::class, [
    'reportType' => 'monthly',
    'userId' => 42,
]);
```

### url()
The `url()` function is a convenient helper for creating a new instance of the UrlGenerator class, which is responsible for handling urls in Doppar. By calling this function, you can easily manage urls.
```php
$url = url('/home');

// http://example.com/home
```

You can use this url() as an object like
```php
 return url()
    ->to('/profile')
    ->withQuery('foo=bar&baz=qux') // you can pass here array also like ['foo' => 'bar', 'baz' => 'qux']
    ->withFragment('about')
    ->withSignature(3600)
    ->setSecure(true)
    ->make();

// it will generate url like that
// http://example.com/profile?foo=bar&baz=qux&expires=1742923201&signature=2cd1656d3557f64a433de7dcb01abbb64c2dc9daa85983b8dad88f8ac732a935#about
```

You can also generate url by passing parameters like
```php
url()->to('profile')->make(); // http://example.com/profile

url('profile', ['id' => 123]); http://example.com/profile?id=123
```

### request()
The `request()` function is a convenient helper for creating a new instance of the Request class, which is responsible for handling HTTP requests in Doppar. By calling this function, you can easily access the incoming request data, such as GET, POST parameters, headers, cookies, and more.
```php
$request = request();
```
This returns a new instance of the Request class, allowing you to interact with the current HTTP request. To access the request data, such as retrieving a query parameter:

```php
$name = request()->query('name');

$name = request('name', 'default'); // Equivalent to previous one
```

It works as an object or a function.This function simplifies the process of working with HTTP requests by providing direct access to the request object, streamlining data retrieval and manipulation.


### response()
The `response()` function in Doppar is a helper used to generate and return HTTP response instances. It provides a flexible way to create a response, whether it’s with content or just an empty response, and allows setting the HTTP status code and headers.
```php
return response('Hello, World!', 200, ['Content-Type' => 'text/plain']);
```
Create a response without content (just a status and headers):
```php
return response(null, 404);
```
Create a response factory when no arguments are passed:
```php
$responseFactory = response();
```
If no arguments are passed, the function returns a ResponseFactory instance, which can be used to generate responses later.

### view()
The `view()` function in Doppar is a helper used to render a view with the given data and return a response containing the rendered content. It simplifies the process of rendering views and sending them as HTTP responses, while also allowing you to add custom headers if needed.
```php
return view('home', ['name' => 'John']);
```
This renders the home view, passing the data array ['name' => 'John'] to the view and returning a response with the rendered content. as third parameters, it accept headers also
```php
return view('welcome', ['user' => 'Alice'], ['X-Custom-Header' => 'Value']);
```
This function provides a simple way to return rendered views as HTTP responses, while also giving you the flexibility to set custom headers and pass data to the views.

### redirect()
The `redirect()` function in Doppar is a helper for creating and returning HTTP redirects. It simplifies the process of redirecting the user to a different URL, optionally with a specific HTTP status code, headers, and security (HTTPS) settings.
```php
return redirect('/home');
```
Redirect with headers and security (HTTPS):
```php
return redirect('/profile', 302, ['X-Redirect-Notice' => 'Redirecting...'], true);
```
Get the redirect instance without redirection (factory usage):
```php
$redirect = redirect();
```
If no arguments are provided, it returns the RedirectResponse instance, which can be used to perform redirects later. This helper function streamlines the process of handling HTTP redirects in doppar application, providing flexibility for status codes, headers, and security options.

### back()
The `back()` function in Doppar is a helper used to create a redirect response to the previous location (the referring page), which is commonly used for scenarios like redirecting a user back after a form submission or an action that requires them to return to where they were.
```php
return back();
```
This redirects the user back to the previous location using the default status code (302). A RedirectResponse instance, which will redirect the user either to the previous location or the fallback location.
```php
// Redirect back with default 302 status
return back();

// Redirect back with a custom status code and headers
return back(301, ['Cache-Control' => 'no-store']);

// Redirect back with a fallback URL if the referer is not available
return back(302, [], '/home');
```
### session()
The `session()` function in Doppar is a helper used to interact with the session data. It provides a convenient way to retrieve, set, or manage session data. This helper simplifies working with sessions by abstracting some of the underlying complexities and making session data access more straightforward.
```php
$value = session('user_id');
```
This retrieves the value stored in the session under the key 'user_id'. 
```php
// Retrieve session value for 'user_id'
$userId = session('user_id');

// Set session value for 'user_id'
session('user_id', 123);

// Retrieve session value with a default fallback
$username = session('user_name', 'Guest');

// Store multiple session values at once
session(['user_id' => 123, 'user_name' => 'John']);

// Access the entire session instance
$session = session();
```

You can also store array data in session like
```php
session([
    'user_id' => 123,
    'username' => 'doppar',
    'is_admin' => true
]);

$userId = session('user_id');
```

### cookie()
The `cookie()` function is a helper designed to interact with HTTP cookies in Doppar. It allows you to retrieve, create, and manage cookies within your application. Cookies are used to store small pieces of data on the user's browser, such as session identifiers, authentication tokens, or user preferences.
#### Example
```php
// Access the entire cookie instance
$cookie = cookie();

// Store cookie values
cookie()->store('cookie_name', 'cookie_value');

// Get cookie value
cookie()->get('cookie_name', 'default');
```

### csrf_token()
The `csrf_token()` function is a helper designed to fetch the CSRF (Cross-Site Request Forgery) token from the session. CSRF tokens are used to protect forms from malicious attacks by ensuring that the request originates from the intended user and not from a potential attacker.
```php
$token = csrf_token();
```
### bcrypt()
The `bcrypt()` function is a helper designed to hash a plain text password using the Bcrypt hashing algorithm. This function is commonly used in authentication systems to store passwords securely by converting them into a hashed value that cannot be easily reversed.
```php
$hashedPassword = bcrypt('myPlainTextPassword');
```
### old()
The `old()` function is a helper used to retrieve the old input value for a given key from the session. This is particularly useful in scenarios like form validation, where you want to repopulate form fields with the user's previously entered data after a validation failure.
```php
$value = old('email');
```
This will return the previously entered value for the input field with the key `email`, or null if no old value is found.
#### Example
```html
// Example in a form input field
<input type="text" name="email" value="{{ old('email') }}">

// If the user previously entered 'mahedi' and the form was submitted with errors,
// the input field will be repopulated with 'mahedi' after the validation failure.
```

### fake()
The `fake()` function is a helper designed to create and return an instance of the Faker generator, which is used to generate fake data. This can be particularly useful for seeding a database with random data, testing, or generating sample data for development purposes.
```php
$faker = fake();
$name = $faker->name;
$email = $faker->email;
```
### route()
The `route()` function is a helper used to generate a full URL for a named route. It allows you to easily create links to specific routes in your application based on their name and any parameters they might require.
```php
$url = route('route.name', ['parameter1' => 'value1']);
```
This will return the full URL for the named route route.name, replacing any required parameters with the provided values.
#### Example
```php
// If you have a route named 'file' with a parameter 'id' in your routes file:
Route::get('/file/1', [FileController::class, 'index'])->name('file');

$url = route('file',1);
// The result might be something like 'http://example.com/file/1'
```

### config()
The `config()` function is a helper used to retrieve configuration values by key. It allows you to access values from the configuration files in your application, providing a centralized way to manage various settings.
```php
$value = config('app.name');
```
This retrieves the value associated with the `app.name` configuration key.
#### Example
If you have a configuration file `config/app.php` with the following content:
```php
return [
    'name' => 'My Application',
    'env' => 'local',
    'timezone' => 'UTC',
];
```
You can retrieve the value like this:
```php
$timezone = config('app.timezone', 'America/New_York'); // 'UTC'
```
If the key doesn’t exist, and you provide a default value:

### is_auth()
The `is_auth()` function is a helper used to check if a user is authenticated (i.e., logged in). It allows you to easily determine whether the current user has been authenticated through the application's authentication system.
```php
if (is_auth()) {
    // The user is logged in
} else {
    // The user is not logged in
}
```
### base_path()
The `base_path()` function is a helper used to retrieve the base path of the application, with an optional path appended to it. It provides an easy way to get the root directory of your application, and optionally append additional subdirectories or files to the base path.
```php
// Get the base path of the application
$basePath = base_path(); // /var/www/html/doppar

// Get the base path with a specific subdirectory or file
$fullPath = base_path('storage/logs'); // /var/www/html/doppar/storage/logs
```
### base_url()
The `base_url()` function is a helper that retrieves the base URL of the application, with an optional path appended to it. This function is used to generate the full URL to your application, considering both the environment (e.g., local development, production) and any given subpath.
```php
// Get the base URL of the application
$baseUrl = base_url();

// Get the full URL with a specific path
$fullUrl = base_url('images/logo.png'); // http://example.com/images/logo.png
```

### storage_path()
The `storage_path()` function is a helper that retrieves the storage path of the application, with an optional path appended to it. This function is used to generate the full path to the storage directory of your application, allowing you to easily access or store files in a consistent manner.
```php
// Get the base storage path
$storagePath = storage_path();

// Get the full storage path with a specific subdirectory or file
$fullPath = storage_path('uploads/images/logo.png');
```

### public_path()
The `public_path()` function is a helper that retrieves the public path of the application, with the option to append a specific path to it. This function is useful for determining the full path to the public directory of your application, enabling easier access to publicly accessible resources, such as assets, uploaded files, and other public files.
```php
// Get the base public path
$publicPath = public_path();

// Get the full public path with a specific subdirectory or file
$fullPath = public_path('images/logo.png');
```

### resource_path()
The `resource_path()` function is a helper that retrieves the application's resource path, with an optional ability to append a specific subdirectory or file path. This function is particularly useful when dealing with application resources such as views, language files, and configuration files that are stored in the resources directory.
```php
// Get the base resources path
$resourcesPath = resource_path();

// Get the full resources path with a specific subdirectory or file
$fullPath = resource_path('views/layouts/app.blade.php');
```

### config_path()
The `config_path()` function is a helper that retrieves the configuration path of the application, with an optional path appended to it. This function is used to generate the full path to the configuration directory of your application, allowing you to easily access or store configuration files in a consistent manner.
```php
// Get the base config path
$configPath = config_path();

// Get the full config path with a specific subdirectory or file
$fullPath = config_path('app.php');
```

### database_path()
The `database_path()` function is a helper designed to retrieve the application's database path, with the ability to append a specific subdirectory or file path if necessary. This function is particularly useful for accessing the database configuration, migration files, and other database-related files that are stored within the database directory.
```php
// Get the base database path
$databasePath = database_path();

// Get the full database path with a specific subdirectory or file
$fullPath = database_path('migrations/');
```
### enqueue()
The `enqueue()` function is a helper designed to generate the URL for assets (such as CSS, JavaScript, images, etc.) located in the public directory of the application. This function allows you to generate a full URL for assets, either with or without a secure (HTTPS) scheme, depending on the provided parameters.
```php
// Generate the URL for a public asset (e.g., CSS, JS, image)
$assetUrl = enqueue('assets/css/styles.css');

// Generate the URL for a public asset with a secure (HTTPS) scheme
$secureAssetUrl = enqueue('assets/js/app.js', true);
```
Generating a Secure (HTTPS) URL for a JavaScript File: If you want to load a JavaScript file over HTTPS:
```php
$jsUrl = enqueue('assets/js/app.js', true);
```

### abort()
The `abort()` function is a helper that allows you to immediately stop the current request and return an HTTP response with a specific status code and an optional message. This is particularly useful for handling error scenarios or preventing further processing when a certain condition is met.
```php
// Abort with a 404 Not Found status and an optional message
abort(404, 'Page not found');

// Abort with a 500 Internal Server Error status and a custom message
abort(500, 'Something went wrong on the server');
```
### abort_if()
The `abort_if()` function is a helper designed to automatically abort the request and send a specific HTTP status code with an optional message when a condition is true. It provides a shorthand way to handle conditional error situations and terminate the request early if a specific condition is met.
```php
// Abort the request if a condition is true
abort_if($userNotAuthenticated, 401, 'Unauthorized access');

// Abort the request if a file does not exist
abort_if(!file_exists($filePath), 404, 'File not found');
```

### Log Helpers
These are helper functions designed to simplify logging at different levels of severity. They utilize Doppar's built-in Log facade to log messages with various log levels. Each function accepts a payload (which can be any type of data) and logs it accordingly at the specified level.
```php
info('This is an info message.'); // Logs an info-level message
warning('This is a warning message.'); // Logs a warning-level message
error('This is an error message.'); // Logs an error-level message
alert('This is an alert message.'); // Logs an alert-level message
notice('This is a notice message.'); // Logs a notice-level message
emergency('This is an emergency message.'); // Logs an emergency-level message
critical('This is a critical message.'); // Logs a critical-level message
debug('This is a debug message.'); // Logs a debug-level message
```
### collect()
The `collect()` function is a helper designed to create a new instance of a Doppar Collection. It simplifies the process of creating and working with collections in your application. Collections allow you to work with arrays in a more expressive and fluent way, providing additional methods for filtering, transforming, and manipulating data.
```php
$collection = collect([1, 2, 3, 4]); // Returns a Collection instance containing [1, 2, 3, 4]

$emptyCollection = collect(); // Returns an empty []
```
#### Example Usage
See some basic example of collection, how you can use it

### filter()
The `filter()` method allows you to filter a collection based on a given condition.

```php
$collection = collect([1, 2, 3, 4, 5, 6]);

// Filter to get only even numbers
$evenNumbers = $collection->filter(function ($value) {
    return $value % 2 === 0;
});

return $evenNumbers; // [2,4,6]
```
### map()
The map() method allows you to transform each item in the collection using a callback function.
```php
$collection = collect([1, 2, 3, 4]);

// Multiply each number by 2
$doubledNumbers = $collection->map(function ($value) {
    return $value * 2;
});

return $doubledNumbers;
```
### first()
The first() method retrieves the first item in the collection.
```php
// Create a collection of numbers
$collection = collect([10, 20, 30, 40]);

// Get the first item
$firstItem = $collection->first();

echo $firstItem; // Output: 10
```

### delete_folder_recursively()
The `delete_folder_recursively()` function is a helper designed to delete a folder and all its contents, including subfolders and files. It performs a recursive deletion, ensuring that every file and folder inside the target folder is deleted before the folder itself is removed.
```php
// Delete a folder and its contents recursively
$result = delete_folder_recursively('/path/to/folder'); // Returns true on success, false on failure
```
### uuid()
The `uuid()` function is a helper designed to generate a UUID v4 (Universally Unique Identifier), which is a random 128-bit value represented as a string. UUIDs are commonly used for generating unique identifiers in distributed systems, databases, and APIs.
```php
// Generate a UUID v4 string
$uuid = str()->uuid();
// or
$uuid = uuid(); // Returns something like "f47ac10b-58cc-4372-a567-0e02b2c3d479"
```


## String Helper function
Doppar provides a collection of global string helper functions in PHP. While many of these functions are integral to the framework’s internal operations, they are also available for use in your own projects whenever they prove usefull. All the string function, you can access it via global `str()` function or you can use `Phaseolies\Support\Facades\Str` facades

### mask()
The `mask()` function is a helper designed to mask parts of a given string while keeping a specified number of characters visible at the start and the end. This can be useful for hiding sensitive information (like credit card numbers, emails, or phone numbers) while displaying a portion of it for the user to verify.
```php
use Phaseolies\Support\Facades\Str;

// Mask a credit card number except for the last four digits
$maskedCard = str()->mask('1234 5678 9876 5432', 4, 4, '*');
$maskedCard = Str::mask('1234 5678 9876 5432', 4, 4, '*');

// Mask a phone number except for the first three and last four digits
$maskedPhone = str()->mask('123-456-7890', 3, 4, '#');
$maskedPhone = Str::mask('123-456-7890', 3, 4, '#');
```
### Example
Masking Credit Card Numbers: To display only the last 4 digits of a credit card number for security reasons, you can use the mask() function:
```php
$maskedCard = str()->mask('1234 5678 9876 5432', 4, 4); // Output: 1234 ******** 5432
```
Masking Phone Numbers: For masking a phone number except for the first 3 and last 4 digits:
```php
$maskedPhone = str()->mask('123-456-7890', 3, 4, '#'); // Output: 123-###-7890
```
Masking Email Addresses: To hide part of an email address except for the first and last characters:
```php
$maskedEmail = str()->mask('john.doe@example.com', 1, 1); // Output: j********e.com
```

### truncate()
The `truncate()` function is a helper designed to truncate a string to a specified maximum length. If the string exceeds this length, it appends a suffix (such as ellipsis ...) to indicate that the string has been shortened.
```php
$shortDescription = str()->truncate('This is a long description that should be shortened for previews.', 30, '... Read More');
$shortDescription = Str::truncate('This is a long description that should be shortened for previews.', 30, '... Read More');
// Output: "This is a long description... Read More"
```
### snake()
The `snake()` function is a helper designed to convert a camelCase string into a snake_case string. This is useful when you need to transform variable names, keys, or identifiers from camelCase (often used in programming) into snake_case (commonly used in database column names or URL routing).
```php
// Convert a camelCase string to snake_case
$snakeString = str()->snake('camelCaseString'); // Output: 'camel_case_string'
```
### camel()
The `camel()` function is a helper designed to convert a snake_case string into a camelCase string. This is useful when you need to transform variable names, keys, or identifiers from snake_case (commonly used in databases or file names) into camelCase (often used in programming languages like JavaScript and PHP for variable names).
```php
// Convert a snake_case string to camelCase
$camelString = str()->camel('snake_case_string'); // Output: 'snakeCaseString'
```

### random()
The `random()` function is a helper designed to generate a random alphanumeric string of a specified length. This is useful when you need to generate secure random tokens, passwords, or unique identifiers.
```php
// Generate a random alphanumeric string of default length 16
$randomString = str()->random(); // Output: 'a1B2c3D4e5F6g7H8'

// Generate a random alphanumeric string of a custom length (e.g., 8)
$randomString = str()->random(8); // Output: 'Xy7GzH8Q'
```
### isPalindrome()
The `isPalindrome()` function is a helper designed to check whether a given string is a palindrome. A palindrome is a word, phrase, or sequence that reads the same backward as forward (ignoring spaces, punctuation, and capitalization).
```php
// Check if a string is a palindrome
$isPalindrome = str()->isPalindrome('racecar'); // Output: true

// Check if a string is not a palindrome
$isPalindrome = str()->isPalindrome('hello'); // Output: false
```
### countWord()
The `countWord()` function is a helper designed to count the number of words in a given string. This is useful when you need to determine the word count of a sentence, paragraph, or any text input.
```php
// Count the number of words in a string
$wordCount = str()->countWord('This is a test sentence.'); // Output: 5

// Count the number of words in a string with punctuation
$wordCount = str()->countWord('Hello, world!'); // Output: 2
```

### title()
The `title()` function is a helper designed to convert a given string into title case, where the first letter of each word is capitalized, and the rest of the letters are in lowercase. This is commonly used for formatting titles or headings.
```php
// Convert a string to title case
$title = str()->title("hello world"); // Returns "Hello World"
```

### slug()
The `slug()` function is a helper designed to generate a URL-friendly slug from a given string. A slug is typically used in URLs, where spaces and special characters are replaced with hyphens (or another separator), and all letters are converted to lowercase.
```php
// Generate a URL-friendly slug
$slug = str()->slug("Hello World!"); // Returns "hello-world"
```

The word separator used in the slug. By default, this is a hyphen (-), but you can specify another separator if needed.
```php
// Generate a URL-friendly slug with the default separator (hyphen)
$slug = str()->slug("Hello World!"); // Returns "hello-world"

// Generate a URL-friendly slug with a custom separator (underscore)
$slug = str()->slug("Hello World!", '_'); // Returns "hello_world"
```

### contains()
The `contains()` function is a helper designed to check if a given string (the haystack) contains another string (the needle), performing a case-insensitive search.
```php
// Check if a string contains another string (case-insensitive)
$contains = str()->contains("Hello World", "world"); // Returns true
```
### limitWords()
The `limitWords()` function is a helper designed to limit the number of words in a string. If the string contains more words than the specified limit, the function truncates the string and appends an optional ending suffix (such as ...).
```php
// Limit the number of words in a string
$truncatedString = str()->limitWords("This is a test string", 3); // Returns "This is a..."
```

The function returns the truncated string with a specified number of words and the optional suffix. If the string has fewer words than the specified limit, it remains unchanged.
```php
$truncatedString = str()->limitWords("This is a test string", 3); // Returns "This is a..."

$truncatedString = str()->limitWords("Hello there, how are you?", 2, '...more'); // Returns "Hello there...more"
```

### removeWhiteSpace()
The `removeWhiteSpace()` function is a helper designed to remove all whitespace characters (spaces, tabs, newlines, etc.) from a given string. This is useful when you need to clean up strings for processing or formatting purposes
```php
$cleanedString = str()->removeWhiteSpace("Hello   World"); // Returns "HelloWorld"

$cleanedString = str()->removeWhiteSpace("Hello \tWorld\n"); // Returns "HelloWorld"
```

## startsWith()
The `startsWith()` function is a helper designed to check if a given string (the haystack) starts with another string (the needle). This check is case-sensitive.
```php
$startsWith = str()->startsWith("Hello World", "Hello"); // Returns true
$startsWith = str()->startsWith("Hello World", "world"); // Returns false (case-sensitive)
```

### endsWith()
The `endsWith()` function is a helper designed to check if a given string (the haystack) ends with another string (the needle). This check is case-sensitive.
```php
$endsWith = str()->endsWith("Hello World", "World"); // Returns true
$endsWith = str()->endsWith("Hello World", "world"); // Returns false (case-sensitive)
```

### studly()
The `studly()` function is a helper designed to convert a given string into StudlyCase (also known as PascalCase), where the first letter of each word is capitalized, and there are no spaces or underscores between words.
```php
$studlyString = str()->studly("hello_world"); // Returns "HelloWorld"

$studlyString = str()->studly("convert_this_string_to_studly_case"); // Returns "ConvertThisStringToStudlyCase"
```

### reverse()
The reverse() function is a helper designed to reverse the characters in a given string while correctly handling multi-byte characters (such as characters in non-Latin alphabets or special symbols). This ensures that characters are reversed properly, without corrupting multi-byte sequences.
```php
$reversedString = str()->reverse("Hello World"); // Returns "dlroW olleH"
```

### extractNumbers()
The `extractNumbers()` function is a helper designed to extract all numeric digits from a given string. It removes any non-numeric characters, leaving only the digits.
```php
$numbers = str()->extractNumbers("Hello 123, World 456!"); // Returns "123456"

$numbers = str()->extractNumbers("Price: $123.45, Discount: 10%"); // Returns "1234510"
```

### longestCommonSubstring()
The `longestCommonSubstring()` function is a helper designed to find the longest common substring shared between two given strings. A common substring is a contiguous sequence of characters that appears in both strings.
```php
$commonSubstring = str()->longestCommonSubstring("hello world", "yellow world"); // Returns "llo world"

$commonSubstring = str()->longestCommonSubstring("abcdef", "xyz"); // Returns ""

$commonSubstring = str()->longestCommonSubstring("banana", "bandana"); // Returns "bana"
```

### leetSpeak()
The `leetSpeak()` function is a helper designed to convert a given string into leetspeak (1337), a playful encoding style where certain letters are replaced with similar-looking numbers or symbols.
```php
$leetString = str()->leetSpeak("leet speak"); // Possible output: "l33t sp34k"

$leetString = str()->leetSpeak("programming is fun"); // Possible output: "pr0gr4mm1ng 15 fun"
```

Common Leetspeak Replacements:
```
A -> 4   B -> 8   C -> (   E -> 3   G -> 6   H -> #
I -> 1   L -> 1   O -> 0   S -> 5   T -> 7   Z -> 2
```
This function is great for fun text transformations, gaming usernames, or encoding messages in a way that is still somewhat readable.

### extractEmails()
The `extractEmails()` function is a helper designed to extract all email addresses from a given string. It scans the string for valid email formats and returns them as an array.
```php
$emails = str()->extractEmails("Contact us at support@example.com or sales@example.org");
// Returns: ["support@example.com", "sales@example.org"]
```

### highlightKeyword()
The `highlightKeyword()` function is a helper designed to highlight all occurrences of a specified keyword in a given string using HTML tags. This is useful for emphasizing search results, user input matches, or key terms in displayed content.
```php
$text = "Doppar is awesome. Learn Doppar now!";
$highlighted = str()->highlightKeyword($text, "Doppar"); 
// Returns: "<strong>Doppar</strong> is awesome. Learn <strong>Doppar</strong> now!"

$highlighted = str()->highlightKeyword($text, "Doppar", "span class='highlight'"); 
// Returns: "<span class='highlight'>Doppar</span> is awesome. Learn <span class='highlight'>Doppar</span> now!"
```