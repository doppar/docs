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

## String Validation Rules
Doppar offers a variety of built-in string validation rules that make it easy to enforce common patterns such as alphabetic characters, numeric strings, email formats, and more. Each rule can be combined with others to create flexible and powerful validation logic tailored to your application's needs.
### Alpha
Validates that the field's value contains only alphabetic characters `(A–Z, a–z)`. Numbers, spaces, symbols, and punctuation will all fail this rule.
```php
$request->sanitize([
    'first_name' => 'required|alpha'
]);
```
"John" will pass, but "John123" will fail because it contains numbers. "John Doe" will also fail because it contains a space, and "John_Doe" will fail because it contains an underscore.

### Alpha Numeric
Validates that the field's value contains only letters and numbers `(A–Z, a–z, 0–9)`. Spaces, dashes, underscores, and special characters are not allowed.
```php
$request->sanitize([
    'username' => 'required|alpha_num'
]);
```
"john123" and "John" will both pass, but "john_123" will fail because it contains an underscore, and "john 123" will fail because it contains a space.

### Alpha Dash
Validates that the field's value contains only letters, numbers, dashes `(-)`, and underscores `(_)`. Spaces and other special characters are not allowed.
```php
$request->sanitize([
    'handle' => 'required|alpha_dash'
]);
```

"my-handle" and "my_handle_01" will both pass, but "my handle" will fail because it contains a space, and "my@handle" will fail because @ is not a permitted character.

### Slug
Validates that the field's value is a valid URL slug — only lowercase letters, numbers, and single hyphens between words are allowed. The value must not start or end with a hyphen, and consecutive hyphens are not permitted.
```php
$request->sanitize([
    'post_slug' => 'required|slug'
]);
```

"my-blog-post" and "post-2025" will pass, but "My-Blog-Post" will fail because it contains uppercase letters. "-my-post" will fail because it starts with a hyphen, "my--post" will fail because of consecutive hyphens, and "my post" will fail because it contains a space.

### String
Validates that the field's value is a PHP string type. Any other PHP type — integers, arrays, or booleans — will fail this rule.
```php
$request->sanitize([
    'bio' => 'required|string'
]);
```

"Hello world" will pass, and even an empty string "" will pass the type check (use required alongside to disallow empty values). However, 123 will fail because it is an integer, and ["a", "b"] will fail because it is an array.

### Uppercase
Validates that the field's value consists entirely of uppercase letters. At least one uppercase letter must be present. Numbers and symbols are permitted alongside uppercase letters, but any lowercase letter will cause validation to fail.
```php
$request->sanitize([
    'country_code' => 'required|uppercase'
]);
```
"BD" and "USD" will both pass, but "Bd" will fail because it contains a lowercase d, and "bd" will fail because the entire value is lowercase.

### Lowercase
Validates that the field's value consists entirely of lowercase letters. At least one lowercase letter must be present. Numbers and symbols are permitted alongside lowercase letters, but any uppercase letter will cause validation to fail.
```php
$request->sanitize([
    'email_prefix' => 'required|lowercase'
]);
```
"john" and "john123" will both pass, but "John" will fail because it contains an uppercase J, and "JOHN" will fail because the entire value is uppercase.

### Starts With
Validates that the field's value begins with a specified prefix string. The check is case-sensitive.
```php
$request->sanitize([
    'reference_code' => 'required|starts_with:REF-'
]);
```
"REF-00123" will pass, but "ref-00123" will fail because the prefix check is case-sensitive. "INV-00123" will fail because it uses a different prefix entirely, and "00123" will fail because there is no prefix at all.

### Ends With
Validates that the field's value ends with a specified suffix string. The check is case-sensitive.
```php
$request->sanitize([
    'domain' => 'required|ends_with:.com'
]);
```
"example.com" will pass, but "example.COM" will fail because the suffix check is case-sensitive. "example.net" will fail because the suffix does not match, and "example" will fail because there is no suffix at all.

### Size
Validates that the field's value is exactly the specified size. For string values, size checks the character count. For numeric values, size checks that the value equals the given number exactly — not the digit count, but the numeric value itself.
```php
$request->sanitize([
    'otp_code'   => 'required|size:6',
    'fixed_rate' => 'required|numeric|size:5'
]);
```

