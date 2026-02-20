---
title: Asset bundling
description: Doppar asset bundling page
meta:
  - name: keywords
    content: asset-bundling
---

## Asset Bundling
In Doppar, managing front-end assets like CSS and JavaScript is straightforward thanks to the `enqueue()` function. This helper simplifies referencing your static files—such as stylesheets, scripts, or images—by generating the correct path from the `public/` directory.

### How It Works
The `enqueue()` function looks inside your project’s `public/` folder and returns the proper URL for any file you pass to it. This ensures that your assets are correctly linked, even if the project is running in a subdirectory or behind a proxy.

### Basic Usage
To include a CSS file from the root of the `public/` directory:
```html
<link rel="stylesheet" href="[[ enqueue('style.css') ]]">
```
This will output something like:
```html
<link rel="stylesheet" href="http://example.com/style.css">
```
###  Using Subfolders
If your CSS, JS, or other assets are organized inside folders within `public/`, you can specify the relative path:
```html
<link rel="stylesheet" href="[[ enqueue('assets/style.css') ]]">
```
Here, `assets` is simply a folder inside the `public/` directory.

Summery
- The file path passed to `enqueue()` is relative to the `public/` directory.
- Do not include the full URL or domain name—enqueue() takes care of that.
- This method is commonly used in templates for loading CSS and JS efficiently.