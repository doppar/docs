---
title: Mail
description: Doppar Mail page
meta:
  - name: keywords
    content: Mail
---

## Mail

### Introduction

Doppar provides a robust and flexible mailing system using [Symfony Mailer](https://symfony.com/doc/current/mailer.html) that enables your application to send emails using various transports. The `runtime/config/mail.php` configuration file manages all mail-related settings, including the default mailer, SMTP credentials, and global "from" address.

This file allows you to define how your application handles outgoing email by setting up one or more mailers, each with its own configuration. By default, Doppar uses the `smtp` mailer, but you can customize or extend this based on your requirements.

You can specify:
- The default mailer your app will use (`default`)
- Detailed configuration for each mailer under the `mailers` array
- A global sender address and name used for all emails
- Optional DKIM/S-MIME signing applied to every outgoing message

This configuration supports environment-based customization via `env.toml` variables, making it easy to adapt for local development, staging, or production environments.

Because Symfony Mailer is DSN-driven, Doppar isn't limited to one transport the way earlier versions were. Out of the box you get `smtp`, `sendmail`, and `null`, plus `failover`/`roundrobin` composition of any of them, and any third-party provider bridge (Mailgun, SES, Postmark, Brevo, SendGrid, ...) once its package is installed.

## Mail Configuration

To enable email sending in your Doppar application, you need to configure the mail settings in your `env.toml` file. Here's a typical setup using an SMTP mailer:

```toml
MAIL_MAILER = "smtp"
MAIL_HOST = ""
MAIL_PORT = 2525
MAIL_USERNAME = ""
MAIL_PASSWORD = ""
MAIL_ENCRYPTION = "tls"
MAIL_FROM_ADDRESS = "hello@example.com"
MAIL_FROM_NAME = "Doppar"
```

Once you fill in your SMTP credentials, you're ready to go. If you'd rather hand Symfony a single connection string instead of separate host/port/username/password fields, set `MAILER_DSN` — it takes priority over everything else:

```toml
MAILER_DSN = "smtp://user:pass@smtp.mailgun.org:587"
```

`runtime/config/mail.php` looks like this:

```php
'default' => env('MAIL_MAILER', 'smtp'),

'mailers' => [
    'smtp' => [
        'dsn' => env('MAILER_DSN'),
        'host' => env('MAIL_HOST', '127.0.0.1'),
        'port' => env('MAIL_PORT', 2525),
        'username' => env('MAIL_USERNAME'),
        'password' => env('MAIL_PASSWORD'),
        'encryption' => env('MAIL_ENCRYPTION', 'tls'),
        'local_domain' => env('MAIL_EHLO_DOMAIN', parse_url(env('APP_URL', 'http://localhost'), PHP_URL_HOST)),
        'timeout' => env('MAIL_TIMEOUT'),
    ],

    'sendmail' => [
        'dsn' => env('MAIL_SENDMAIL_DSN', 'sendmail://default'),
    ],

    'null' => [
        'dsn' => 'null://null',
    ],
],
```

`encryption` controls how the connection is secured:

| Value | Effect |
|---|---|
| `ssl` | Implicit TLS from the first byte (typically port 465). |
| `tls` (default) | STARTTLS — Symfony upgrades the connection automatically when the server offers it. |
| `null` / empty | Plaintext, no TLS negotiation at all. |

`local_domain` is sent as the EHLO/HELO domain during the SMTP handshake, and `timeout` is the socket connect/read timeout in seconds — both are optional.

## Sending Mail

When building Doppar applications, each type of email sent by your application is represented as a "mailable" class. These classes are stored in the `app/Mail` directory. Don't worry if you don't see this directory in your application, since it will be generated for you when you create your first mailable class using the `make:mail` Pool command:

```bash
php pool make:mail InvoiceMail
```

### Configuring the Sender

You specify a global "from" address in your `runtime/config/mail.php` configuration file. This address will be used to send mail unless a Mailable sets its own.

```php
'from' => [
    'address' => env('MAIL_FROM_ADDRESS', 'hello@example.com'),
    'name' => env('MAIL_FROM_NAME', 'Example'),
],
```

### Configuring the Subject

Every mail has a subject. Doppar lets you define one in a very convenient way — just implement the `subject` method on your mailable class.

```php
/**
 * Define mail subject
 *
 * @return Phaseolies\Support\Mail\Mailable\Subject
 */
public function subject(): Subject
{
    return new Subject(
        subject: 'New Mail'
    );
}
```

### Configuring the View

Within a mailable class's `content` method, you may define the view — or which template should be used when rendering the email's contents. Since each email typically uses an Odo template to render its contents, you have the full power and convenience of the Odo templating engine when building your email's HTML:

```php
/**
 * Set the message body and data
 *
 * @return Phaseolies\Support\Mail\Mailable\Content
 */
public function content(): Content
{
    return new Content(
        view: 'Optional view.name'
    );
}
```

If you want to pass data without a view, you can pass a string or an array:

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

You can even send mail without passing any data to `Content`. If you just want to send an attachment, for example, leave `content` empty:

```php
public function content(): Content
{
    return new Content();
}
```

## Example of Sending Mail

Doppar provides `to` and `send` methods to send a basic mail. Use the `Phaseolies\Support\Facades\Mail` facade to handle mail functionality.

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
    }
}
```

Or send mail by passing only an email address:

```php
Mail::to('recipient@example.com')->send(new InvoiceMail($data));
```

You can also pass a name as the second argument:

```php
Mail::to('recipient@example.com', 'recipient_name')->send(new InvoiceMail($data));
```

`send()` and `deliver()` are interchangeable — `deliver()` is a plain alias, pick whichever reads better in your chain:

```php
Mail::to('recipient@example.com')->deliver(new InvoiceMail($data));
```

Both return a `Symfony\Component\Mailer\SentMessage`, which you can inspect after sending:

```php
$sent = Mail::to('recipient@example.com')->send(new InvoiceMail($data));

