---
title: Validation
description: Doppar Validation page
meta:
  - name: keywords
    content: Validation
---

## Validation
### Introduction
Doppar offers multiple flexible ways to validate incoming data in your application. The most common approach is using the `sanitize()` method available on all HTTP request instances, making it easy to apply validation directly where data enters your system.

Beyond this, Doppar provides a robust set of built-in validation rules that you can apply to ensure your data meets expected formats and constraints. This includes powerful features like checking for unique values within database tables.

In this section, we'll explore all the available validation rules and techniques, so you're fully equipped to handle any data validation scenario in your application.

## Validation Quickstart
To get a clear view of how Doppar’s validation system works, let’s walk through a full example of validating a form and returning error messages to the user. This overview will help you build a strong foundational understanding of how to validate incoming request data efficiently.

In Doppar, validating form data is clean and straightforward. Whether you're working with API inputs or web forms, the process ensures that only properly formatted and safe data reaches your application logic.

Here’s what we’ll cover:
- Defining validation rules
- Validating the request data
- Handling and displaying error messages

By the end of this example, you'll have a practical understanding of Doppar's validation workflow and how to integrate it into your own applications effectively.

## Creating the Controller
Now let's take a look at a simple controller that handles incoming requests to these routes. We'll leave the store method empty for now:
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Utilities\Attributes\Route;
use Phaseolies\Http\Request;

class RegisterController extends Controller
{
    /**
     * Show the form to create a new user
     */
    #[Route(uri: 'register')]
    public function index()
    {
        return view('auth.register');
    }

