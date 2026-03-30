---
title: Deployment
description: Doppar deployment page
meta:
  - name: keywords
    content: deployment
---

## Deployment
### Introduction

Deploying a Doppar application is straightforward and optimized for both development and production environments. This guide outlines the essential steps to get your Doppar app running on a live server.

## Server Requirements
The Doppar framework has a few essential system requirements. Before installing or deploying Doppar, make sure your web server environment meets the minimum PHP version and required extensions:

### Minimum PHP Version
PHP >= 8.3
### Required PHP Extensions
- Ctype PHP Extension
- cURL PHP Extension
- DOM PHP Extension
- Fileinfo PHP Extension
- Filter PHP Extension
- Hash PHP Extension
- Mbstring PHP Extension
- OpenSSL PHP Extension
- PCRE PHP Extension
- PDO PHP Extension
- Session PHP Extension
- Tokenizer PHP Extension
- XML PHP Extension

> ⚠️ Your server must be configured to meet all of the above requirements for Doppar to run properly.

## Server Configuration
### Nginx
If you're deploying your Doppar application to a server running Nginx, you can start with the sample configuration below. Be aware that this configuration may require adjustments based on your server’s specific setup.

> Ensure your web server routes all incoming requests to your application's public/index.php file. Do not move the index.php file into the project root. Doing so exposes sensitive configuration and internal files to the public internet, creating serious security risks.
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name example.com;
    root /srv/example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";

    index index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ ^/index\.php(/|$) {
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
        fastcgi_hide_header X-Powered-By;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

## Directory Permissions
Doppar will need to write to the `storage` directories, so you should ensure the web server process owner has permission to write to these directories.


## Optimization
To optimize the performance of your Doppar application in production, it's important to use the available caching commands. By default, the environment variable `APP_ROUTE_CACHE` is set to `false`, which disables route caching. This is ideal for development, where routes are frequently modified. However, in a production environment, you should set `APP_ROUTE_CACHE=true` to enable route caching for improved performance.

### Boost Performance By Caching
When deploying your application to production, performance and consistency are critical. To ensure everything runs smoothly, `php pool` provides a set of optimization commands that should be executed after code or configuration changes.

You should run `php pool boost` command **after each deployment**, especially when:

- Routes have been modified.  
- Configuration files (`config/*.php`) have changed.  
- Odo views have been updated.  
- You want to ensure your production environment is using cached, optimized resources.

### View Cache
Compiles and caches all odo views, which reduces the time spent rendering views on every request.
```bash
php pool view:cache
```
This command will precompile all odo templates and store them in a cache for faster rendering.

### Route Cache
Caches the entire route definition to speed up routing by eliminating the need to parse the route files with each request.
```bash
php pool route:cache
```
Running this command will generate a cached file containing all of your route definitions, speeding up request handling

### Config Cache
Caches all the configuration files into a single file, which helps reduce the overhead of loading and parsing configuration on every request.
```bash
php pool config:cache
```
This command will combine all your configuration files and cache them into a single, optimized file, improving application performance.

### Clearing Cache
If you need to make changes to your views, routes, or configuration files, you should clear the cache before re-caching to ensure the changes take effect. Use the following commands to clear the caches:
```bash
php pool view:clear
php pool route:clear
php pool config:clear
```

By using these commands, you ensure that Doppar runs at peak performance in production environments, making the application faster and more efficient for users.

## Debug Mode
The debug option in your `config/app.php` configuration file controls the level of error information displayed to the user. By default, this option respects the value set in the `APP_DEBUG` environment variable, which is stored in your application's `.env` file.

In a production environment, it is crucial to set the `APP_DEBUG` variable to false. When `APP_DEBUG` is set to true, detailed error messages and stack traces may be displayed to users, which can inadvertently expose sensitive information, such as database credentials, API keys, or other confidential configuration values. This can pose a serious security risk, as malicious users could potentially exploit this information.

To prevent this, ensure that the following line is set in your .env file for production:
```php
APP_DEBUG=false
```

This will ensure that detailed error messages are not shown to end users, providing a cleaner and more secure experience. Instead, general error messages or custom error pages should be displayed to end users in production.

For local development, you can set `APP_DEBUG=true` to help with debugging, as it will show full error details, including stack traces, which are useful for identifying and resolving issues quickly.