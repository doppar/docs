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

Doppar notification system offers robust features including multi-channel delivery with support for `database`, `Slack`, `Discord`, and custom channel, secure model serialization that safely handles Eloquent models in notification payloads, and automatic retry logic with configurable attempts.

It supports bulk notifications for sending to thousands of users efficiently, delayed and scheduled delivery for future notifications, `read/unread` tracking for in-app notifications, and a fluent API with intuitive, chainable syntax that makes sending notifications a breeze.

## Features
The Doppar notification system is designed to deliver messages reliably across multiple platforms with flexibility and scalability in mind. Its feature set ensures smooth notification handling, better user engagement, and full control over how messages are sent. See the features of doppar notifier.

- Multi-Channel Delivery - Send via database, Slack, Discord, custom channel
- Background Processing - Queue-based delivery with automatic retries
- Secure Serialization - Safe handling of Eloquent models in payloads
- Bulk Operations - Efficiently notify thousands of users
- Scheduled Delivery - Send notifications at specific times
- Read Tracking - Mark notifications as read/unread
- Fluent API - Intuitive, chainable syntax
- Custom Channels - Easily create custom delivery channels
- Conditional Sending - Control when notifications should be sent

## Notification Channels
Doppar supports multiple notification channels out of the box:
- database - Store in database
- slack - Post to Slack channels
- discord - Post to Discord channels
- Custom channel - User extended channel

## Installation
You may install Doppar Notifier via the composer require command:
```bash
composer require doppar/notifier
```
> 🚩 The Doppar Notifier depends on the Doppar Queue. Please ensure that the Doppar Queue is properly set up in your application

### Register Provider
Next, register the Notifier service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `NotifierServiceProvider` to the providers array:
```php
'providers' => [
    // Other service providers...
    \Doppar\Queue\QueueServiceProvider::class,
    \Doppar\Notifier\NotifierServiceProvider::class,
],
```
This step ensures that Doppar knows about `Notifier` and can load its functionality when the application boots.

### Publish Configuration
Now we need to publish the configuration files by running this pool command.
```bash
php pool vendor:publish --provider="Doppar\Notifier\NotifierServiceProvider"
```

Now run migrate command to migrate notification related tables
```bash
php pool migrate
```

This creates the `notifications` table.

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

use Phaseolies\Database\Entity\Model;
use Doppar\Notification\Notifiable;

class User extends Model
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
    return ['database', 'slack', 'discord'];
}
```

## Channel-Specific Content
The `content()` method defines the notification message for multiple channels at once, formatting the information—like payment amount and invoice link—appropriately for each platform so that database records, Slack messages, and Discord notifications all receive consistent and relevant content.
```php
public function content($notifiable): array
{
    return [
        // Database content
        'title' => 'Payment Received',
        'message' => 'We received your payment of $' . $this->amount,
        'action_url' => url('/invoices/' . $this->invoice->id),
        'action_text' => 'View Invoice',

        // Slack content
        'text' => 'Payment received: $' . $this->amount,
        'username' => 'Payment Bot',
        'icon' => ':moneybag:',

        // Discord content
        'content' => 'Payment of $' . $this->amount . ' received!',
    ];
}
```

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
- Slack: slack_webhook_url
- Discord: discord_webhook_url

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
        $content = $notification->content($notifiable);

        $this->sendToTelegram($chatId, $content['message']);
    }

    protected function sendToTelegram(string $chatId, string $message): void
    {
       //
    }
}
```
Now register your custom channel within a service provider so it becomes available to the notification system:
```php
use Doppar\Notifier\NotificationManager;
use App\Notifications\Channels\TelegramChannel;

public function boot()
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
### Retrieve User Notifications
```php
// Get unread notifications
Auth::user()->unreadNotifications();

// Get read notifications
Auth::user()->readNotifications();

// Get all notifications
Auth::user()->notifications();

// Mark all notifications as read
Auth::user()->markNotificationsAsRead();
```

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
```

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

The Doppar Notification System provides a robust solution for multi-channel notifications. With its intuitive API and powerful features, you can send notifications to users via slack, discord, database, and more, all while maintaining a smooth user experience through background processing.