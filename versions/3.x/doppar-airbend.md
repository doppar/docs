---
title: Airbend
description: Doppar Airbend Broadcasting System
meta:
  - name: keywords
    content: websocket, broadcasting, workerman, php
---

## Airbend

### Introduction
Doppar Airbend is a high-performance, real-time broadcasting component of the Doppar PHP Framework, engineered to deliver seamless WebSocket-based event broadcasting, robust channel authorization, and comprehensive performance monitoring. Designed for modern, scalable web applications, Airbend allows developers to build interactive and collaborative features—such as live notifications, chat systems, dashboards, and multiplayer experiences—directly within the Doppar ecosystem.

Airbend is highly configurable, offering schema-driven validation and caching mechanisms for robust configuration management. It also includes a comprehensive metrics and monitoring suite, allowing teams to track connection states, message flows, memory usage, and performance statistics in real time. With built-in CLI commands, PHP attributes for declarative event broadcasting, and a developer-friendly facade interface, Doppar Airbend empowers developers to implement real-time features with minimal overhead while maintaining full control over the broadcasting pipeline.

### System Requirements
Before installing and running Doppar Airbend, ensure your system meets the following prerequisites:

Required PHP extensions:
- `sockets` – for handling WebSocket connections
- `pcntl` – for process control in CLI mode
- `posix` – for process management and signal handling

### Supported Drivers
Doppar Airbend supports multiple broadcast drivers that determine how events are delivered across your application. Each driver handles message distribution differently, allowing you to choose the one that fits your environment.

- `Workerman:` Uses a built-in Workerman WebSocket server for handling real-time communication
- `Redis:` The redis driver uses redis connection to deliver events to the Airbend WebSocket server.
- `Null:` The null driver disables broadcasting. Events will be dispatched inside your application but will not be transmitted to the WebSocket server or any subscribers.


### Installation
You may install Doppar Airbend via the composer require command:
```bash
composer require doppar/airbend
```

### Register Provider
Next, register the Airbend service provider so that Doppar can initialize it properly. Open your `config/app.php` file and add the `AirbendServiceProvider` to the providers array:
```php
'providers' => [
    \Doppar\Airbend\AirbendServiceProvider::class,
],
```

### Publish Configuration
Now we need to publish the configuration files by running this pool command.
```bash
php pool vendor:publish --provider="Doppar\Airbend\AirbendServiceProvider"
```

This will publish `config/airbend.php` and `routes/channels.php` file and `Doppar Airbender` clinet. Update it as per your broadcasting events.

## Update Environment
Update your `.env` file to configure Doppar Airbend WebSocket broadcasting:

### Redis Driver
The Redis driver is recommended when you already have Redis in your infrastructure or are running a horizontally scaled application.
```bash
BROADCAST_DRIVER=redis
WEBSOCKET_APP_SECRET=random_secured_string
```

`BROADCAST_DRIVER` Sets the broadcasting driver to Redis and `WEBSOCKET_APP_SECRET` Shared secret used to authenticate broadcast requests between the app and WebSocket layer.

### Workerman Driver
The Workerman driver runs a dedicated WebSocket server using the Workerman PHP library. This is useful for lightweight or self-contained deployments.

Add the following variables to your `.env` file:
```bash
BROADCAST_DRIVER=workerman
WEBSOCKET_INTERNAL_PORT=6002
WEBSOCKET_APP_SECRET=custom_app_secret
```

Here `WEBSOCKET_INTERNAL_PORT` Internal port used by the Workerman WebSocket server.

> Switching drivers only requires updating the `.env` file and restarting your application

## Starting the Server
After configuring your environment, you can start the WebSocket server using the following command:
```bash
php pool websocket:start
```

You may combine SSL with custom host/port:
```bash
php pool websocket:start --host=0.0.0.0 --port=8443 --ssl
```

## Interact with Broadcasting
Doppar includes a `make:event` command that allows developers to quickly generate new broadcast event classes.
```bash
php pool make:event OrderShippedEvent
```

