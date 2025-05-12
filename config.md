## Configuration
### Introduction
All configuration files for the Doppar framework are located in the `config` directory. These files define various options and settings that control how different parts of your application behave. Each configuration file is self-documented, so you are encouraged to explore and familiarize yourself with the available options.

Doppar stores the compiled configuration cache in the following path `storage/framework/cache/*`. Doppar usage this config data where needed from this complied cache file.

### Accessing Configuration Values
You can access configuration values from anywhere in your application using either the Config facade or the global `config()` helper function. Configuration keys are accessed using "dot" syntax, where the first part is the file name (without the .php extension) and the second part is the option key.

You can also specify a default value that will be returned if the requested configuration option does not exist.
```php
<?php

use Phaseolies\Support\Facades\Config;

// Access a config value using the facade
$value = Config::get('app.name');

// Access the same config value using the global helper
$value = config('app.name');

// Provide a default if the key doesn't exist
$value = config('app.name', 'Doppar');
```

### Set Configuration Values
During runtime, you may also set configuration values dynamically using the Config facade or the global `config()` helper function. This can be useful in situations where you need to override or define configuration values on the fly—such as in testing, bootstrapping, or service providers.

```php
<?php

use Phaseolies\Support\Facades\Config;

// Set a configuration value using the facade
Config::set('app.timezone', 'UTC');

// Set a configuration value using the global helper
config(['app.timezone' => 'UTC']);
```

> These changes are applied at runtime and persists in the configuration cached files.

### Clear Config Cache
When you make changes to configuration files or add new ones, Doppar may continue using cached versions. To clear the configuration cache and force Doppar to reload the configuration files from the config directory, run the following command:
```bash
php pool config:clear
```
This command removes the cached config file located at: `storage/framework/cache/*`. Clearing the cache is essential when adding new configuration files or updating existing ones, especially during development if you face unexpected result.

### Cache Configuration
After clearing the cache, you can regenerate it to improve performance in production environments. Caching all of your configuration files into a single file speeds up application bootstrapping.

To cache the configuration:
```bash
php pool config:cache
```
This command will compile all configuration files into a single cache file and store it at: `storage/framework/cache/*`
> Always run `php pool config:cache` after deploying changes to your configuration files in production.