$sent->getMessageId();
```

Now build your `InvoiceMail` mailable class:

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
            view: 'emails.order.invoice',
            data: $this->data
        );
    }

    public function attachment(): array
    {
        return [];
    }
}
```

> Passing `data` will be available in `resources/views/emails/order/invoice.odo.php`, access it via `[[ $data ]]`

## Queueing Mail

`Mail::to(...)->send(...)` connects to your mailer and sends synchronously — the request waits for the SMTP round-trip (or the provider's API call) to finish before it can respond. For anything user-facing, that latency is usually not worth spending on the critical path. Wrap the send in a [queued job](/versions/4.x/doppar-queue) instead, and dispatch that:

```php
<?php

namespace App\Jobs;

use App\Mail\InvoiceMail;
use Doppar\Queue\Attributes\Queueable;
use Doppar\Queue\Dispatchable;
use Doppar\Queue\Job;
use Phaseolies\Support\Facades\Mail;

#[Queueable(tries: 3, retryAfter: 60)]
class SendInvoiceMailJob extends Job
{
    use Dispatchable;

    public function __construct(protected string $recipient, protected array $data) {}

    public function handle(): void
    {
        Mail::to($this->recipient)->send(new InvoiceMail($this->data));
    }

    public function failed(\Throwable $exception): void
    {
        // Logged, retried, or escalated, depending on your app.
    }
}
```

Now dispacth the queue job like the following way.
```php
SendInvoiceMailJob::dispatchWith($user->email, $data);
```

The controller returns immediately; the worker sends the mail — including retries via `#[Queueable(tries: ...)]` if the provider has a transient failure. Nothing about the Mailable itself changes; only where `Mail::to(...)->send(...)` gets called moves.

## Sending Mail with Attachment

To send mail with an attachment, pass the attachment path using the `attachment` method.

```php
public function attachment(): array
{
    return [
        storage_path('invoice.pdf') => [
            'as' => 'rename_invoice.pdf',
            'mime' => 'application/pdf',
        ]
    ];
}
```

The file will be sent using the name `rename_invoice.pdf` and mime type `application/pdf`.

### Multiple Attachments

You can also send mail with multiple attachments. Just pass an array of files in the `attachment` method:

```php
public function attachment(): array
{
    return [
        storage_path('invoice_1.pdf') => [
            'as' => 'invoice_1.pdf',
            'mime' => 'application/pdf',
        ],
        storage_path('invoice_2.pdf') => [
            'as' => 'invoice_2.pdf',
            'mime' => 'application/pdf',
        ],
        storage_path('invoice_3.pdf') => [
            'as' => 'invoice_3.pdf',
            'mime' => 'application/pdf',
        ],
    ];
}
```

Both `as` and `mime` are optional — you can simply send email attachments like this:

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

If you'd rather attach something decided at dispatch time instead of baking it into the Mailable, chain `attach()` on the instance before sending — it wins over `attachment()` if both are present:

```php
Mail::to($user)->send(
    (new InvoiceMail($data))->attach(storage_path('invoice.pdf'), 'invoice.pdf')
);
```

## With CC and BCC

You are not limited to just specifying the "to" recipients when sending a message. You are free to set "to", "cc", and "bcc" recipients by chaining their respective methods together:

```php
use Phaseolies\Support\Facades\Mail;

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));
```

`cc()`/`bcc()` accept a single address, a `['address' => ..., 'name' => ...]` array, or a list of either — repeated calls accumulate rather than overwrite. These are real SMTP envelope recipients, not just header decoration, so they genuinely receive the message.

## Inline (Embedded) Images

For an image referenced from inside the HTML body itself — a logo in the header, not a downloadable attachment — embed it and reference it by filename with `cid:`:

```php
Mail::to($user)->send(
    (new InvoiceMail($data))->embed(public_path('logo.png'), 'logo.png')
);
```

```html
<img src="cid:logo.png" alt="Company logo">
```

## Plain Text Alternative

Symfony Mailer automatically builds a `multipart/alternative` message when both an HTML and a text body are present:

```php
(new InvoiceMail($data))->text('Your order has shipped. View details at https://example.com/orders/2381');
```

If you don't set one, only the HTML part is sent — most mail clients render that fine, but a text part improves spam scoring and accessibility.

## Custom Headers

Sometimes the built-in fields aren't enough — you need to stamp a message with something your own application, a webhook, or a downstream system will read back later. The `header` method adds any arbitrary header to the outgoing email, and you can call it as many times as you need for different header names:

```php
(new InvoiceMail($data))->header('X-Order-Id', '123-123');
```

## Tags and Metadata

Useful for filtering and searching in your mail provider's dashboard (Mailtrap, Postmark, SES, and others all surface these):

```php
Mail::to($user)->send(
    (new InvoiceMail($data))
        ->tag('order-shipped')
        ->metadata('order_id', '123-123')
        ->metadata('customer_tier', 'gold')
);
```

Tags become an `X-Doppar-Tag` header per tag; metadata becomes `X-Doppar-Metadata-{key}` headers.

## Priority

Priority is a hint, not a guarantee — it's sent as the standard `X-Priority` email header, and it's entirely up to the recipient's mail client whether to act on it (Outlook, for instance, shows a red exclamation mark for high-priority mail; most webmail clients ignore it outright). Use it sparingly, for things like account security alerts or payment failures, rather than routine notifications.

```php
use Symfony\Component\Mime\Email;

(new InvoiceMail($data))->priority(Email::PRIORITY_HIGH);
```

Available constants: `PRIORITY_HIGHEST`, `PRIORITY_HIGH`, `PRIORITY_NORMAL` (default), `PRIORITY_LOW`, `PRIORITY_LOWEST`.

## Reply-To, Sender, Return-Path

These three addresses solve different problems, and it's easy to reach for the wrong one. `replyTo` is the most common — it's where a human reply lands when your "from" address is a no-reply mailbox that can't (or shouldn't) receive mail back:

