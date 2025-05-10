## Mail
### Introduction
Doppar provides a robust and flexible mailing system that enables your application to send emails using various drivers. The `config/mail.php` configuration file manages all mail-related settings, including the default mailer, SMTP credentials, and global “from” addresses.

This file allows you to define how your application handles outgoing email by setting up one or more mailers, each with its own configuration. By default, Doppar uses the smtp mailer, but you can customize or extend this based on your requirements.

You can specify:
- The default mailer your app will use (default)
- Detailed configurations for each mailer under the mailers array
- A global sender address and name used for all emails

This configuration supports environment-based customization via .env variables, making it easy to adapt for local development, staging, or production environments.

Doppar provides a convenient way to setup your mail configuration. Doppar currently support only smtp driver for mail configuration. Need to update .env's mail configuration before starting with mail features. Doppar usage PHPMailer tp send mail.

### Mail Configuration
```markdown
MAIL_MAILER=smtp
MAIL_HOST=
MAIL_PORT=2525
MAIL_USERNAME=
MAIL_PASSWORD=
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS="hello@example.com"
MAIL_FROM_NAME="${APP_NAME}"
```

> Remember, if you update, config/mail.php file, don't forget to clear your system cache.
```bash
php pool cache:clear
php pool config:cache // to cache again
```

Now if you setup with your smtp mail credentials, now you are ready to go.

### Sending Mail
When building Doppar applications, each type of email sent by your application is represented as a "mailable" class. These classes are stored in the `app/Mail` directory. Don't worry if you don't see this directory in your application, since it will be generated for you when you create your first mailable class using the `make:mail` Pool command:
```bash
php pool make:mail InvoicMail
```

#### Configuring the Sender
You specify a global "from" address in your `config/mail.php` configuration file. This address will be used to send mail.
```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

#### Configuring the Subject
By time to time, every Mail has a subject. Doppar allows you to define a Mail subject in a very convenient way. To define Mail subject, just need to update the subject method from your mailable class.
```php
/**
 * Define mail subject
 * @return Phaseolies\Support\Mail\Mailable\Subject
 */
public function subject(): Subject
{
    return new Subject(
        subject: 'New Mail'
    );
}
```

#### Configuring the View
Within a mailable class's content method, you may define the view, or which template should be used when rendering the email's contents. Since each email typically uses a Blade template to render its contents, you have the full power and convenience of the Blade templating engine when building your email's HTML:
```php
/**
 * Set the message body and data
 * @return Phaseolies\Support\Mail\Mailable\Content
 */
public function content(): Content
{
    return new Content(
        view: 'Optional view.name'
    );
}
```

If you want to pass the data without views, you can pass string or array data.

```php
public function content(): Content
{
    return new Content(
        data: [
           'order_status' => true
        ]
    );
}
```

Even you can send mail without passing any data to Content. Suppose you just want to pass attachment only. you can do in this case, just make content empty.
```php
public function content(): Content
{
    return new Content();
}
```

### Complete Example of Sending Mail
Doppar provides to and send method primaritly to send a basic mail. You can use `Phaseolies\Support\Facades\Mail` call to handle mail functionalities.
```php
<?php

namespace App\Http\Controllers;

use Phaseolies\Support\Facades\Mail;
use App\Models\User;
use App\Http\Controllers\Controller;
use App\Mail\InvoiceMail;

class OrderController extends Controller
{
    public function index()
    {
        $user = User::find(1);

        $data = [
            'order_status' => 'success',
            'invoice_no' => '123-123'
        ];

        Mail::to($user)->send(new InvoiceMail($data));

        // or you can send mail by passing only mail address
        Mail::to('recipient@example.com')->send(new InvoiceMail($data));

        // also by passing name as the second argument
        Mail::to('recipient@example.com', 'recipient_name')->send(new InvoiceMail($data));
    }
}
```

Now update your `InvoiceMail` mailable class like
```php
<?php

namespace App\Mail;

use Phaseolies\Support\Mail\Mailable\Subject;
use Phaseolies\Support\Mail\Mailable\Content;
use Phaseolies\Support\Mail\Mailable;

class InvoiceMail extends Mailable
{
    public function __construct(protected $data) {}

    public function subject(): Subject
    {
        return new Subject(
            subject: "Order Shipped Confirmation"
        );
    }

    public function content(): Content
    {
        return new Content(
            view: 'emails.order.invoice', // 'resources/views/emails/order/invoice.blade.php'
            data: $this->data // Passing data will be available in invoice.blade.php, access it via {{ $data }}
        );
    }

    public function attachment(): array
    {
        return [];
    }
}
```

### Sending Mail with Attachment
To send mail with attachment, you have to pass data attachment path using attachment method.

```php
public function attachment(): array
{
    return [
        storage_path('invoice.pdf') => [
            'as' => 'rename_invoice.pdf', // The file will be sent using this name
            'mime' => 'application/pdf',  // file mime types
        ]
    ];
}
```

### Multiple Attachments with Mime Types
You can also send mail with multiple attachment. Just pass your file arrays in the attachment method like
```php
public function attachment(): array
{
    return [
        storage_path('invoice_1.pdf') => [
            'as' => 'invoice_3.pdf',
            'mime' => 'application/pdf',
        ],
        storage_path('invoice_2.pdf') => [
            'as' => 'invoice_3.pdf',
            'mime' => 'application/pdf',
        ],
        storage_path('invoice_3.pdf') => [
            'as' => 'invoice_3.pdf',
            'mime' => 'application/pdf',
        ],
    ];
}
```
Here both `as` and `mime` is optional, you can simply send email attachment like
```php
public function attachment(): array
{
    return [
        storage_path('invoice_1.pdf'),
        storage_path('invoice_2.pdf'),
        storage_path('invoice_3.pdf')
    ];
}
```

### Sending Mail with CC and BCC
You are not limited to just specifying the "to" recipients when sending a message. You are free to set "to", "cc", and "bcc" recipients by chaining their respective methods together:
```php
use Phaseolies\Support\Mail\Mail;

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