Here’s an example of how you could broadcast the `OrderShippedEvent` event from a controller using both the `Broadcast` facade and the `broadcast()` helper function:
```php
<?php

namespace App\Http\Controllers;

use App\Events\OrderShippedEvent;
use App\Models\Order;
use Doppar\Airbend\Support\Facades\Broadcast;
use Phaseolies\Http\Request;

class OrderController extends Controller
{
    public function shipOrder(Request $request, int $orderId)
    {
        $order = Order::find($orderId);

        // Your order shipping logic here

        // Broadcast using the Broadcast facade
        Broadcast::event(new OrderShippedEvent($order));

        // OR broadcast using the helper function
        broadcast(new OrderShippedEvent($order));
    }
}
```

The `OrderShippedEvent` event broadcasts order shipment information, including order ID, customer name, tracking number, and shipped time, on the orders channel using Doppar Airbend’s real-time broadcasting system.
```php
<?php

namespace App\Events;

use Doppar\Airbend\Broadcasting\Events\BaseBroadcastEvent;

class OrderShippedEvent extends BaseBroadcastEvent
{
    public function __construct(
        public $order
    ) {}

    /**
     * Get the channels the event should broadcast on
     */
    public function broadcastOn(): array
    {
        return ['orders'];
    }

    /**
     * Get the data to broadcast
     */
    public function broadcastWith(): array
    {
        return [
            'order_id' => $this->order->id,
            'customer' => $this->order->customer->name,
            'tracking_number' => $this->order->tracking_number,
            'shipped_at' => now()->toIso8601String(),
        ];
    }
}
```

## Doppar Airbender
Doppar Airbender is a real-time WebSocket client designed to simplify subscribing to channels, listening for server-side broadcast events, and sending ephemeral client-side updates. It provides a developer-friendly API for integrating live, interactive features into your web applications, such as chat systems, notifications, collaborative dashboards, and multiplayer experiences.

With Airbender, you can easily:
- Connect to public, private, or presence channels.
- Listen for server-broadcasted events and handle them in real time.
- Track online users and detect when members join or leave a presence channel.
- Send client-to-client messages (whispers) for ephemeral updates like typing indicators.
- Handle connection events, automatic reconnections, and errors with minimal setup.

Airbender abstracts the complexities of managing WebSocket connections, channel subscriptions, and event handling, allowing you to focus on building interactive, responsive applications.

## Public Channels
Once your first broadcasting event is fired, you can catch it on the client side using the Doppar `Airbender`. The following example demonstrates how to connect to the WebSocket server, subscribe to a channel, and listen for the `OrderShippedEvent` event in real time:
```html
<!DOCTYPE html>
<html>
<head>
    <title>Airbender Client</title>
</head>
<body>
    <!-- Include the Airbender client library -->
    <script src="[[ enqueue('vendor/airbend/assets/js/airbender.js') ]]"></script>
    <script>
        // Initialize Airbender with the WebSocket server host
        const airbender = new Airbender({
            host: 'ws://127.0.0.1:6001'
        });

        // Subscribe to the 'orders' channel
        const channel = airbender.channel('orders');

        // Listen for the 'OrderShippedEvent' event
        channel.listen('OrderShippedEvent', (data) => {
            console.log('Received order shipped data:', data);
        });
    </script>
</body>
</html>
```
This setup ensures that every time the `OrderShippedEvent` event is broadcasted from your server, the client will receive the event payload instantly and log the order details to the console.

You can also configure additional connection options like automatic reconnection and encryption:
```js
const airbender = new Airbender({
    host: "ws://127.0.0.1:6001", // WebSocket server host
    encrypted: false,            // Use secure WSS connection
    reconnect: true,             // Enable automatic reconnection
    reconnectAttempts: 5,        // Maximum reconnect attempts
    reconnectInterval: 3000      // Interval between reconnection attempts in ms
});
```

### Handling Connection Events
You can listen for connection status and errors using the on method on your Doppar client instance:

```javascript
// Connection events
airbender.on('connected', () => {
    //
});

airbender.on('disconnected', () => {
    //
});

airbender.on('error', (error) => {
    console.error('Error:', error);
});

airbender.on('ready', (socketId) => {
    console.log('Socket ID:', socketId);
});
```

### Listening for Custom Client Events
You can also define and trigger your own global events like this way:
```javascript
// Add a custom event listener
airbender.on('new-notification', (data) => {
    console.log('Received a notification:', data.title, '-', data.message);
});

// Trigger the custom event manually
airbender.trigger('new-notification', {
    title: 'Order Shipped',
    message: 'Order #1234 has been shipped!'
});
```