```php
(new InvoiceMail($data))
    ->replyTo('support@example.com', 'Support Team')
    ->replyTo('sales@example.com'); // repeatable — adds another Reply-To
```

`sender` and `returnPath` are set directly on the Mailable when you need bounce handling to point somewhere other than the "from" address:

```php
class InvoiceMail extends Mailable
{
    public function __construct(protected $data)
    {
        $this->returnPath = 'bounces@example.com';
    }

    // ...
}
```

## Escaping the Fluent API: `build()`

Every Mailable can override `build(Email $email): Email` to reach anything Symfony's `Email` object exposes that the helpers above don't cover. It runs last, right before the message is handed to the transport:

```php
use Symfony\Component\Mime\Email;

public function build(Email $email): Email
{
    $email->getHeaders()->addTextHeader('X-Custom-Tracking', $this->trackingId);

    return $email;
}
```

## Swapping the Transport per Send

`runtime/config/mail.php`'s `default` mailer is a sensible fallback, but any single send can go through a different transport entirely — a raw DSN string or a real `TransportInterface`:

```php
Mail::to($user)
    ->driver('smtp://user:pass@backup-smtp.example.com:587')
    ->send(new InvoiceMail($data));
```

```php
use Symfony\Component\Mailer\Transport;

Mail::to($user)
    ->driver(Transport::fromDsn(env('URGENT_MAILER_DSN')))
    ->send(new InvoiceMail($data));
```

