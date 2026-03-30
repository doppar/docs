---
title: Setting Up Trusted Proxies in Doppar
description: Setting Up Trusted Proxies in Doppar page
meta:
  - name: keywords
    content: Setting Up Trusted Proxies in Doppar
---

## Setting Up Trusted Proxies in Doppar
When deploying your Doppar application behind a reverse proxy or load balancer—especially one that handles TLS/SSL termination—you might find that the application fails to generate secure (`https://`) URLs, defaulting instead to plain `http://`. This typically occurs because requests are being forwarded over port 80, causing the framework to assume the original request was insecure.

To resolve this, Doppar allows you to explicitly trust your proxy servers. This ensures accurate detection of the original request scheme, host, and client IP.

In `App\Http\Kernel.php`, locate the `$middleware` array. Ensure that the `App\Http\Middleware\TrustProxies` middleware is uncommented:
```php
/**
 * The application's global HTTP middleware stack.
 *
 * These middleware are run during every request to your application.
 *
 * @var array
 */
public array $middleware = [
    \App\Http\Middleware\TrustProxies::class,
];
```

## Define Your Trusted Proxies
Within the TrustProxies middleware file (typically located at `App\Http\Middleware\TrustProxies`), set your trusted proxy IP addresses:
```php
/**
 * The trusted proxies for this application.
 *
 * @var array|string|null
 */
protected $proxies = [
    '192.168.1.1', // Example: internal IP of your load balancer
    '10.0.0.0/8',  // CIDR range for internal network,
    '127.0.0.1'    // For localhost testing
];
```
You can also use `'*'` to trust all proxies, though this is not recommended in production for security reasons.

## Verifying Trusted Proxies with cURL
Once you've configured your trusted proxies in the Doppar framework, it's important to validate that your application correctly identifies requests originating through those proxies.

Make sure the TrustProxies middleware is active and correctly configured with your trusted IPs or CIDR ranges. Now create a route like as follows
```php
use Phaseolies\Support\Facades\Route;
use Phaseolies\Http\Request;

Route::get('verify_proxies', function (Request $request) {
    return [
        'trusted_proxies' => $request::getTrustedProxies(),
        'client_ip' => $request->getClientIp(),
        'forwarded_for' => $request->headers->get('X-Forwarded-For'),
        'is_from_trusted_proxy' => $request->isFromTrustedProxy(),
        'all_headers' => $request->headers->all()
    ];
});
```

You can simulate this behavior locally using the curl command now by sending a custom `X-Forwarded-For` header as follows.
```bash
curl -H "X-Forwarded-For: 203.0.113.45, 10.1.2.3" http://example.com/verify_proxies
```

Now if you run this curl command, you should see the response like that
```json
{
  "trusted_proxies": [
    "192.168.1.1",
    "10.0.0.0/8",
    "127.0.0.1"
  ],
  "client_ip": "203.0.113.45",
  "forwarded_for": "203.0.113.45, 10.1.2.3",
  "is_from_trusted_proxy": true,
  "all_headers": {
    "host": [
      "localhost:8000"
    ],
    "user-agent": [
      "curl/7.81.0"
    ],
    "accept": [
      "*/*"
    ],
    "x-forwarded-for": [
      "203.0.113.45, 10.1.2.3"
    ]
  }
}
```
This check is especially useful during staging or local development when validating proxy behavior, ensuring features like HTTPS redirects, IP filtering, and geo-detection work correctly behind a load balancer or reverse proxy.

> If you are using AWS Elastic Load Balancing, the headers value should be `Request::HEADER_X_FORWARDED_AWS_ELB`. If your load balancer uses the standard Forwarded header from [RFC 7239](https://www.rfc-editor.org/rfc/rfc7239#section-4), the headers value should be `Request::HEADER_FORWARDED`. For more information on the constants that may be used in the headers value, check out Symfony's documentation on [trusting proxies](https://symfony.com/doc/current/deployment/proxies.html).