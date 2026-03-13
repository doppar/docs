---
title: Encryption
description: Doppar Encryption page
meta:
  - name: keywords
    content: Encryption
---

## Encryption
### Introduction
Doppar's encryption services provide a simple, convenient interface for encrypting and decrypting text via PHP's OpenSSL using `AES-256` and `AES-128` encryption. All of Doppar's encrypted values are signed using a message authentication code (MAC) so that their underlying value cannot be modified or tampered with once encrypted.

Before using Doppar's encrypter, you must set the key configuration option in your `config/app.php` configuration file. This configuration value is driven by the `APP_KEY` environment variable. You should use the php pool `key:generate` command to generate this variable's value since the `key:generate` command will use PHP's secure random bytes generator to build a cryptographically secure key for your application. Typically, the value of the `APP_KEY` environment variable will be generated for you during Doppar's installation.


## Encrypting a Value
To encrypt a value in Doppar, you can use the Crypt facade, which provides a simple and secure way to encrypt strings. Here's an example of how to encrypt a value in a controller:

You can use `encrypt` method to encrypt a string.

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use Phaseolies\Support\Facades\Crypt;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $encrypted = Crypt::encrypt("Hello World");
    }
}
```

## Decrypting a Value
To decrypt a value in Doppar, you can use the Crypt facade, just like how you encrypt data. The `decrypt()` method provided by `Phaseolies\Support\Facades\Crypt` allows you to safely decrypt a previously encrypted string.

You can use `decrypt` method to decrypt a string.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;
use Phaseolies\Support\Facades\Crypt;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $decrypted = Crypt::decrypt($encrypted);
    }
}
```

## Global Helper Functions
Doppar also provides convenient global helper functions for encryption and decryption, allowing you to encrypt and decrypt values without importing the Crypt facade.

### `encrypt()`
The global `encrypt()` function provides a quick way to encrypt strings without using the Crypt facade:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $encrypted = encrypt("Hello World");
    }
}
```

### `decrypt()`
The global `decrypt()` function provides a quick way to decrypt strings without using the Crypt facade:

```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Http\Request;

class UserController extends Controller
{
    public function store(Request $request)
    {
        $decrypted = decrypt($encrypted);
    }
}
```

Both global helper functions work identically to their Crypt facade counterparts, using the same encryption key and providing the same level of security. They are particularly useful for quick encryption/decryption operations where importing the facade would be unnecessary overhead.