### Custom Event Names
Airbend allows you to customize the event name that will be broadcast to the client. You can define a custom event name by implementing the `broadcastAs()` method in your event class:
```php
/**
 * Get the event name for broadcasting
 */
public function broadcastAs(): string
{
    return 'order.shipped';
}
```

On the client side, you can listen for this event using the same defined name:
```js
channel.listen('order.shipped', (data) => {
    console.log('Received order shipped data:', data);
});
```

This allows you to fully control the event naming structure, making it easier to organize and standardize your application’s real-time events.

### Using Attributes for Event Broadcasting
Airbend also supports PHP attributes, allowing you to declaratively define your event’s broadcasting configuration directly above the class. This provides a cleaner and more expressive way to register channels and custom event names.
```php
use Doppar\Airbend\Support\Attributes\Broadcast;

#[Broadcast(channels: 'orders', as: 'order.shipped')]
class OrderShippedEvent extends BaseBroadcastEvent
{
    public function __construct(
        public $order
    ) {}
}
```
With this approach:

- `channels` defines where the event will be broadcast.
- `as` specifies the custom event name your client will listen for.

This offers an elegant alternative to manually implementing `broadcastOn()` and `broadcastAs()` inside the event class.

### Broadcasting Only to Other Clients
Airbend allows events to be broadcast only to other connected clients, excluding the sender. This is useful when the initiating client does not need to receive its own broadcasted event.

You can enable this behavior in two different ways:

### Using Attributes
You may configure `toOthers` directly inside the Broadcast attribute:
```php
use Doppar\Airbend\Support\Attributes\Broadcast;

#[Broadcast(channels: 'orders', as: 'order.shipped', toOthers: true)]
class OrderShippedEvent extends BaseBroadcastEvent
{
    public function __construct(
        public $order
    ) {}
}
```

This declaratively tells Airbend to broadcast the event only to other clients on the specified channel.

### Using the Class Property
Alternatively, you can control this behavior by defining the `$toOthers` property inside your event class:
```php
class OrderShippedEvent extends BaseBroadcastEvent
{
    /**
     * Broadcast only to others
     *
     * @var bool
     */
    protected bool $toOthers = true;

    public function __construct(
        public $order
    ) {}
}
```

Setting `$toOthers = true` ensures the event will not be echoed back to the client that triggered the broadcast.

### Conditional Broadcasting
Airbend allows you to conditionally determine whether an event should be broadcast at runtime. This is useful when you want to broadcast only under specific circumstances, such as premium orders, flagged actions, or system states.

To enable conditional broadcasting, simply override the `shouldBroadcast()` method in your event class:
```php
class OrderShippedEvent extends BaseBroadcastEvent
{
    public function shouldBroadcast(): bool
    {
        return $this->order->isPremium();
    }
}
```

If `shouldBroadcast()` returns false, the event is skipped and no message is sent to the channel.

### Broadcast with Specific Socket Exclusion

Airbend allows you to exclude a specific client connection from receiving a broadcast. This is especially useful when the same user triggers an event and you don’t want to broadcast the update back to the triggering client.

You can retrieve the current client’s socket ID from the request header `X-Socket-ID` and pass it to the broadcaster:
```php
$socketId = $request->header('X-Socket-ID');

Broadcast::except($socketId)->event(new OrderShippedEvent($order));
```
With this approach:
- All subscribed clients except the one with the provided socket ID will receive the event.
- Ideal for preventing duplicate UI updates on the client that initiated the action.

### Broadcasting to Multiple Channels
You can broadcast a single event to multiple channels by returning an array of channel names from the `broadcastOn()` method:
```php
public function broadcastOn(): array
{
    return [
        'alerts',
        'admin-notifications',
        'monitoring-dashboard'
    ];
}
```
This allows the same event to reach different parts of your system—for example, alert systems, admin panels, or monitoring dashboards—simultaneously.

## Private Channels
Private channels ensure that only authorized users can subscribe to sensitive or user-specific data streams. Airbend uses server-side authorization rules defined in your `routes/channels.php` file to validate access.