For a string field with `size:6`, `"928471"` will pass because it is exactly 6 characters, but `"9284"` will fail because it is only 4 characters, and `"92847123"` will fail because it is 8 characters. For a numeric field with `size:5`, the value 5 will pass because it equals 5, but the value 10 will fail because 10 does not equal 5.

## Numeric & Digit Validation Rules
Doppar provides flexible validation options to handle different numeric scenarios. Whether you need to accept any numeric value (including strings that represent numbers) or enforce stricter formats like integers or specific digit lengths, these rules help you define clear boundaries for valid input.
### Numeric
Validates that the field's value is a valid number — this includes integers, floats, and numeric strings. If you need to enforce integers only or floats with decimal constraints, use the int or float rules instead.
```php
$request->sanitize([
    'price' => 'required|numeric'
]);
```
"42", "3.14", and the integer 42 will all pass, but "abc" will fail because it is not a number, and "12abc" will fail because it is a mixed string.

### Digits
Validates that the field's value is a numeric string containing exactly N digits. This rule is strict — decimal points, negative signs, and non-digit characters are not allowed. Use this for fixed-length codes such as PINs, OTPs, or verification codes.
```php
$request->sanitize([
    'pin' => 'required|digits:4'
]);
```
"1234" will pass with digits:4, but "123" will fail because it has only 3 digits, and "12345" will fail because it has 5. "12.4" will fail because it contains a decimal point, and "12a4" will fail because it contains a non-digit character.

### Min Digits
Validates that the field's value is a numeric string containing at least N digits. Decimal points, signs, or non-digit characters are not allowed — the value must be a pure sequence of digits.
```php
$request->sanitize([
    'phone_number' => 'required|min_digits:7'
]);
```
"01712345678" (11 digits) and "1234567" (exactly 7 digits) will both pass with `min_digits:7`, but "123456" will fail because it only has 6 digits. "123-4567" will also fail because it contains a non-digit character.

### Max Digits
Validates that the field's value is a numeric string containing no more than N digits. Like `min_digits`, the value must be a pure digit sequence with no decimal points or special characters.
```php
$request->sanitize([
    'zip_code' => 'required|max_digits:10'
]);
```
"1234" and "1234567890" (exactly 10 digits) will both pass with max_digits:10, but "12345678901" will fail because it has 11 digits. "123.456" will also fail because it contains a decimal point.

## Boolean & Type Validation
Boolean and type validation rules ensure that incoming data matches the expected data type, which is critical for maintaining predictable application behavior. These rules are especially useful when handling form inputs, API payloads, or configuration flags where values may come in different formats but need to be interpreted consistently.
### Boolean
Validates that the field's value is a boolean or boolean-like value. This is useful when accepting toggle states, flags, or checkbox inputs where the underlying type may vary across form and API submissions.

The following values are all considered valid: true, false, 1, 0, "1", "0", "true", and "false".
```php
$request->sanitize([
    'is_active'          => 'required|boolean',
    'receive_newsletter' => 'required|boolean'
]);
```
true, "false", "1", and 0 will all pass, but "yes" will fail because it is not a recognised boolean value, and "on" will also fail for the same reason.

### Array
Validates that the field's value is a PHP array type. This is particularly relevant for API requests where a field is expected to hold multiple values such as a list of tag IDs, selected options, or permission slugs.
```php
$request->sanitize([
    'tags'        => 'required|array',
    'permissions' => 'required|array'
]);
```
[1, 2, 3] and ["admin", "editor"] will both pass. An empty array [] will also pass the type check — use required alongside to disallow empty submissions. However, "admin" will fail because it is a plain string, and "[1,2,3]" will fail because a JSON string representation of an array is not a PHP array.

## URL, IP & Network Validation
URL, IP, and network validation rules are used to ensure that input values related to web addresses and network identifiers are correctly formatted and valid. These rules are especially important when working with external resources, server configurations, APIs, or any system that relies on accurate networking data.
### URL
Validates that the field's value is a well-formed URL including the protocol `(e.g., http:// or https://)`. Bare domain names without a protocol will fail.
```php
$request->sanitize([
    'website' => 'required|url'
]);
```
"https://example.com" and "http://sub.example.com/path?q=1" will both pass, but "example.com" will fail because it is missing the protocol. "not a url" will also fail.

