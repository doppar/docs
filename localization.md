## Localization
### Introduction
Localization is a key feature in Doppar that allows your application to support multiple languages. It enables you to display content in the user's preferred language by using translation files stored in the `lang` directory. Doppar provides several tools to work with localization, including facades and helper functions.

### Setting the Locale
To change the application's active locale (language), you can use the `App::setLocale()` method. This sets the language that will be used when retrieving translation strings.

```php
use Phaseolies\Support\Facades\App;

App::setLocale('en'); // Sets the application locale to English
App::setLocale('fr'); // Sets the application locale to French
```
The locale you set must correspond to a subdirectory in your lang folder (e.g., lang/en, lang/fr), where the respective language files are stored.

You can also set the application's locale using the `app()` helper, which provides a convenient shortcut for accessing services in the container. Here's how you would set the locale this way:

```php
app()->setLocale('en'); // Set locale to English
app()->setLocale('fr'); // Set locale to French
```
This is functionally identical to using `App::setLocale()`, and is often used when you want to avoid importing the `App` facade directly.

### Getting the Current Locale
To retrieve the currently active language (locale) of your application, use the `App::getLocale()` method. This is helpful when you need to conditionally render content based on the current language setting.
```php
use Phaseolies\Support\Facades\App;

$currentLang = App::getLocale();

echo $currentLang; // Output: "en", "fr", etc.
```
This method returns the locale currently being used by the application, typically corresponding to a folder name under the lang directory (like lang/en or lang/fr).

### Fetching Translations
Doppar offers a flexible and consistent API for retrieving language lines from your localization files. This allows your application to support multiple languages seamlessly by defining language strings in separate files for each locale. These language files are typically located in the `lang/{locale}` directory and return an array of key-value pairs.

Once you’ve defined your language strings, you can retrieve them using several methods provided by Doppar. These methods support parameter replacement, so you can inject dynamic values into your translation strings.

Below is an example language file located at `lang/en/messages.php`:
```php
<?php

return [
    'welcome' => 'Thank you for choosing Doppar :version. Build something amazing',
];
```
You can retrieve this translation using different approaches:

#### Using the lang() Helper
```php
lang()->get('messages.welcome', ['version' => Application::VERSION]);
// or
lang()->trans('messages.welcome', ['version' => Application::VERSION]);
```

#### Using the Lang Facade
```php
use Phaseolies\Supports\Facades\Lang;

$message = Lang::trans('messages.welcome', ['version' => Application::VERSION]);
```

#### Using the trans() Helper
```php
$message = trans('messages.welcome', ['version' => Application::VERSION]);
```

All of these methods will return:
```markdown
"Thank you for choosing Doppar X.X.X. Build something amazing"
```