### Creating Private Event
Private events are broadcasted to channels that require authentication, ensuring that only authorized users receive sensitive data.
```php
<?php

namespace App\Events;

use Doppar\Airbend\Broadcasting\Events\BaseBroadcastEvent;

class MessageSent extends BaseBroadcastEvent
{
    public function __construct(
        public $message,
        public $conversation
    ) {}

    public function broadcastOn(): array
    {
        return ["private-conversation.{$this->conversation->id}"];
    }

    public function broadcastAs(): string
    {
        return 'message.sent';
    }

    public function broadcastWith(): array
    {
        return [
            'id' => $this->message->id,
            'conversation_id' => $this->conversation->id,
            'content' => $this->message->content
        ];
    }
}
```

Once a private event is created, you can broadcast it to authorized users on the defined private channel:
```php
use App\Events\MessageSent;

// Broadcast to all participants in the conversation
broadcast(new MessageSent($message, $conversation));
```

### Private Channel Authorization
Define the authorization logic in `routes/channels.php` to ensure that only authorized users can join the private conversation channel:
```php
use Doppar\Airbend\Broadcasting\Channel;
use Phaseolies\Http\Request;

Channel::authorize('private-conversation.{conversationId}', function (Request $request, int $conversationId) {
    // Implement your authorization logic here
});
```

### Listening to Private Conversation Channels
To listen to private conversation events from the client-side, you must connect using the private method and provide any necessary authentication headers:
```javascript
const airbender = new Airbender({
  host: "ws://127.0.0.1:6001",
  authHeaders: {
    "X-CSRF-TOKEN": csrfToken,
  },
});

const channel = airbender.private(`private-conversation.${conversationId}`);

channel.listen("message.sent", (data) => {
  console.log("Conversation data received:", data);
});
```

## Presence Channels
Presence channels allow you to track which users are currently subscribed to a channel. This is particularly useful for chat rooms, collaborative dashboards, or any real-time system where knowing who is online is important.

Presence channels are an extension of private channels and require authentication.
### Creating a Presence Event
You can create a broadcast event for a presence channel similar to a regular event:
```php
<?php

namespace App\Events;

use Doppar\Airbend\Broadcasting\Events\BaseBroadcastEvent;

class UserJoinedRoomEvent extends BaseBroadcastEvent
{
    public function __construct(
        public $user,
        public $room
    ) {}

    public function broadcastOn(): array
    {
        return ["presence-room.{$this->room->id}"];
    }

    public function broadcastAs(): string
    {
        return 'user.joined';
    }

    public function broadcastWith(): array
    {
        return [
            'user' => [
                'id' => $this->user->id,
                'name' => $this->user->name,
                'avatar' => $this->user->avatar_url,
            ],
            'text' => "{$this->user->name} joined the room",
            'joined_at' => now()->toIso8601String(),
        ];
    }
}
```

Now broadcast the event like this way
```php
$user = auth()->user();
$room = (object) ['id' => 1, 'name' => 'Test Room'];

broadcast(new UserJoinedRoomEvent($user, $room));
```

Presence channels require server-side authorization to verify users and provide user metadata:
```php
use Doppar\Airbend\Broadcasting\Channel;
use Phaseolies\Http\Request;

Channel::authorize('presence-room.{roomId}', function (Request $request, int $roomId) {
    // Only allow authenticated users who are part of the room
});
```

### Client-Side Integration
Use the Doppar Airbend client to connect to a presence channel, track online users, and handle real-time events. There are bunch of function to work with Airbend presence channel.

```javascript
const airbender = new Airbender({
    host: 'ws://127.0.0.1:6001',
    authHeaders: {
        'X-CSRF-TOKEN': "[[ csrf_token() ]]",
    },
});
```

### Connecting to a Presence Channel
To start interacting with a room, you first need to connect to its presence channel. The `join()` method connects your client to a channel and sets up authentication if required.

```javascript
const roomId = 1;
const roomChannel = airbender.join(`room.${roomId}`);
```

### Listening for Subscription Success
You can detect when the client has successfully subscribed to a channel by listening for the `subscribed` event.
This event is useful for confirming that authentication and channel binding were completed.
```javascript
channel.listen('subscribed', () => {
    console.log('Successfully subscribed to the channel');
});
```