### IP Address
Validates that the field's value is a valid IP address in either IPv4 or IPv6 format. Use this when you need to accept either format without restriction. If you need to enforce a specific version, use ipv4 or ipv6 instead.
```php
$request->sanitize([
    'server_ip' => 'required|ip'
]);
```
"192.168.1.1" (IPv4) and "2001:0db8:85a3::8a2e:0370:7334" (IPv6) will both pass, but "999.999.999.999" will fail because it is out of the valid IPv4 range, and "not-an-ip" will fail entirely.

### IPv4
Validates that the field's value is a valid IPv4 address specifically. IPv6 addresses will be rejected. A valid IPv4 address consists of four octets separated by dots, each ranging from 0 to 255.
```php
$request->sanitize([
    'server_ip' => 'required|ipv4'
]);
```
"192.168.0.1", "0.0.0.0", and "255.255.255.255" will all pass, but "2001:db8::1" will fail because IPv6 addresses are not accepted, and "192.168.0" will fail because it is an incomplete address.

### IPv6
Validates that the field's value is a valid IPv6 address specifically. IPv4 addresses will be rejected.
```php
$request->sanitize([
    'server_ip' => 'required|ipv6'
]);
```
"2001:0db8:85a3:0000:0000:8a2e:0370:7334", "::1" (loopback), and "fe80::1" (link-local) will all pass, but "192.168.1.1" will fail because IPv4 addresses are not accepted under this rule.

## Format & Pattern Validation
Format and pattern validation rules are designed to ensure that input values follow a specific structure or encoding standard. These rules are especially useful when working with structured data, identifiers, or inputs that must conform to strict formatting requirements.
### JSON
Validates that the field's value is a valid JSON-encoded string. The value must be of string type and must successfully parse as JSON. Use this for endpoints that accept serialised configuration, metadata, or structured payloads as a string.
```php
$request->sanitize([
    'metadata' => 'required|json'
]);
```

'{"key":"value"}', '[1,2,3]', and '"simple string"' will all pass because they are valid JSON. However, '{"key":}' will fail because it is malformed JSON, 'hello' will fail because it is not valid JSON, and the integer 123 will fail because the value must be a string type.

### UUID
Validates that the field's value is a valid UUID conforming to versions 1 through 5 of the UUID specification. The expected format is `xxxxxxxx-xxxx-Mxxx-Nxxx-xxxxxxxxxxxx` where `M` is the version digit (1–5). Matching is case-insensitive.
```php
$request->sanitize([
    'product_id' => 'required|uuid'
]);
```

"550e8400-e29b-41d4-a716-446655440000" and "6ba7b810-9dad-11d1-80b4-00c04fd430c8" will both pass, but "not-a-uuid" will fail entirely, and "550e8400e29b41d4a716446655440000" will fail because the required dashes are missing.

### Phone
Validates that the field's value is a valid phone number. The rule accepts an optional leading `+` for international format, followed by digits, spaces, dashes, and parentheses. The total length must be between 7 and 20 characters.
```php
$request->sanitize([
    'phone' => 'required|phone'
]);
```

"+8801712345678", "01712345678", "(017) 1234-5678", and "+880 17 1234 5678" will all pass. However, "123" will fail because it is too short (under 7 characters), and "phone#123" will fail because # is not a permitted character.

### Regex
Validates that the field's value matches a custom regular expression pattern you define. This is the most flexible validation rule — use it when none of the built-in rules cover your specific format requirement. The pattern must be a valid PHP-compatible regular expression including delimiters.
```php
$request->sanitize([
    'postal_code'    => 'required|regex:/^[0-9]{4}$/',
    'vehicle_number' => 'required|regex:/^[A-Z]{2}-\d{4}$/'
]);
```
For r`egex:/^[0-9]{4}$/`, "1234" will pass but "12345" will fail because it has 5 digits. For `regex:/^[A-Z]{2}-\d{4}$/`, "AB-1234" will pass but "ab-1234" will fail because the regex expects uppercase letters.

> Always include regex delimiters `(e.g., /pattern/)` and test your pattern thoroughly before using it in production, as an invalid regex will silently cause validation to fail.

