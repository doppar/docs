## Encryption
### Introduction
Doppar's encryption services provide a simple, convenient interface for encrypting and decrypting text via PHP's OpenSSL using `AES-256` and `AES-128` encryption. All of Doppar's encrypted values are signed using a message authentication code (MAC) so that their underlying value cannot be modified or tampered with once encrypted.

Before using Doppar's encrypter, you must set the key configuration option in your `config/app.php` configuration file. This configuration value is driven by the `APP_KEY` environment variable. You should use the php pool `key:generate` command to generate this variable's value since the `key:generate` command will use PHP's secure random bytes generator to build a cryptographically secure key for your application. Typically, the value of the `APP_KEY` environment variable will be generated for you during Doppar's installation.


### Encrypting a Value
You can use `Phaseolies\Support\Facades\Crypt` facades to encrypt a string.

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

        $encrypted // This is now encrypted
    }
}
```

### Decrypting a Value
You can use `Phaseolies\Support\Facades\Crypt` facades to decrypt a string.
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

        $encrypted // This is now decrypted
    }
}
```