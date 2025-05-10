## CSRF Protectection
Cross-site request forgery (CSRF) attacks are a type of security threat where unauthorized actions are executed on behalf of an authenticated user without their knowledge. Fortunately, Doppar provides robust built-in protection to safeguard your application against such attacks.

You can access the current session's CSRF token either through the request's session data or by using the `csrf_token()` helper function. This seamless integration ensures that your application remains secure while requiring minimal effort on your part. With Doppar, you can focus on building your application with confidence, knowing that CSRF protection is handled efficiently in the background.

Without CSRF protection, a malicious website could create an HTML form that points to your application's `/user/email` route and submits the malicious user's own email address:
```php
<form action="https://example.com/user/email" method="POST">
    <input type="email" value="malicious-email@example.com">
</form>

<script>
    document.forms[0].submit();
</script>
```

To prevent this vulnerability, we need to inspect every incoming `POST`, `PUT`, `PATCH`, or `DELETE` request for a secret session value that the malicious application is unable to access.

### Preventing CSRF Requests
Doppar simplifies CSRF protection by automatically generating a unique CSRF token for every active user session. This token acts as a secure identifier to ensure that requests made to the application are genuinely coming from the authenticated user. The token is stored in the user's session and is regenerated whenever the session is refreshed, making it virtually impossible for malicious actors to replicate or misuse it.

```php
use Phaseolies\Http\Request;

Route::get('/token', function (Request $request) {
    $token = $request->session()->token();

    $token = csrf_token();

    // ...
});
```

Anytime you define a "POST" HTML form in your application, you should include a hidden CSRF `_token` field in the form so that the CSRF protection middleware can validate the request. For convenience, you may use the @csrf Blade directive to generate the hidden token input field:
```php
<form method="POST" action="/profile">
    @csrf

    <!-- Equivalent to... -->
    <input type="hidden" name="_token" value="{{ csrf_token() }}" />
</form>
```
The `\Phaseolies\Middleware\CsrfTokenMiddleware`, middleware, which is included in the web middleware group by default, will automatically verify that the token in the request input matches the token stored in the session.

#### X-CSRF-TOKEN
In addition to checking for the CSRF token as a POST parameter, the `\Phaseolies\Middleware\CsrfTokenMiddleware` middleware, which is included in the web middleware group by default, will also check for the X-CSRF-TOKEN request header. You could, for example, store the token in an HTML meta tag:
```php
<meta name="csrf-token" content="{{ csrf_token() }}">
```
Then, you can instruct a library like jQuery to automatically add the token to all request headers. This provides simple, convenient CSRF protection for your AJAX based applications using legacy JavaScript technology:
```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```