### Date Format
Validates that the field's value matches a specific PHP date format string exactly. Unlike the general date rule which accepts a wide range of date strings, date_format requires the value to conform precisely to the format you specify.
```php
$request->sanitize([
    'appointment_date' => 'required|date_format:Y-m-d',
    'log_timestamp'    => 'required|date_format:Y-m-d H:i:s'
]);
```

For date_format:Y-m-d, "2025-04-15" will pass, but "15-04-2025" will fail because the order is wrong, and "2025/04/15" will fail because the separator is incorrect. For date_format:Y-m-d H:i:s, "2025-04-15 10:30:00" will pass but "2025-04-15 10:30" will fail because the seconds component is missing.

## Comparison Validation Rules
Comparison validation rules are used to validate relationships between multiple fields within the same request. These rules are essential when you need to ensure consistency, confirmation, or distinction between related inputs, such as passwords, email addresses, or other paired values.
### Same As
Validates that the field's value exactly matches the value of another field in the same request. The comparison is strict and case-sensitive. This is most commonly used for password confirmation.
```php
$request->sanitize([
    'password'         => 'required|min:8',
    'password_confirm' => 'required|same_as:password'
]);
```

If password is "secret123", then password_confirm with the same value "secret123" will pass. However, "Secret123" will fail because the comparison is case-sensitive, and "secret" will fail because the values do not match.

### Confirmed
Works similarly to `same_as`, but instead of specifying the comparison field explicitly, Doppar automatically looks for a field named `{fieldname}_confirmation` in the request. If the field being validated is `password`, Doppar will automatically look for a `password_confirmation` field. Both fields must contain identical values.
```php
$request->sanitize([
    'password' => 'required|min:8|confirmed'
]);
```
If password is "mypassword" and `password_confirmation` is also "mypassword", validation will pass. It will fail if `password_confirmation` contains "myPassword" because the comparison is case-sensitive, and it will also fail if the `password_confirmation` field is absent from the request entirely.

### Different
Validates that the field's value differs from the value of another specified field. Use this to prevent users from submitting identical values in two fields that should be distinct, such as a new email address that must differ from the current one on file.
```php
$request->sanitize([
    'new_email' => 'required|email|different:current_email'
]);
```
If current_email is "old@example.com", then new_email with value "new@example.com" will pass. However, if new_email is also "old@example.com", validation will fail because both fields hold the same value.

## List Validation Rules
List validation rules are used to control whether a field’s value belongs to a predefined set of acceptable or restricted options. These rules are particularly useful when you want to limit user input to known values, such as statuses, roles, categories, or sorting options.
### In
Validates that the field's value is one of a predefined set of allowed values. The comparison is strict and case-sensitive.
```php
$request->sanitize([
    'status'     => 'required|in:active,inactive,pending',
    'sort_order' => 'required|in:asc,desc'
]);
```

"active" and "pending" will both pass for the status field, but "deleted" will fail because it is not in the allowed list. "Active" will also fail because the comparison is case-sensitive.

### Not In
Validates that the field's value is not present in a given list of disallowed values. Use this to block reserved values, forbidden roles, or restricted keywords from being submitted.
```php
$request->sanitize([
    'role'     => 'required|not_in:superadmin,root',
    'username' => 'required|not_in:admin,administrator,system'
]);
```
"editor" will pass for the role field because it is not in the disallowed list, but "root" will fail because it is explicitly blocked. Similarly, "admin" will fail for the username field because it appears in the disallowed values.

## Timezone Validation
Timezone validation rules ensure that a given value is a valid and recognized timezone identifier based on the IANA (Internet Assigned Numbers Authority) timezone database. This is important for applications that rely on accurate date and time calculations across different regions.
### Timezone
Validates that the field's value is a valid IANA timezone identifier. Doppar checks the submitted value against PHP's official list of timezone identifiers, ensuring the timezone can be reliably used for date and time operations in your application.
```php
$request->sanitize([
    'user_timezone' => 'required|timezone'
]);
```
"Asia/Dhaka", "America/New_York", "Europe/London", and "UTC" will all pass. However, "Asia/Dhaka123" will fail because it is not a valid identifier, "GMT+6" will fail because offset notation is not a valid IANA identifier, and "dhaka" will fail because it does not follow the correct format.
> To browse all valid IANA timezone identifiers, refer to PHP's DateTimeZone::listIdentifiers() or the [IANA Time Zone Database](https://www.iana.org/time-zones).