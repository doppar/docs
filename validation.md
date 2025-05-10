## Validation
### Introduction
Doppar offers multiple flexible ways to validate incoming data in your application. The most common approach is using the `sanitize()` method available on all HTTP request instances, making it easy to apply validation directly where data enters your system.

Beyond this, Doppar provides a robust set of built-in validation rules that you can apply to ensure your data meets expected formats and constraints. This includes powerful features like checking for unique values within database tables.

In this section, we'll explore all the available validation rules and techniques, so you're fully equipped to handle any data validation scenario in your application.

### Validation Quickstart
To get a clear view of how Doppar’s validation system works, let’s walk through a full example of validating a form and returning error messages to the user. This overview will help you build a strong foundational understanding of how to validate incoming request data efficiently.

In Doppar, validating form data is clean and straightforward. Whether you're working with API inputs or web forms, the process ensures that only properly formatted and safe data reaches your application logic.

Here’s what we’ll cover:
- Defining validation rules
- Validating the request data
- Handling and displaying error messages

By the end of this example, you'll have a practical understanding of Doppar's validation workflow and how to integrate it into your own applications effectively.

### Defining the Routes
First, let's assume we have the following routes defined in our `routes/web.php` file:
```php
<?php

use Phaseolies\Support\Facades\Route;
use App\Http\Controllers\RegisterController;

Route::get('register', [RegisterController::class, 'index']);
Route::post('register', [RegisterController::class, 'store']);
```

The `GET` route will display a form for the user to create a new new user, while the POST route will store the new user in the database.

### Creating the Controller
Now let's take a look at a simple controller that handles incoming requests to these routes. We'll leave the store method empty for now:
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\RedirectResponse;
use Phaseolies\Http\Request;

class RegisterController extends Controller
{
    /**
     * Show the form to create a new user
     */
    public function index()
    {
        return view('auth.register');
    }

    /**
     * Store a new user.
     */
    public function store(Request $request): RedirectResponse
    {
        // Validate and store the user

        return back();
    }
}
```

### Writing the Validation Logic
Now, let’s implement the logic for validating a new user in our store method. To do this, we’ll use the `sanitize()` method available through the `Phaseolies\Http\Request` object. This method helps ensure that incoming data meets the specified rules.

If the data passes validation, the method continues executing as expected. If validation fails, an exception will be thrown, and the appropriate error response will be automatically sent back to the user.

For traditional HTTP requests, Doppar will redirect the user back to the previous page with validation errors. In the case of an XHR (AJAX) request, Doppar will return a JSON response containing the validation errors.

Let’s take a closer look at how to use the validate() method in the store method:
```php
/**
 * Store a new blog post.
 */
public function store(Request $request): RedirectResponse
{
    $validated = $request->sanitize([
        'name' => 'required|min:2|max:20',
        'email' => 'required|email|unique:users|min:2|max:50',
        'password' => 'required|min:2|max:20',
    ]);

    // The requested data is valid...
    // Create the user
    return redirect('/dashboard');
}
```

Have a look on this validation code, let's explore. Here’s a table summarizing the validation rules for each field (email, password, and name) using the `sanitize()` method:

| **Field**    | **Validation Rule** | **Description**  |
| ------------ | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| **email**    | `required`, `email`, `unique:users`, `between:2,100` | - Must be present. <br> - Must be a valid email format. <br> - Must be unique in the `users` table. <br> - Must be between 2 and 100 characters. |
| **password** | `required`, `between:2,20`                           | - Must be present. <br> - Must be between 2 and 20 characters.                                                                                   |                                   |
| **name**     | `required`, `between:2,20`                           | - Must be present. <br> - Must be between 2 and 20 characters.                                                                                   |

Upon successful validation, the sanitized and validated data is stored in the `$validated` variable. You can also get failed and passed data from the request.
```php
use App\Models\User;

$data = $request->sanitize([
    'email' => 'required|email|unique:users|min:2|max:100',
    'password' => 'required|min:2|max:20',
    'name' => 'required|min:2|max:20'
]);

$data // validation passed data
$request->passed(); // validation passed data

// Safely create the user now
User::create($request->passed());
```

### Show Validation Error Message
To show validation error message in your blade file, doppar has a very elegent syntax. Showing validation error message specific key wise. So, in our example, the user will be redirected to our controller's create method when validation fails, allowing us to display the error messages in the view:
```html
 @errors
    <div class="alert alert-danger">
        <ul>
            @foreach (session()->get('errors') as $field => $messages)
                @foreach ($messages as $message)
                    <li>{{ $message }}</li>
                @endforeach
            @endforeach
        </ul>
    </div>
@enderrors
```

To show validation error message in your blade file, doppar has a very elegent syntax. Showing validation error message specific key wise
```html
<input .... class="form-control @error('email') is-invalid @enderror value="{{ old('email') }}">

@error('email')
    <div class="alert alert-danger mt-1 p-1">{{ $message }}</div>
@enderror
```
Doppar will automatically trace the error message and display here.

### Customizing the Error Messages
Doppar's built-in validation rules each have an error message that is located in your application's `lang/en/validation.php` file. You can customize error message from this file as you want.

### XHR Requests and Validation
In this example, we've demonstrated using a traditional form to send data to the application. However, many modern applications handle XHR requests from JavaScript-driven frontends. When using the `sanitize()` method during an XHR request, Doppar behaves differently compared to traditional form submissions.

Instead of generating a redirect response, Doppar will send a JSON response containing all of the validation errors. This response will be returned with a `422 HTTP status` code to indicate that the request was well-formed but contained invalid data that is unprocessable entities.

### Repopulating Forms
To retrieve flashed input from the previous request, invoke the old method on an instance of `Phaseolies\Http\Request`. The old method will pull the previously flashed input data from the session:
```html
<input type="text" name="name" value="{{ old('name') }}">
```
### Validation Facades

Doppar provides the `Phaseolies\Support\Facades\Sanitize` facade to help sanitize and validate incoming request data efficiently. This facade gives you a fluent, easy-to-use approach to handle form validation and cleaning in one go.

Here’s an example of how to use the `Sanitize` facade to validate and sanitize a login form request:
```php
<?php