### Listening for Broadcast Events
You can listen for server-side events broadcasted to the channel using `listen()`. This allows you to receive messages, updates, or notifications in real time.
```javascript
roomChannel.listen('user.joined', (message) => {
    console.log('Broadcast event received:', message);
});
```

### Tracking Members
Once connected, you can retrieve a list of all currently online users in the channel using the `here()` method. This is useful for rendering user lists or showing active participants.
```javascript
roomChannel.here((members) => {
    console.log('Currently online users:', members);
});
```

### Detecting Members Joining
The `joining()` method allows you to listen for new users entering the channel in real time. You can use this to update the UI or show notifications.
```javascript
roomChannel.joining((member) => {
    console.log(`${member.user_info.name} joined the room`);
});
```

### Detecting Members Leaving
Use the `leaving()` method to handle users who disconnect or leave the channel. This is helpful for maintaining accurate online user lists and showing exit notifications.
```javascript
roomChannel.leaving((member) => {
   console.log(`${member.user_info.name} left the room`);
});
```

### Sending Client-Side Events
The `whisper()` method sends events from your client to others in the same channel without going through the server. This is ideal for ephemeral updates like typing indicators.
```javascript
roomChannel.whisper('typing', {
    user_id: currentUserId,
    user_name: currentUser.name,
    typing: true
});
```

### Listening for Client-Side Events
Client-side events (whispers) are automatically prefixed with `client-.` Use `listen()` to detect these events from other users in the channel.
```javascript
roomChannel.listen('client-typing', (data) => {
   console.log(`${data.user_name} is typing...`, data);
});
```

### Listen to All Events on a Channel
You can listen to every event on a channel using the special `*` event name. This is extremely useful for debugging or when you want to log all incoming events.
```javascript
roomChannel.listen('*', (event, data) => {
    console.log('Event received:', event, data);
});
```

## Metrics Collector

Doppar Airbend includes a comprehensive metrics collection system that automatically tracks connection states, message flows, channel activity, performance metrics, and error rates. The `MetricsCollector` provides real-time insights into your WebSocket server's health and performance, making it easier to monitor, debug, and optimize your broadcasting infrastructure.

The metrics collector operates as a singleton instance, ensuring consistent metric tracking across your application. It integrates seamlessly with Redis for persistent storage, allowing metrics to be shared across multiple server instances in distributed deployments.

### Available Metrics

The metrics collector tracks the following categories of data:

#### Connection Metrics
- **Total Connections**: Cumulative count of all connections since server start
- **Active Connections**: Current number of connected clients
- **Failed Connections**: Number of connection attempts that failed

#### Message Metrics
- **Sent Messages**: Total number of messages sent to clients
- **Received Messages**: Total number of messages received from clients
- **Failed Messages**: Number of messages that failed to send

#### Channel Metrics
- **Subscriptions**: Total number of channel subscriptions
- **Unsubscriptions**: Total number of channel unsubscriptions
- **Broadcasts**: Total number of events broadcasted to channels

#### Performance Metrics
- **Memory Usage**: Tracks current and peak memory consumption (last 100 records)
- **Response Times**: Tracks operation response times for various operations (last 100 records)

#### Error Metrics
- **Connection Errors**: Number of connection-related errors
- **Broadcast Errors**: Number of broadcast operation failures
- **Authentication Errors**: Number of authentication failures

### Accessing Metrics

You can retrieve metrics using the `MetricsCollector` class. The collector automatically tracks all relevant events throughout the WebSocket server lifecycle.

#### Get All Metrics

To retrieve all collected metrics:

```php
use Doppar\Airbend\Monitoring\MetricsCollector;

$metrics = MetricsCollector::getMetrics();

// Returns an array with:
// - connections: ['total' => int, 'active' => int, 'failed' => int]
// - messages: ['sent' => int, 'received' => int, 'failed' => int]
// - channels: ['subscriptions' => int, 'unsubscriptions' => int, 'broadcasts' => int]
// - performance: ['memory_usage' => array, 'response_times' => array]
// - errors: ['connection_errors' => int, 'broadcast_errors' => int, 'authentication_errors' => int]
```

#### Get Specific Metric Category

To retrieve metrics for a specific category:

