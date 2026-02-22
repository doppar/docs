---
title: Flash Message
description: Doppar flash message page
meta:
  - name: keywords
    content: flash-message
---

## Flash Message
### Introduction
Flash messages are short-lived session messages used to give users feedback after certain actions — like form submissions, authentication, or redirects. These messages persist only for the next HTTP request and are automatically removed after they are read.

Doppar provides an elegant and expressive syntax to retrieve flash messages during redirects, making it easy to give users a seamless experience.

## Setting Flash Messages
There are several convenient ways to flash data to the session during a redirect. Doppar supports dynamic flash methods like `withSuccess`, `withError`, `withInfo`, and more like `withAnyKey` — or you can use the generic `with()` method.
```php
return redirect('/register')->withSuccess('your message');
```
Or using the generic `with()` method
```php
return redirect('/register')->with('success', 'your message');
```
You use naming route while redirecting. Flashing a message with a named route or using `to()` method
```php
return redirect()->route('register')->withSuccess('your message');
return redirect()->to('/register')->withSuccess('your message');
```

## How It Works Internally
When you call methods like, `withAnyKey` dynamically maps it to a flash key like `any_key`, and stores the message in the session with a `any_key` key. This ensures the message is only available on the next request, and then automatically cleared.

The dynamic method resolution is handled via the `__call()` method on RedirectResponse, which maps any `withXyz()` call to with('xyz', ...) using `Str::snake()`.

So you can use like
```php
return back()->withAnyKey('your message');
```

You can directly flash a key-value pair to the session using the `flash()` method:
```php
session()->flash('success', 'your message');

return redirect('/register');
```
This is useful when you're not chaining off a redirect or doing session manipulation manually.

## Displaying Flash Messages
In your Odo template, simply check if a flash key exists using `session()->has()`, and then display it using `session()->pull()`:
```html
#if (session()->has('success'))
    <div class="alert alert-success">
        [[ session()->pull('success') ]]
    </div>
#endif

#if (session()->has('error'))
    <div class="alert alert-danger">
        [[ session()->pull('error') ]]
    </div>
#endif
```
You can wrap these in partials or components to keep your views clean and reusable.
