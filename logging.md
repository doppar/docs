## Logging
### Introduction
Doppar provides a comprehensive logging system to facilitate application debugging and monitoring. By leveraging the powerful Monolog library, Doppar offers flexible logging capabilities, allowing you to record various events and messages to different channels.

### Configuration
All logging configuration settings for your Doppar application are located in the `config/log.php` file. This file defines the available log channels and allows you to customize how logs are written and stored. Make sure to explore each channel and its settings to configure logging according to your needs.

By default, Doppar logs messages using the stack channel, which combines multiple log channels into one unified stream. This is useful for sending logs to multiple destinations simultaneously. To learn more about setting up and customizing stack channels, refer to the official documentation below.

You can customize the logging behavior by modifying the `config/log.php` file and the `.env` file. This allows you to select your preferred logging channel, adjust log levels, and configure other logging options.

### Logging Channels:
Doppar supports three primary logging channels, configurable via the `config/log.php` file and your .env environment variables:

- stack: Allows you to group multiple log handlers into a single channel.
- daily: Creates a new log file each day, useful for managing large log volumes.
- single: Writes all log messages to a single file `(storage/logs/doppar.log)`.
- slack: Creates log each day in slack channel

### Writing Log Messages
Doppar provides a convenient Log Facades class to simplify the logging process. To log a message, simply use the following syntax:
```php
use Phaseolies\Support\Facades\Log;

Log::info('Howdy');
```

You may write information to the logs using the Log facade. As previously mentioned, the logger provides the eight logging levels: `emergency`, `alert`, `critical`, `error`, `warning`, `notice`, `info` and `debug`:
```php
use Phaseolies\Support\Facades\Log;

Log::warning('Howdy');
Log::notice('Howdy');
Log::alert('Howdy');
Log::emergency('Howdy');
Log::error('Howdy');
Log::debug('Howdy');
Log::critical('Howdy');
```

### Log channel
Sometimes you may wish to log a message to a channel other than your application's default channel. You may use the channel method on the Log facade to retrieve and log to any channel defined in your configuration file:

You can use channel to show Log message, like you want to show Log a daily file or slack channel or single channel, you can use like below
```php
use Phaseolies\Support\Facades\Log;

Log::channel('stack')->info('Howdy');
Log::channel('daily')->warning('Howdy');
Log::channel('single')->notice('Howdy');
Log::channel('slack')->alert('Howdy');
```

### Log Helper Functions
Instead of explicitly specifying a channel every time, you can use Doppar’s global log helper functions. These will use the default log channel (usually stack) defined in your logging configuration:
```php
info('Howdy');
warning('Howdy');
notice('Howdy');
alert('Howdy');
emergency('Howdy');
error('Howdy');
debug('Howdy');
critical('Howdy');
```
These helpers make it easy to log messages with the appropriate severity level without additional boilerplate.