## Third-Party Providers: Mailgun, SES, Postmark, Brevo, SendGrid

The `smtp`/`sendmail`/`null` mailers built into `runtime/config/mail.php` are just the transports Symfony ships without any extra dependencies. Every major transactional email provider has its own Symfony Mailer bridge — install it, drop in a DSN, and nothing else in your Mailables or your `Mail::` calls changes.

Each bridge is its own package, plus `symfony/http-client` for providers that send over their HTTP API instead of raw SMTP:

```bash
composer require symfony/mailgun-mailer symfony/http-client
```

Then point any mailer's `dsn` at the provider — via `env.toml`, `MAILER_DSN`, or a named entry in `runtime/config/mail.php`:

| Provider | Package | API DSN | SMTP DSN |
|---|---|---|---|
| Mailgun | `symfony/mailgun-mailer` | `mailgun+api://KEY:DOMAIN@default?region=us` | `mailgun+smtp://USERNAME:PASSWORD@default` |
| Amazon SES | `symfony/amazon-mailer` | `ses+api://ACCESS_KEY:SECRET_KEY@default?region=eu-west-1` | `ses+smtp://USERNAME:PASSWORD@default` |
| Postmark | `symfony/postmark-mailer` | `postmark+api://KEY@default` | `postmark+smtp://USERNAME:PASSWORD@default` |
| Brevo | `symfony/brevo-mailer` | `brevo+api://KEY@default` | `brevo+smtp://USERNAME:PASSWORD@default` |
| SendGrid | `symfony/sendgrid-mailer` | `sendgrid+api://KEY@default` | `sendgrid+smtp://USERNAME:PASSWORD@default` |

```env
MAILER_DSN=mailgun+api://key-xxxxxxxx:example.com@default?region=us
```

The `+api` variants skip the SMTP handshake per message and are what most providers recommend; the `+smtp` variants are a drop-in fallback if you'd rather not add the HTTP client dependency, or your provider account only issues SMTP credentials.

Everything else — `Mail::to(...)->send(...)`, attachments, tags, signing — works identically regardless of which transport is behind the DSN. That's the whole point of `dsn` living in `runtime/config/mail.php`: swapping providers is a one-line change, not a code change.

## High Availability: Failover and Round-Robin

Symfony understands compound DSNs natively — no extra Doppar config for this, just write the DSN:

```php
'smtp' => [
    'dsn' => 'failover(smtp://primary-host sendmail://default)',
],
```

```php
'smtp' => [
    'dsn' => 'roundrobin(smtp://host-one smtp://host-two)',
],
```

**Failover** tries transports in order and only moves to the next one if the current one fails. **Round-robin** load-balances across all of them.

## Message Signing: DKIM and S/MIME

Both sign every outgoing email automatically, right before it's handed to the transport — no changes needed in any Mailable. Both require the `openssl` PHP extension.

```php
'signing' => [
    'dkim' => [
        'enabled' => env('MAIL_DKIM_ENABLED', false),
        'private_key' => env('MAIL_DKIM_PRIVATE_KEY'), // PEM string, or a "file://..." path
        'passphrase' => env('MAIL_DKIM_PASSPHRASE', ''),
        'domain' => env('MAIL_DKIM_DOMAIN'),
        'selector' => env('MAIL_DKIM_SELECTOR'),
    ],

    'smime' => [
        'enabled' => env('MAIL_SMIME_ENABLED', false),
        'certificate' => env('MAIL_SMIME_CERTIFICATE'), // path to a PEM certificate
        'private_key' => env('MAIL_SMIME_PRIVATE_KEY'), // path to a PEM private key
        'passphrase' => env('MAIL_SMIME_PASSPHRASE'),
    ],
],
```

```env
MAIL_DKIM_ENABLED=true
MAIL_DKIM_PRIVATE_KEY="file:///etc/doppar/dkim/private.pem"
MAIL_DKIM_DOMAIN=example.com
MAIL_DKIM_SELECTOR=mail
```

Leave both disabled (the default) to skip signing entirely.