namespace App\Http\Controllers\Auth;

use Phaseolies\Http\Request;
use Phaseolies\Support\Facades\Sanitize;
use App\Http\Controllers\Controller;
use Phaseolies\Http\Response\RedirectResponse;

class LoginController extends Controller
{
    /**
     * Create the login
     */
    public function login(Request $request): RedirectResponse
    {
        // Sanitize and validate the request data
        $sanitizer = Sanitize::request($request->all(), [
            'email' => 'required|email|min:2|max:100',
            'password' => 'required|min:2|max:20'
        ]);

        // Check if validation fails
        if ($sanitizer->fails()) {
            return back()->withErrors($sanitizer->errors())->withInput();
        }

        // Validation passed, you can retrieve the sanitized data
        $validated = $sanitizer->passed();

        // Continue with your logic (e.g., authenticating the user)
    }
}
```

### Validation Using Form Request Class
We can also validate requested data using a class to make our code more clean and maintable. Doppar provides a pool command to create a new form request class.
```bash
php pool make:request LoginRequest
```

This command will create a new Request class to handle login request form data inside the `App\Http\Validations` folder. In this class, you will find two methods: `authorize()` and `rules()`. If you want to perform validation, ensure that the `authorize()` method returns true. See the example
```php
<?php

namespace App\Http\Validations;

use Phaseolies\Http\Validation\FormRequest;

class LoginRequest extends FormRequest
{
    /**
     * Determine if the user is authorized to make this request.
     *
     * @return bool
     */
    public function authorize(): bool
    {
        return true;
    }

    /**
     * Get the validation rules that apply to the request.
     *
     * @return array
     */
    public function rules(): array
    {
        return [
            'email' => 'required|email|min:2|max:100',
            'password' => 'required|min:2|max:20'
        ];
    }
}
```

Now you this `App\Http\Validations\LoginRequest` validation class in your controller like
```php
<?php

namespace App\Http\Controllers\Auth;

use App\Http\Validations\LoginRequest;
use App\Http\Controllers\Controller;

class LoginController extends Controller
{
    public function login(LoginRequest $request)
    {
        $data = $request->passed(); // validated data
    }
}
```

### Image validation
In Doppar, you can use the `sanitize()` method to validate and sanitize incoming request data. This ensures that the submitted data meets specific criteria before being processed. To validate an uploaded file, use the following code:

```php
$request->sanitize([
    'file' => 'required|image|mimes:jpg,png,jpeg|dimensions:min_width=100,min_height=100,max_width=1000,max_height=1000|max:2048'
]);
```
The following validation rules are applied to image uploads to ensure that only properly formatted and sized images are processed.
| **Rule**                                                                        | **Description**                                                                                                            |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| **required**                                                                    | Ensures that a file is provided in the upload request. If no file is uploaded, validation fails.                           |
| **image**                                                                       | Verifies that the uploaded file is an image (checks the MIME type).                                                        |
| **mimes\:jpg,png,jpeg**                                                         | Restricts the accepted image file types to JPEG (jpg, jpeg) and PNG (png). Other image formats are rejected.               |
| **dimensions\:min\_width=100,min\_height=100,max\_width=1000,max\_height=1000** | Enforces size constraints on the image: <br> - Minimum width and height of 100px <br> - Maximum width and height of 1000px |
| **max:2048**                                                                    | Limits the maximum file size to 2048 kilobytes (2MB). Files exceeding this size will be rejected.                          |

#### Validation Failure:

If the uploaded file fails to meet any of these criteria, the validation process will fail. An error response will be generated, indicating the specific validation failures.

### Date Validation
Doppar offers powerful date validation rules that help ensure the correct format and logical consistency when working with date-related fields.

Here are the key rules and how to use them:
```php
'date' => 'required|date|gte:today'
```

`required` Ensures the date field is present in the requested data and Validates that the field is a valid date. `gte:today` Ensures the date is greater than or equal to today’s date.


#### Greater Than Today
`gt:today` Ensures the date is greater than today, i.e., a future date.
```php
'date' => 'required|date|gt:today'
```

#### Less Than or Equal to Today
`lte:today` Ensures the date is less than or equal to today.
```php
'date' => 'required|date|lte:today'
```

#### Less Than Today
lt:today: Ensures the date is less than today, i.e., a past date.
```php
'date' => 'required|date|lt:today'
```

#### Date Validation Summary
| **Rule**       | **Description**                                |
| -------------- | ---------------------------------------------- |
| **gte\:today** | Date must be greater than or equal to today.   |
| **gt\:today**  | Date must be greater than today (future date). |
| **lte\:today** | Date must be less than or equal to today.      |
| **lt\:today**  | Date must be less than today (past date).      |

### Number Validation
To validate number, you follow this example
```php
$request->sanitize([
    'number' => 'null|int|between:2,5'
]);
```
The above validation will be applied like that, the number can be nullable and if number provides, it must be integer and digit must in between greater than equal 2 and less than equal 5

To validate float number, you follow this example
```php
$request->sanitize([
    'number' => 'null|float:2|between:2,5'
]);
```
The above validation will be applied like that, the number can be nullable and if number provides, it must be float and digit must in between greater than equal `2` and less than equal `5` and decimal after number will be 2 digit to "have exactly two decimal places" or "with two decimal places" like `3.33` not `3.333`



