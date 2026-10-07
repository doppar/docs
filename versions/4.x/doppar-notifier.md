---
title: Doppar Notifier
description: A Notificication component of doppar framework
meta:
  - name: keywords
    content: Notification, queueable notification, php notification
---

## Notifier

### Introduction
A notification system allows you to send messages to users across multiple channels like email, SMS, Slack, Discord, and in-app notifications. Instead of sending notifications immediately and blocking your application, Doppar's notification system runs in the background using the powerful queue system, ensuring smooth user experiences and reliable delivery even under heavy load.

Doppar notification system offers robust features including multi-channel delivery with support for `database`, `mail`, `Slack`, `Discord`, `webhook`, and custom channel, secure model serialization that safely handles Eloquent models in notification payloads, and automatic retry logic with configurable attempts. Every channel is delivered by its own queued job, so a channel that fails is retried on its own and never sends the others again.

It supports bulk notifications for sending to thousands of users efficiently, delayed and scheduled delivery for future notifications, `read/unread` tracking for in-app notifications, a delivery log that shows what happened on every channel, events, a test fake, and a fluent API with intuitive, chainable syntax that makes sending notifications a breeze.

## Features
The Doppar notification system is designed to deliver messages reliably across multiple platforms with flexibility and scalability in mind. Its feature set ensures smooth notification handling, better user engagement, and full control over how messages are sent. See the features of doppar notifier.

- Multi-Channel Delivery - Send via database, mail, Slack, Discord, webhook, custom channel
- Background Processing - Queue-based delivery with automatic retries, one job per channel
- Delivery Log - See what happened to every notification on every channel, with attempts and the last error
- Events - Listen for sending, sent and failed deliveries, and cancel a delivery
- Testing - `Notification::fake()` with assertions for the tests of your application
- Secure Serialization - Safe handling of Eloquent models in payloads
- Bulk Operations - Efficiently notify thousands of users
- Scheduled Delivery - Send notifications at specific times
- Read Tracking - Mark notifications as read/unread
- Fluent API - Intuitive, chainable syntax
- Custom Channels - Easily create custom delivery channels
- Conditional Sending - Control when notifications should be sent, per notification, per channel and per person
- Throttling - Limit how often the same notification reaches the same person
- Queue Options - Priority, unique notifications, retry backoff and a queue connection per notification

## Notification Channels
Doppar supports multiple notification channels out of the box:
- database - Store in database
- mail - Send an email
- slack - Post to Slack channels
- discord - Post to Discord channels
- webhook - Post JSON to any URL, optionally signed
- Custom channel - User extended channel

## Installation
You may install Doppar Notifier via the composer require command:
```bash
composer require doppar/notifier
```
> 🚩 The Doppar Notifier depends on the Doppar Queue. Please ensure that the Doppar Queue is properly set up in your application

### Register Launcher
Next, register the Notifier launcher so that Doppar can initialize it properly. Open your `runtime/config/app.php` file and add the `NotifierLauncher` to the `launchers` array:
```php
'launchers' => [
    // Other launchers
    \Doppar\Queue\QueueLauncher::class,
    \Doppar\Notifier\NotifierLauncher::class,
],
```
This step ensures that Doppar knows about `Notifier` and can load its functionality when the application launchs.

### Publish Configuration
Now we need to publish the configuration files by running this pool command.
```bash
php pool vendor:publish --launcher="Doppar\Notifier\NotifierLauncher"
```

Now run migrate command to migrate notification related tables
```bash
php pool migrate
```

