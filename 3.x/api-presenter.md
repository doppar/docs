---
title: API Presenter
description: Doppar API Presenter page
meta:
  - name: keywords
    content: API Presenter, API Bundle, API Resource, API Collection
---
## Introduction
The API Presenter provides a clean, consistent way to transform your application’s data models into structured API responses.

Instead of returning raw Entity models or arrays directly from your controllers, the Presenter layer allows you to:

- **Control** exactly which fields are exposed.
- **Transform** and format data for API consumers.
- **Conditionally** include or merge attributes based on runtime logic.
- **Bundle collections** and paginate results with minimal boilerplate.

Presenters act as a view layer for your API, sitting between your models and the JSON output.
This approach ensures that:

- Your internal model structure can evolve without breaking API contracts.
- Sensitive or unnecessary fields can be excluded from the output.
- Complex transformations (e.g., date formatting, computed values) remain encapsulated in one place.

Doppar’s Presenter system has two main building blocks:

- **Presenter** – Wraps a single resource (model) and defines how it should be represented.
- **PresenterBundle** – Wraps a collection of resources, applying a Presenter to each and optionally handling pagination.

Together, they help you standardize API responses across your application while keeping presentation logic separate from business logic

## Presenter
Doppar makes it easy to create a new Presenter class using the built-in pool command:
```bash
php pool make:presenter UserPresenter
```

This will generate a basic Presenter skeleton in the `App\Http\Presenters` namespace:
```php
<?php

namespace App\Http\Presenters;

use Phaseolies\Support\Presenter\Presenter;

class UserPresenter extends Presenter
{
    /**
     * Transform the underlying resource into an array for API output.
     *
     * @return array
     */
    protected function toArray(): array
    {
        return [
            // Map your model properties here
        ];
    }
}
```

## Basic Usage of Presenter
The simplest way to use a Presenter is to wrap a single model or data object and return the transformed result.
A Presenter defines exactly which fields are exposed and how they are formatted, ensuring a consistent and predictable API response.
This separation of presentation logic from business logic keeps controllers clean and makes your API easier to maintain over time.

The simplest way to use a Presenter is to wrap a single model instance and return it directly from your controller.
```php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use App\Http\Presenters\UserPresenter;

class UserController extends Controller
{
    public function index()
    {
        $user = User::find(1);

        return new UserPresenter($user);
    }
}
```

Now the presenter implementation will be look like this

```php
<?php

namespace App\Http\Presenters;

use Phaseolies\Support\Presenter\Presenter;

class UserPresenter extends Presenter
{
    /**
     * Define the array representation of the user.
     *
     * @return array
     */
    protected function toArray(): array
    {
        return [
            'user_id' => $this->id,
            'name'    => $this->name,
            'email'   => $this->email,
        ];
    }
}
```

Output
```json
{
    "user_id": 1,
    "name": "John Doe",
    "email": "john@example.com"
}
```

### Excluding Fields from Output
You can exclude specific fields from a Presenter’s output using the `except()` method.
This is useful when you want to reuse the same Presenter but omit certain attributes for a particular response.
```php
return (new UserPresenter($user))->except('name', 'user_id');
```

Output
```json
{
    "email": "john@example.com"
}
```
This allows you to keep a single Presenter definition while adapting the output for different API contexts.

## Including Only Specific Fields
To limit the output to certain fields, use the `only()` method. This is useful when you need a minimal response containing just a subset of the available attributes.
```php
return (new UserPresenter($user))->only('name', 'user_id');
```

Output
```json
{
    "user_id": 1,
    "name": "John Doe"
}
```
Using `only()` helps keep responses lightweight when you don’t need all available fields.

### Presenter with Relationships
A Presenter can also include related data from your models. This allows you to expose nested resources (such as related models or collections) directly within your API response, without additional transformation logic in your controllers.
```php
$user = User::query()
    ->where('id', 1)
    ->embed(['posts', 'otp'])
    ->first();

return new UserPresenter($user);
```

Now the presenter implementation will be look like this
```php
protected function toArray(): array
{
    return [
        'user_id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'user_posts' => $this->posts, // linkMany/binToMany relationships
        'otp' => $this->otp, // linkOne/bindTo relationships,
    ];
}
```

> You don’t need to explicitly call `->embed(['posts', 'otp'])` in your query. Even without it, related data is available in your Presenter, and you can access it directly, for example: `$this->posts`. But we recommend, always use eager loading.

### Conditional Attributes in Presenter
Presenters provide helper methods to include or exclude attributes conditionally in the output array. The most commonly used methods are `when`, `mergeWhen`, and `unless`.
`when`
The when method allows you to include an attribute only if a given condition is true.
```php
'formatted_date' => $this->when(
    isset($this->created_at),
    fn() => $this->created_at->format('Y-m-d')
),
```
In this example, `formatted_date` will be included in the output only if `created_at` is set.

`mergeWhen`