    /**
     * Store a new user.
     */
    #[Route(uri: 'register', methods:['POST'])]
    public function store(Request $request)
    {
        // Validate and store the user
    }
}
```

## Writing the Validation Logic
Now, let’s implement the logic for validating a new user in our store method. To do this, we’ll use the `sanitize()` method available through the `Phaseolies\Http\Request` object. This method helps ensure that incoming data meets the specified rules.

If the data passes validation, the method continues executing as expected. If validation fails, an exception will be thrown, and the appropriate error response will be automatically sent back to the user.

For traditional HTTP requests, Doppar will redirect the user back to the previous page with validation errors. In the case of an XHR (AJAX) request, Doppar will return a JSON response containing the validation errors.

Let’s take a closer look at how to use the validate() method in the store method:
```php
#[Route(uri: 'register', methods:['POST'])]
public function store(Request $request)
{
    $sanitized = $request->sanitize([
        'name' => 'required|min:2|max:20',
        'email' => 'required|email|unique:users|min:2|max:50',
        'password' => 'required|min:2|max:20',
    ]);

    // The requested data is valid...
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

$data; // validation passed data
$request->passed(); // validation passed data

// Safely create the user now
User::create($data);
// Or
User::create($request->passed());
```

## Conditionally Sanitize Request Data
The `sanitizeIf()` method allows you to apply sanitization rules only if a specific condition is met. This is useful for applying stricter cleaning or validation logic in certain environments (like development vs. production) or based on dynamic runtime context.
```php
$request->sanitizeIf(!app()->isProduction(), [
    'excerpt' => 'required|min:100',
]);
```
In this example, the excerpt field will only be sanitized using the provided rules if the app is not in production.

## Show Validation Error Message
To show validation error message in your Odo file, doppar has a very elegent syntax. Showing validation error message specific key wise. So, in our example, the user will be redirected to our controller's create method when validation fails, allowing us to display the error messages in the view:
```html
#errors
    <div class="alert alert-danger">
        <ul>
            #foreach (session()->pull('errors') as $messages)
                #foreach ($messages as $message)
                    <li>[[ $message ]]</li>
                #endforeach
            #endforeach
        </ul>
    </div>
#enderrors
```

To show validation error message in your Odo file, doppar has a very elegent syntax. Showing validation error message specific key wise
```html
#error('email')
    <div class="alert alert-danger mt-1 p-1">[[ $message ]]</div>
#enderror
```
Doppar will automatically trace the error message and display here.

### Customizing the Error Messages
Doppar's built-in validation rules each have an error message that is located in your application's `lang/en/validation.php` file. You can customize error message from this file as you want.

## XHR Requests and Validation
In this example, we've demonstrated using a traditional form to send data to the application. However, many modern applications handle XHR requests from JavaScript-driven frontends. When using the `sanitize()` method during an XHR request, Doppar behaves differently compared to traditional form submissions.

Instead of generating a redirect response, Doppar will send a JSON response containing all of the validation errors. This response will be returned with a `422 HTTP status` code to indicate that the request was well-formed but contained invalid data that is unprocessable entities.

## Repopulating Forms
To retrieve flashed input from the previous request, invoke the old method on an instance of `Phaseolies\Http\Request`. The old method will pull the previously flashed input data from the session:
```html
<input type="text" name="name" value="[[ old('name') ]]">
```
## Validation Facades

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

## Validation Using Form Request Class
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
## Exists In Validation
To validate if a value exists in a specific database table, you follow this example
```php
$request->sanitize([
    'category_id' => 'required|exists_in:category,id'
]);
```
The above validation will be applied like that, the `category_id` field is required and must exist in the `id` column of the `category` table.

You can also skip the column name, by default it will use id as the column name
```php
$request->sanitize([
    'category_id' => 'required|exists_in:category'
]);
```
The above validation will be applied like that, it will check if the given `category_id` exists in the id column of the `category` table.

## Image validation
In Doppar, you can use the `sanitize()` method to validate and sanitize incoming request data. This ensures that the submitted data meets specific criteria before being processed. To validate an uploaded file, use the following code:

```php
$request->sanitize([
    'file' => 'required|image|mimes:jpg,png,jpeg|dimensions:min_width=100,min_height=100,max_width=1000,max_height=1000|max:2048'
]);
```
The following validation rules are applied to image uploads to ensure that only properly formatted and sized images are processed.
| **Rule**                | **Description**                                                                                                               |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| **required**            | Ensures that a file is provided in the upload request. If no file is uploaded, validation fails.                              |
| **image**               | Verifies that the uploaded file is an image by checking its MIME type (e.g., jpeg, png, gif).                                 |
| **mimes\:jpg,png,jpeg** | Restricts the accepted image file types specifically to JPEG (jpg, jpeg) and PNG (png). Other image formats will be rejected. |
| **dimensions**          | Applies validation on the image dimensions (width and height).                                                                |
|   **min\_width=100**    | Requires the image to have a minimum width of 100 pixels. If less, validation fails.                                          |
|   **min\_height=100**   | Requires the image to have a minimum height of 100 pixels. If less, validation fails.                                         |
|   **max\_width=1000**   | Ensures the image width does not exceed 1000 pixels. If wider, validation fails.                                              |
|   **max\_height=1000**  | Ensures the image height does not exceed 1000 pixels. If taller, validation fails.                                            |
| **max:2048**            | Limits the maximum file size to 2048 kilobytes (2MB). Files larger than this will be rejected by validation.                  |

## Validation Failure:

If the uploaded file fails to meet any of these criteria, the validation process will fail. An error response will be generated, indicating the specific validation failures.

## Date Validation
Doppar offers powerful date validation rules that help ensure the correct format and logical consistency when working with date-related fields.

Here are the key rules and how to use them:
```php
'date' => 'required|date|gte:today'
```

`required` Ensures the date field is present in the requested data and Validates that the field is a valid date. `gte:today` Ensures the date is greater than or equal to today’s date.


### Greater Than Today
`gt:today` Ensures the date is greater than today, i.e., a future date.
```php
'date' => 'required|date|gt:today'
```

### Less Than or Equal to Today
`lte:today` Ensures the date is less than or equal to today.
```php
'date' => 'required|date|lte:today'
```

### Less Than Today
lt:today: Ensures the date is less than today, i.e., a past date.
```php
'date' => 'required|date|lt:today'
```

### Date Validation Summary
| **Rule**       | **Description**                                |
| -------------- | ---------------------------------------------- |
| **gte\:today** | Date must be greater than or equal to today.   |
| **gt\:today**  | Date must be greater than today (future date). |
| **lte\:today** | Date must be less than or equal to today.      |
| **lt\:today**  | Date must be less than today (past date).      |

## Number Validation
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