```php
$connectionMetrics = MetricsCollector::getMetricCategory('connections');
// Returns: ['total' => int, 'active' => int, 'failed' => int]

$messageMetrics = MetricsCollector::getMetricCategory('messages');
// Returns: ['sent' => int, 'received' => int, 'failed' => int]

$channelMetrics = MetricsCollector::getMetricCategory('channels');
// Returns: ['subscriptions' => int, 'unsubscriptions' => int, 'broadcasts' => int]
```

### Performance Statistics

The metrics collector provides detailed performance statistics, including response time analysis and memory usage tracking.

#### Get Performance Stats

```php
$performanceStats = MetricsCollector::getPerformanceStats();

// Returns:
// [
//     'response_times' => [
//         'count' => int,        // Number of recorded response times
//         'avg' => float,        // Average response time in seconds
//         'min' => float,        // Minimum response time in seconds
//         'max' => float,        // Maximum response time in seconds
//     ],
//     'memory' => [
//         'current_mb' => float,  // Current memory usage in MB
//         'peak_mb' => float,    // Peak memory usage in MB
//         'recent_mb' => float,  // Most recent recorded memory usage in MB
//     ]
// ]
```

#### Get Metrics Summary

For quick overviews and logging, you can retrieve a concise summary:

```php
$summary = MetricsCollector::getSummary();

// Returns:
// [
//     'connections' => int,              // Active connections
//     'total_messages' => int,           // Total sent + received messages
//     'error_rate' => float,             // Error rate percentage
//     'avg_response_time_ms' => float,  // Average response time in milliseconds
//     'memory_usage_mb' => float,        // Current memory usage in MB
// ]
```

### Manual Metric Recording

While the metrics collector automatically tracks most events, you can manually record custom metrics if needed:

Record Connection Events

```php
// Record a successful connection
MetricsCollector::recordConnection('connect');

// Record a disconnection
MetricsCollector::recordConnection('disconnect');

// Record a failed connection attempt
MetricsCollector::recordConnection('failed');
```

Record Message Events

```php
// Record sent messages (count defaults to 1)
MetricsCollector::recordMessage('sent');
MetricsCollector::recordMessage('sent', 5); // Record 5 messages

// Record received messages
MetricsCollector::recordMessage('received');

// Record failed messages
MetricsCollector::recordMessage('failed');
```

Record Channel Events

```php
// Record a subscription
MetricsCollector::recordChannel('subscription');

// Record an unsubscription
MetricsCollector::recordChannel('unsubscription');

// Record a broadcast
MetricsCollector::recordChannel('broadcast');
MetricsCollector::recordChannel('broadcast', 3); // Record 3 broadcasts
```

Record Errors

```php
// Record connection errors
MetricsCollector::recordError('connection');

// Record broadcast errors
MetricsCollector::recordError('broadcast');

// Record authentication errors
MetricsCollector::recordError('authentication');
```

### Performance Timing

The metrics collector can track the duration of operations for performance analysis:

```php
// Start timing an operation
MetricsCollector::startTiming();

// Perform your operation
// ... your code here ...

// End timing and record (returns elapsed time in seconds)
$elapsed = MetricsCollector::endTiming('operation_name');
```

The timing data is automatically included in performance statistics and can be retrieved via `getPerformanceStats()`.

### Memory Usage Tracking

Memory usage is automatically tracked when you call `getMetrics()`, but you can also manually record memory snapshots:

```php
MetricsCollector::recordMemoryUsage();
```

This records the current memory usage, peak memory usage, and timestamp. The collector maintains the last 100 memory usage records.

### Resetting Metrics

To reset all collected metrics (useful for testing or periodic resets):

```php
MetricsCollector::reset();
```

This will:
- Reset all counters to zero
- Clear performance timing records
- Clear memory usage history
- Clear all Redis-stored metrics

**Note**: Use this carefully in production, as it will permanently delete all historical metrics data.

### Redis Integration

The metrics collector automatically integrates with Redis when available, providing:

- **Persistent Storage**: Metrics are stored in Redis with a 1-hour TTL, allowing metrics to persist across server restarts
- **Distributed Metrics**: In multi-server deployments, metrics are aggregated across all instances
- **Atomic Operations**: Counter increments are performed atomically to ensure accuracy in concurrent environments

If Redis is unavailable, the metrics collector will continue to function using in-memory storage only. Metrics will be lost on server restart if Redis is not configured.