The mergeWhen method conditionally merges an array of values into the presenter’s output.
```php
'meta' => $this->mergeWhen(true, [
    'admin_since' => $this->admin_since,
    'permissions' => $this->permissions
]),
```
Here, the `meta` array will be merged into the output only if the condition evaluates to `true`.
`unless`

The unless method is the inverse of when. It includes an attribute only if the condition is `false`.
```php
'guest_mode' => $this->unless(true, true),
```
In this example, `guest_mode` will not be included because the condition is true

### Nested or Embedded Presenters
In many applications, your models have relationships with other models, such as a User having many Posts or an Order having a related Product. Instead of manually formatting each related resource, you can use nested presenters to wrap related models or collections inside your main presenter

You can embed one presenter inside another to include related resources:
```php
protected function toArray(): array
{
    return [
        'user_id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'user_profile' => new ProfilePresenter($this->profile)
    ];
}
```
This wraps a single `user_profile` relational data using the `ProfilePresenter`. Works seamlessly with other Presenter features like `only()`, `except()`, and `when()`. This pattern allows you to build complex API responses while keeping each transformation modular and reusable.

## Alternative Way to Call Presenter
You can also instantiate your presenter directly using the `make()` method on the presenter class.
```php
UserPresenter::make(User::find(1));
```
> ⚠️ Keep in mind: when you use this approach, you cannot chain methods like `except()`, `only()`, etc.

## Presenter Bundle
Presenter bundle wraps a collection of resources, applying a Presenter to each and optionally handling pagination. The bundle in Doppar is designed to wrap a collection of models and apply a presenter to each item. This allows you to transform collections of data consistently, with support for pagination, selective fields, lazy serialization, and preserving keys.

### Basic Usage of Bundle
A PresenterBundle allows you to transform an entire collection of models using a specified presenter. When you instantiate a bundle, each item in the collection is automatically wrapped with the presenter class you provide, ensuring consistent formatting and transformation across the dataset.

You can wrap a collection of models and specify a presenter class:
```php
UserPresenter::bundle(User::all());
```
In this example, each User in the collection is transformed by UserPresenter, and the resulting bundle can be directly returned in a JSON response

## Lazy Serialization
By default, a `PresenterBundle` eagerly transforms all items in the collection when serializing. However, for large datasets, this can be resource-intensive. Enabling lazy serialization defers the transformation of each item until it is actually needed, improving performance and reducing memory usage.

You can enable lazy serialization by calling the `lazy()` method on your bundle:
```php
UserPresenter::bundle(User::all())->lazy()
```
When lazy serialization is enabled, the bundle uses a generator internally to `yield` each transformed item one at a time. This is particularly useful for API endpoints that return large collections, as it avoids loading all transformed data into memory at once.

## Excluding Fields from a Bundle
Sometimes you may want to omit certain fields from all items in a collection. The `except()` method allows you to specify which fields should be excluded when the `PresenterBundle` serializes each resource
```php
UserPresenter::bundle(User::all())->except(['user_id', 'name']);
```
In this example, the `user_id` and `name` fields will be removed from every user in the bundle. This is useful for hiding sensitive data or simplifying responses for certain API endpoints.

### Including Only Specific Fields
The `only()` method allows you to include a limited set of fields for every resource in the bundle. This is especially useful when you want to reduce payload size or return only relevant data to clients.
```php
UserPresenter::bundle(User::all())
        ->lazy()
        ->only(['user_id', 'name']);
```
This combination is ideal for APIs where you need selective and efficient serialization of large collections.

Here’s a clean diagram for comparing `only()`, `except()`, and `lazy()`:
| Method     | Purpose                                                                      | Example Usage                       | Notes                                                                        |
| ---------- | ---------------------------------------------------------------------------- | ----------------------------------- | ---------------------------------------------------------------------------- |
| `only()`   | Include only the specified fields in the output                              | `->only(['user_id', 'name'])`       | Filters each resource to return just these fields.                           |
| `except()` | Exclude specified fields from the output                                     | `->except(['email', 'created_at'])` | Removes fields from each resource while keeping all others.                  |
| `lazy()`   | Use generator-based serialization for better performance with large datasets | `->lazy()`                          | Serializes resources on-demand, reducing memory usage for large collections. |

## Pagination with Bundle
When working with paginated collections, you can wrap the paginated data in a `PresenterBundle` and return a structured response with metadata:
```php
UserPresenter::bundle(User::oldest('id')->paginate(6))
        ->except('email')
        ->lazy()
        ->toPaginatedResponse()
```

Here

- `lazy()` ensures resources are serialized on-demand, improving performance for large datasets.
- `toPaginatedResponse()` returns an array containing both the serialized data and pagination metadata (current page, total items, per page, URLs, etc.).

You don’t need to manually extract the data or metadata; `PresenterBundle` handles it automatically.

> This approach keeps your API responses consistent and optimized, even when dealing with large paginated collections.

You can also generate a paginated result directly using the `paginate()` method on the presenter
```php
UserPresenter::paginate(User::oldest('id'), 6);
```
> Note: when using `paginate()` this way, you cannot chain methods like only(), except(), etc