This creates the `notifications` table, and the `notification_deliveries` table that holds the [`delivery log`](#delivery-log-and-reliability).

> 💡 The `notification_deliveries` table is recommended but not required. Without it notifications are still delivered; only the delivery log, skipping a channel that was already delivered when a job is retried, and throttling are off.

## Quick Start
Doppar makes it easy to create a new notification using the pool command. For example, to generate a notification for order shipment, run:
```bash
php pool make:notification OrderShippedNotification
```

This will create a ready-to-use notification class in `app/Notifications/OrderShippedNotification.php` that you can customize.

### Create Your Notification
Once generated, update your notification class like this:
```php
<?php

namespace App\Notifications;

use Doppar\Notifier\Contracts\Notification;
use App\Models\Order;

class OrderShippedNotification extends Notification
{
    public function __construct(public Order $order) {}

    /**
     * Define which channels to use for delivery
     *
     * @param mixed $notifiable
     * @return array
     */
    public function channels($notifiable): array
    {
        return ['database'];
    }

    /**
     * Define the notification content for each channel
     *
     * @param mixed $notifiable
     * @return array
     */
    public function content($notifiable): array
    {
        return [
            // Database content
            'title' => 'Your Order Has Shipped!',
            'message' => "Order #{$this->order->id} is on its way",
            'action_url' => url("/orders/{$this->order->id}/track"),
            'action_text' => 'Track Shipment',
        ];
    }
}
```

## Make Your Model Notifiable
Add the Notifiable trait to your User model (or any model that should receive notifications):
```php
<?php

namespace App\Models;

use Phaseolies\Auth\Authable;
use Doppar\Notifier\Concerns\Notifiable;

class User extends Authable
{
    use Notifiable;
}
```

### Send the Notification
Send your notification from any controller or service:
```php
namespace App\Http\Controllers;

use App\Notifications\OrderShippedNotification;
use App\Http\Controllers\Controller;

class NotificationController extends Controller
{
    #[Route(uri: 'notify')]
    public function shipOrder(Request $request, $orderId)
    {
        $order = Order::find($orderId);
        $order->status = 'shipped';
        $order->save();

        $order->customer->notify(new OrderShippedNotification($order));

        return response()->json(['message' => 'Order shipped successfully']);
    }
}
```

> 💡 `send()` returns the notification id, a UUID shared by the queued jobs of that send. Use it to find the notification in the [`delivery log`](#delivery-log-and-reliability). It returns `null` when nothing was sent, for example when the notification has no channels or was sent with `notifyNow()`.

### Using Global Helper
Use the `notify()` helper function for a more functional approach:
```php
#[Route(uri: 'notify')]
public function shipOrder(Request $request, $orderId)
{
    $order = Order::find($orderId);
    $order->status = 'shipped';
    $order->save();

    notify($order->customer)->send(new OrderShippedNotification($order));

    return response()->json(['message' => 'Order shipped successfully']);
}
```

## Run the Worker
To start processing queued notifications, run the worker using the following command:
```bash
php pool queue:run --queue=notifications
```

> 💡 Your notifications will now be processed in the background, ensuring your application remains fast and responsive. The queue worker will automatically retry failed notifications based on your configuration.

## Define Notification Channels
In your notification class, define which channels to use:
```php
public function channels($notifiable): array
{
    return ['database', 'mail', 'slack', 'discord'];
}
```

Each channel becomes its own queued job. If Slack is down, only the Slack job is retried: the database row and the email that already went out are not repeated.

## Channel-Specific Content
Every channel can have its own content. Add a method named after the channel and Doppar uses it for that channel only: `toMail()`, `toSlack()`, `toDiscord()`, `toWebhook()`, `toDatabase()`, or `toTelegram()` for a custom `telegram` channel.
```php
class PaymentReceivedNotification extends Notification
{
    public function __construct(public Invoice $invoice, public float $amount) {}

    public function channels($notifiable): array
    {
        return ['database', 'mail', 'slack'];
    }

    // Used by the database channel and
    // any channel without a method of its own
    public function content($notifiable): array
    {
        return [
            'title' => 'Payment Received',
            'message' => 'We received your payment of $' . $this->amount,
            'action_url' => url('/invoices/' . $this->invoice->id),
        ];
    }

    // Used by the mail channel only
    public function toMail($notifiable): array
    {
        return [
            'subject' => 'Payment received',
            'greeting' => "Hi {$notifiable->name}",
            'lines' => ['We received your payment of $' . $this->amount . '.'],
            'action' => ['text' => 'View invoice', 'url' => url('/invoices/' . $this->invoice->id)],
        ];
    }

    // Used by the Slack channel only
    public function toSlack($notifiable): array
    {
        return [
            'text' => 'Payment received: $' . $this->amount,
            'username' => 'Payment Bot',
            'icon' => ':moneybag:',
        ];
    }
}
```

When a notification has no method for a channel, `content()` is used for it. A single `content()` array can therefore still hold the keys of several channels, as in earlier versions:
```php
public function content($notifiable): array
{
    return [
        // Database content
        'title' => 'Payment Received',
        'message' => 'We received your payment of $' . $this->amount,

        // Slack content
        'text' => 'Payment received: $' . $this->amount,
        'username' => 'Payment Bot',
        'icon' => ':moneybag:',

        // Discord content
        'content' => 'Payment of $' . $this->amount . ' received!',
    ];
}
```

Channel names with a dash or underscore map to StudlyCase: a `push-alert` channel uses `toPushAlert()`.

## Mail Channel
The `mail` channel sends an email through Doppar's mailer, so it uses your `mail` configuration. The email is addressed to `routeNotificationForMail()` on the notifiable, or its `email` attribute when the method is not defined.

Return the parts of the message from `toMail()` and Doppar lays them out for you:
```php
public function toMail($notifiable): array
{
    return [
        'subject' => 'Welcome to Doppar',
        'greeting' => "Hi {$notifiable->name}",
        'lines' => [
            'Your account is ready.',
            'Here is where to start.',
        ],
        'action' => ['text' => 'Get started', 'url' => 'https://app.example.com/start'],
        'outro' => ['If you did not sign up, ignore this email.'],
        'footer' => 'Doppar Inc.',
    ];
}
```

Or give your own body with `html` and `text`:
```php
public function toMail($notifiable): array
{
    return [
        'subject' => 'Invoice',
        'html' => "<h1>Invoice #{$this->invoice->id}</h1><p>Your invoice is ready.</p>",
        'text' => "Invoice #{$this->invoice->id}: your invoice is ready.",
    ];
}
```

| Key | Meaning |
|---|---|
| `subject` | The subject line (default `Notification`) |
| `greeting`, `lines`, `action`, `outro`, `footer` | The parts laid out for you. `action` is `['text' => ..., 'url' => ...]` |
| `html`, `text` | Your own body. When `html` is given the parts above are not used |
| `cc`, `bcc`, `replyTo` | Extra recipients and reply address |
| `attachments` | A list of file paths, or `['path' => ..., 'name' => ..., 'mime' => ...]` |
| `tags`, `priority` | Mail tags and priority |

> 🚩 Every value in the laid-out message is escaped, and an `action` link must be a plain `http(s)` address. A message with an invalid action link, or a notifiable without a valid email address, fails the delivery instead of sending a broken email.

## Webhook Channel
The `webhook` channel posts JSON to any URL. The URL comes from `routeNotificationForWebhook()` on the notifiable, or its `webhook_url` attribute, and must be an `http(s)` address.
```php
public function toWebhook($notifiable): array
{
    return [
        'event' => 'order.shipped',
        'order_id' => $this->order->id,
    ];
}
```

The returned array is the JSON body. Use the `payload`, `headers` and `secret` keys for more control:
```php
public function toWebhook($notifiable): array
{
    return [
        'payload' => ['event' => 'order.shipped', 'order_id' => $this->order->id],
        'headers' => ['X-Source' => 'shop'],
        'secret' => config('services.hooks.secret'),
    ];
}
```

When a `secret` is given (or `notification.webhook.secret` is set in the configuration) the body is signed. The receiver gets the header `X-Notification-Signature: sha256=<hmac of the body>` and can check who sent it:
```php
$expected = 'sha256=' . hash_hmac('sha256', $rawBody, $secret);
$valid = hash_equals($expected, $request->header('X-Notification-Signature'));
```

A response that is not 2xx fails the delivery, so the queue retries it. The `Content-Type` header cannot be overridden, and a custom header containing a line break is dropped.

## Slack and Discord
Slack and Discord post to the webhook URL of the notifiable. A failed request (a non-2xx answer, and for Slack an answer other than `ok`) throws, so the delivery is retried and, after the last attempt, recorded as failed.

## Channel Routing
The User model defines how notifications should be delivered to each channel by providing routing information. The `routeNotificationForDiscord()` method returns the user’s `discord_webhook_url` for Discord notifications, while `routeNotificationForSlack()` returns the user’s Slack webhook URL, ensuring that messages are sent to the correct destination for each platform.
```php
class User extends Model
{
    use Notifiable;

    public function routeNotificationForDiscord()
    {
        return $this->discord_webhook_url;
    }

    public function routeNotificationForSlack()
    {
        return $this->slack_webhook_url;
    }
}
```

If no routing method is defined, the system looks for these properties:
- Mail: email
- Slack: slack_webhook_url
- Discord: discord_webhook_url
- Webhook: webhook_url

### Send Using Facade
You can send notification by using the `Notification` facade for explicit notification handling:
```php
use Doppar\Notifier\Supports\Facades\Notification;

Notification::to($subscription->user)
    ->send(new SubscriptionRenewedNotification($subscription));
```

## Immediate vs Queued
By default, all doppar notifications are queued for background processing. But you can send them immediately if needed:
```php
// Queued (default) - runs in background
$user->notify(new WelcomeNotification());

// Immediate - runs right now (blocks the request)
$user->notifyNow(new UrgentAlertNotification());

// Immediate - runs right now (blocks the request)
$user->notify(new WelcomeNotification())->immediate();
```

## Specifying Channels
Override the default channels defined in your notification class:
```php
// Send only via discord and slack
// This will ignore notification's channels() method
$user->notify(new OrderShippedNotification($order))
     ->via(['discord', 'slack'])
     ->send();

// Send to database only
$user->notify(new SystemUpdateNotification())
     ->via(['database'])
     ->send();
```

## Delayed Notifications
Schedule notifications to be sent after a specific delay:
```php
// Send after 1 hour (3600 seconds)
$user->notify(new FollowUpNotification())
     ->delay(3600)
     ->send();

// Send after 24 hours
notify($user)
    ->after(86400)
    ->send(new ReminderNotification());
```

## Combining Options
Chain multiple options together for complex scenarios:
```php
// Send via discord and slack, after 30 minutes
$user->notify(new PaymentReminderNotification($invoice))
     ->via(['discord', 'slack'])
     ->delay(1800)
     ->send();

// Immediate delivery, specific channels
$user->notify(new SecurityAlertNotification())
     ->via(['discord', 'slack'])
     ->immediate();
```

## Bulk Notifications
You can efficiently send notifications to multiple users at once, ensuring scalability and optimized delivery. The Notification facade provides a fluent interface for bulk operations.
```php
use Doppar\Notifier\Supports\Facades\Notification;

Notification::toMany($users)
    ->via(['database', 'slack'])
    ->batchSize(100)
    ->send(new SystemAnnouncementNotification($request->message));
```

## Query-Based Notifications
You can target notifications to users dynamically by querying based on specific criteria. This approach allows you to send messages only to relevant recipients, improving efficiency and relevance.
```php
Notification::toAll(User::class)
    ->where('subscription', 'premium')
    ->where('active', true)
    ->via(['slack', 'database'])
    ->chunkSize(50)
    ->send(new PremiumCampaignNotification());
```

## Scheduled Notifications
Notifications can be scheduled to be sent at a specific time, allowing you to automate reminders, alerts, or timed campaigns.
```php
$appointment = Appointment::find($appointmentId);

// Schedule for 24 hours before appointment
$reminderTime = strtotime($appointment->scheduled_at) - 86400;

Notification::schedule(new AppointmentReminderNotification($appointment))
    ->to($appointment->user)
    ->at($reminderTime);
```

## Conditional Notifications
You can control whether a notification should be sent by implementing the `shouldSend()` method on the notification class. This allows for fine-grained logic based on user preferences, time of day, or any custom conditions.
```php
class OrderNotification extends Notification
{
    /**
     * Check if notification should be sent
     */
    public function shouldSend($notifiable): bool
    {
        // Only send if user wants order notifications
        if (!$notifiable->preferences->order_notifications) {
            return false;
        }

        // Don't send notifications at night
        $hour = (int) date('H');
        if ($hour >= 22 || $hour <= 7) {
            return false;
        }

        return true;
    }
}
```
The `shouldSend()` method is executed before dispatching the notification. If it returns false, the notification is skipped—regardless of channel, queue, or schedule.

### Skipping One Channel
Implement `shouldSendVia()` to skip a single channel while the others go out:
```php
public function shouldSendVia($notifiable, string $channel): bool
{
    // Only email people who confirmed their address
    return $channel !== 'mail' || $notifiable->email_verified;
}
```

A skipped channel is recorded as `skipped` in the [`delivery log`](#delivery-log-and-reliability).

## Channel Preferences
Let each person choose how they are notified by implementing `wantsNotification()` on the notifiable. It is asked for every channel a notification chooses for itself, and a channel that returns false is not used:
```php
use Doppar\Notifier\Contracts\Notification;

class User extends Authable
{
    use Notifiable;

    public function wantsNotification(Notification $notification, string $channel): bool
    {
        return !in_array($channel, $this->muted_channels ?? [], true);
    }
}
```

> 💡 Preferences filter the channels returned by the notification's `channels()` method. Channels you pass explicitly with `->via([...])` are a deliberate choice and are not filtered. If every channel is filtered out, nothing is queued.

## Throttling
Stop a notification from reaching the same person too often by implementing `throttle()` on the notification:
```php
public function throttle(): ?array
{
    // At most 3 of these per person and per channel in an hour
    return ['max' => 3, 'per' => 3600];
}
```

Deliveries over the limit are skipped and recorded as `throttled`. The limit is counted from the [`delivery log`](#delivery-log-and-reliability), so it needs the `notification_deliveries` table. Under heavy concurrency the limit is best effort: several workers delivering at the same moment can pass it by a little.

## Notification Metadata
You can attach metadata to a notification to support filtering, categorization, analytics, or custom processing within your system. The `metadata()` method returns an array of key–value pairs that travel with the notification across all channels.
```php
class OrderNotification extends Notification
{
    public function metadata(): array
    {
        return [
            'category' => 'orders',
            'priority' => 'high',
            'order_id' => $this->order->id,
            'tracking_number' => $this->order->tracking_number,
            'tags' => ['shipping', 'ecommerce'],
        ];
    }
}
```

## Queue Options
A notification can tune how it is queued by implementing these optional methods. They use features of the Doppar Queue (priority, unique jobs, backoff and connections), so they take effect with a Doppar Queue release that has them; with an older release they are accepted and ignored.
```php
class PasswordResetNotification extends Notification
{
    // From -100 to 100
    // Higher priority notifications are delivered first
    public function priority(): int
    {
        return 80;
    }

    // The same notification is not queued twice 
    // for the same person and channel while one is waiting
    public function uniqueId($notifiable): ?string
    {
        return 'password-reset';
    }

    // Seconds to wait before each retry
    public function backoff(): int|array|null
    {
        return [10, 60, 300];
    }

    // The queue connection to use, null for the default
    public function onConnection(): ?string
    {
        return 'redis';
    }
}
```

## Custom Notification Channels
You can extend the notification system by creating your own custom channels to support third-party services or internal delivery mechanisms. A custom channel defines how a notification is sent and how the system should interact with your chosen platform.

### Creating a Custom Channel
```php
namespace App\Notifications\Channels;

use Doppar\Notifier\Channels\Contracts\ChannelDriver;
use Doppar\Notifier\Contracts\Notification;

class TelegramChannel extends ChannelDriver
{
    public function send($notifiable, Notification $notification): void
    {
        $chatId  = $notifiable->routeNotificationFor('telegram');
        $content = $notification->contentFor('telegram', $notifiable);

        $this->sendToTelegram($chatId, $content['message']);
    }

    protected function sendToTelegram(string $chatId, string $message): void
    {
       //
    }
}
```

> 🚩 Throw an exception when delivery fails. A channel that returns normally is counted as delivered; one that throws is retried and recorded as failed.

`contentFor('telegram', $notifiable)` returns what `toTelegram()` returns, or `content()` when the notification has no such method. Add the route for the channel on the notifiable with `routeNotificationForTelegram()`.
Now register your custom channel within a launcher so it becomes available to the notification system:
```php
use Doppar\Notifier\NotificationManager;
use App\Notifications\Channels\TelegramChannel;

public function launch()
{
    $manager = app(NotificationManager::class);
    $manager->extend('telegram', TelegramChannel::class);
}
```

Now you can send notification via telegram channel like
```php
public function channels($notifiable): array
{
    return ['telegram'];
}
```

## Reading Notifications
You can easily retrieve, inspect, and update the read status of user notifications. Doppar notifier provides convenient methods for accessing unread, read, or all notifications, as well as marking them accordingly.

> 🚩 These methods always belong to the entity you call them on, whoever is signed in. `$admin->notifications()` returns the admin's own, and `$user->notifications()` returns that user's, in a controller, a queue worker or a console command alike.

### Retrieve User Notifications
```php
// Get unread notifications
$user->unreadNotifications();

// Count unread notifications
$user->unreadNotificationsCount();

// Get read notifications
$user->readNotifications();

// Get all notifications, newest first
$user->notifications();

// Mark all notifications as read (returns how many were unread)
$user->markNotificationsAsRead();

// Mark one notification as read
$user->markNotificationAsRead($id);

// Delete one notification, or all the read ones
$user->deleteNotification($id);
$user->deleteReadNotifications();
```

`markNotificationAsRead($id)` and `deleteNotification($id)` only work on the entity's own notifications. An id that belongs to somebody else returns `false` and changes nothing.

### Working With a Specific Notification
Want to work with specific notification and want to mark as read or unread, follow this way:
```php
$notification = DatabaseNotification::find(81);

// Update read status
$notification->markAsRead();
$notification->markAsUnread();

// Check state
if ($notification->isRead()) {
    // Notification has been read
}

if ($notification->isUnread()) {
    // Notification is unread
}

// The stored content as an array
$data = $notification->payload();
```

### Deleting Old Notifications
Remove old notifications so the table does not grow forever:
```bash
# Delete read notifications older than 90 days
php pool notification:prune --days=90

# Delete every notification older than 30 days, read or not
php pool notification:prune --days=30 --all
```

The same is available in code:
```php
use Doppar\Notifier\Models\DatabaseNotification;

DatabaseNotification::prune(90);          // read ones only
DatabaseNotification::prune(30, false);   // read and unread
```

## Delivery Log and Reliability
Doppar records what happened to every notification on every channel in the `notification_deliveries` table. Each row has the channel, a status, the number of attempts and the last error.

| Status | Meaning |
|---|---|
| `sent` | The channel delivered the notification |
| `failed` | The last attempt failed (see the `error` column). The queue retries it |
| `skipped` | `shouldSend()` or `shouldSendVia()` said no |
| `throttled` | The notification reached its `throttle()` limit |
| `cancelled` | A `sending` event listener cancelled it |

`send()` returns the notification id, which is the `notification_id` of these rows:
```php
$id = $user->notify(new OrderShippedNotification($order))->send();

// Everything that happened to this user's notifications, newest first
$deliveries = $user->notificationDeliveries();

foreach ($deliveries as $delivery) {
    echo "{$delivery->channel}: {$delivery->status} after {$delivery->attempts} attempt(s)";

    if ($delivery->hasFailed()) {
        echo " - {$delivery->error}";
    }
}
```

The log is also what makes retries safe: when a job runs again, a channel that already delivered the notification is skipped, so a retry never sends it twice. By default a channel is tried 3 times, 60 seconds apart, and after the last attempt it lands in the failed jobs table of the queue.

> 💡 The log is best effort. If the `notification_deliveries` table does not exist, notifications are still delivered and a warning is logged; you only lose the log, the skip-on-retry safety net and throttling.

## Events
Listen for deliveries to log them, count them, or stop them. A listener receives the notifiable, the notification and the channel name:
```php
use Doppar\Notifier\Supports\Facades\Notification;

// Before a delivery. Return false to cancel it
Notification::sending(function ($notifiable, $notification, string $channel) {
    return $channel !== 'mail' || $notifiable->email_verified;
});

// After a successful delivery
Notification::sent(function ($notifiable, $notification, string $channel) {
    Metrics::increment("notifications.{$channel}");
});

// After a failed attempt. The exception is the fourth argument
Notification::failed(function ($notifiable, $notification, string $channel, \Throwable $e) {
    Log::warning("Notification failed on {$channel}: " . $e->getMessage());
});
```

Register listeners in a launcher so they are active in the web process and in the queue worker. A cancelled delivery is recorded as `cancelled`. A listener that throws an exception is logged and ignored: it never stops or fails a delivery. Remove every listener with `Notification::flushListeners()`.

## Testing Notifications
Use `Notification::fake()` in the tests of your application to check what would have been sent, without sending anything:
```php
use Doppar\Notifier\Supports\Facades\Notification;

public function test_paying_an_invoice_notifies_the_customer(): void
{
    $fake = Notification::fake();

    $this->post('/invoices/42/pay');

    $fake->assertSentTo($user, InvoicePaidNotification::class);
    $fake->assertNotSentTo($otherUser, InvoicePaidNotification::class);

    // Check the notification and its channels with a callback
    $fake->assertSentTo($user, fn ($notification, $channels, $notifiable) =>
        $notification->invoiceId === 42 && in_array('mail', $channels)
    );

    $fake->assertSentVia($user, InvoicePaidNotification::class, 'mail');
    $fake->assertSentTimes(InvoicePaidNotification::class, 1);
    $fake->assertCount(1);
}
```

| Method | Checks |
|---|---|
| `assertSentTo($notifiable, $class, $times = null)` | A notification (a class name or a callback) was sent to the notifiable, optionally an exact number of times |
| `assertNotSentTo($notifiable, $class)` | It was not sent to the notifiable |
| `assertSentTimes($class, $times)` | It was sent that many times, to anybody |
| `assertSentVia($notifiable, $class, $channel)` | It went out through the channel |
| `assertNothingSent()` | Nothing was sent |
| `assertCount($count)` | The total number of notifications sent |
| `sent($notifiable = null, $class = null)` | The recorded notifications, to inspect yourself |

While a fake is active nothing is queued, stored or sent, and no events fire. The recorded channels follow the same rules as a real send: `shouldSend` is not run, but channel preferences are applied, and `->via()` and `->delay()` are recorded. Go back to sending for real with `Notification::unfake()`. Calling `fake()` again starts with an empty record.

## Production Setup
### Supervisor Configuration
To run Doppar notification workers in production reliably, use Supervisor to manage worker processes. Supervisor ensures that workers automatically restart if they fail and allows you to run multiple processes in parallel.
```bash
[program:notification-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /var/www/html/pool queue:run --queue=notifications --sleep=3 --memory=256
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
user=www-data
numprocs=3
redirect_stderr=true
stdout_logfile=/var/www/html/storage/logs/notification-worker.log
stopwaitsecs=3600
```

### Start Supervisor
Reload Supervisor to apply the new configuration and start the workers:
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start notification-worker:*
```

### Monitor Workers
Check the status of your notification workers:
```bash
sudo supervisorctl status notification-worker:*
```

The Doppar Notification System provides a robust solution for multi-channel notifications. With its intuitive API and powerful features, you can send notifications to users via mail, slack, discord, webhook, database, and more, all while maintaining a smooth user experience through background processing.