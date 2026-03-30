---
title: Relationships
description: Doppar Entity Relationships page
meta:
  - name: keywords
    content: Entity Relationships
---

  - [One to One / Link One](#one-to-one-link-one)
  - [One To One Inverse / Bind To](#one-to-one-inverse-bind-to)
  - [One to Many / Link Many](#one-to-many-link-many)
  - [Eager Loading, Lazy Loading](#eager-loading-lazy-loading)
  - [Many To Many / bindToMany ](#many-to-many-bind-to-many)
  - [Nested Relationship](#nested-relationship)
  - [Quering Relationship Existence](#quering-relationship-existence)
  - [Quering Relationship Missing](#quering-relationship-missing)
  - [Relationship Count](#relationship-count)

## Relationships
### Introduction
The Doppar Entity relationship system provides a powerful way to represent and manage connections between database tables in an object-oriented manner.

In real-world applications, tables are often interconnected. For instance, a library system might have Book records linked to multiple Author records, or a Customer may have multiple Orders. Doppar Entity ORM makes it simple to define and work with these relationships, whether one-to-one, one-to-many, or many-to-many.

With Entity, managing related data is intuitive: you can retrieve, filter, and manipulate related records seamlessly, while taking advantage of features like eager loading, nested relationships, and relationship-based query constraints.

Let’s explore the types of relationships supported by Doppar, their advantages, and how they streamline working with complex relational data compared to traditional methods.

  - [One to One / Link One](#one-to-one-link-one)
  - [One To One Inverse / Bind To](#one-to-one-inverse-bind-to)
  - [One to Many / Link Many](#one-to-many-link-many)
  - [Many To Many / bindToMany ](#many-to-many-bind-to-many)

## Defining Relationships
Entity relationships are defined as methods on your Entity model classes. Since relationships also serve as powerful query builders, defining relationships as methods provides powerful method chaining and querying capabilities. For example, we may chain additional query constraints on this posts relationship:
```php
$user = User::find(1);

$user->posts()->where('status', true)->get();
```

But, before diving too deep into using relationships, let's learn how to define each type of relationship supported by Entity.

## One to One / Link One
A one-to-one relationship is one of the simplest types of database associations. For instance, a `User` model may be linked to a single `Otp` model. To define this relationship, you can add an otp method to the User model. This method should invoke the `linkOne` method and return its result. The `linkOne` method is provided by the `Phaseolies\Database\Entity\Model` base class, making it easily accessible within your model.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class User extends Model
{
    /**
     * Get the otp associated with the user.
     */
    public function otp()
    {
        return $this->linkOne(Otp::class, 'user_id', 'id');
    }
}
```

The first argument passed to the linkOne method is the name of the related model class. Second argument is the Foreign key on related table (otps.user_id) and the third one is Local key on this table (users.id). All the arguments is mandatory.

```php
$otp = User::find(1)->otp;
```

### Fetch Users with Associated OTP
In Doppar managing relationships between models is made seamless through expressive query methods. One powerful feature offered by Doppar is the `embed` method, which allows you to eager-load related models directly within your queries.

This is especially useful when you want to retrieve a primary model (like User) along with associated models (like Otp) in a single, optimized query.

To fetch all users along with their associated OTP data, you can use the embed method without any constraints:
```php
User::embed('otp')->get()
```
This is simple and useful when you need all fields from the otp table.

For better performance and cleaner results, especially in APIs, you can limit the fields returned from the related otp model. Doppar’s embed method accepts a closure to customize the subquery:

To retrieve all users along with their associated OTPs, including only specific fields from the OTP model (id, otp, and user_id), use the following query:
```php
User::query()
    ->embed('otp', function ($query) {
        $query->select('id', 'otp', 'user_id');
    })
    ->get();
```

This approach works great when you’re building APIs that require user data along with their verification details, such as OTPs, but without unnecessary fields like timestamps or status flags.

> You must always include the relational key, such as `user_id`, when selecting specific fields by passing callback. Otherwise, the relation will not work properly.

## One To One Inverse / Bind To
So, we can access the Otp model from our User model. Next, let's define a relationship on the Otp model that will let us access the user that owns the otp. We can define the inverse of a `linkOne` relationship using the `bindTo` method:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Otp extends Model
{
    /**
     * Get the user that owns the otp.
     */
    public function user()
    {
        return $this->bindTo(User::class, 'id', 'user_id');
    }
}
```

The first argument passed to the bindTo method is the name of the related model class. Second argument is the Local key on related table (users.id) and the third one is Foreign key on this table (otps.user_id). All the arguments is mandatory.

Now you can get the user that owns the otp
```php
$otp = Otp::find(1);

$otp->user;
```

You can add extra condition like this as a method query builder.
```php
$otp = Otp::find(1);

$activeUser = $otp->user()
    ->where('status', true)
    ->first();
```

## One to Many / Link Many
A one-to-many relationship is ideal for scenarios where a single model serves as the parent to multiple related models. For example, a single blog post might have countless comments attached to it. To define this type of relationship in your model, simply create a method that represents the connection. As with all relationships in Entity, this method encapsulates the logic and structure of the association, making your code expressive and intuitive.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments()
    {
        return $this->linkMany(Comment::class, 'post_id', 'id');
    }
}
```
The first argument passed to the linkMany method is the name of the related model class. Second argument is the Foreign key on related table (comments.post_id) and the third one is Local key on this table (posts.id). All the arguments is mandatory.

Once the relationship method has been defined, we can access the collection of related comments by accessing the comments property. Remember, since Entity provides "dynamic relationship properties", we can access relationship methods as if they were defined as properties on the model:

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

Since all relationships also serve as query builders, you may add further constraints to the relationship query by calling the comments method and continuing to chain conditions onto the query like this:
```php
$comment = Post::find(1)
    ->comments()
    ->where('title', 'bar')
    ->first();
```
If you want to load all the posts with their associated comments, you can use `embed()` method like this. The following line demonstrates how to perform eager loading using the embed method to load related models efficiently:
```php
Post::embed('comments')->get();
```
This approach ensures that all related articles for each User are loaded in a single query, By retrieving the posts and their associated comments in one go, you significantly improve performance, especially when dealing with large datasets or nested relationships.

## One to Many Inverse / Bind To
Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `LinkMany` relationship, define a relationship method on the child model which calls the belongsTo method:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post()
    {
      return $this->bindTo(Post::class, 'id', 'post_id');
    }
}
```

Once the relationship has been defined, we can retrieve a comment's parent post by accessing the post "dynamic relationship property":
```php
use App\Models\Comment;

$comment = Comment::find(1);

$comment->post?->title ?? 'default';
```
In the example above, Entity will attempt to find a Post model that has an id which matches the post_id column on the Comment model.

## Eager Loading, Lazy Loading
Efficiently managing related data is crucial in applications that rely on relational databases. Doppar Entity ORM provides mechanisms to control when and how related models are fetched, helping you optimize performance and avoid common pitfalls such as the `N+1` query problem.

### Lazy Loading
By default, Doppar uses lazy loading. This means that related models are only fetched when you access the relationship property. While convenient, lazy loading can lead to multiple queries when iterating over collections of parent models.

Example:
```php
$users = User::all(); // 1 query

foreach ($users as $user) {
    echo $user->posts->count(); // 1 query per user
}
```
If there are `100` users, lazy loading will execute `101` queries in total:

| Operation                 | Queries |
| ------------------------- | ------- |
| Fetch all users           | 1       |
| Fetch posts for each user | 100     |
| **Total**                 | 101     |

This is the classic `N+1` query problem, where `N` is the number of parent records.

### Eager Loading
Doppar solves this problem with eager loading using the `embed()` method. Eager loading retrieves the parent models and their related models in a single optimized query.

Example:
```php
$users = User::embed('posts')->get();

foreach ($users as $user) {
    echo $user->posts->count(); // No additional queries
}
```

With eager loading, only 2 queries are executed:

| Operation                         | Queries |
| --------------------------------- | ------- |
| Fetch all users                   | 1       |
| Fetch posts for all users (embed) | 1       |
| **Total**                         | 2       |

This approach dramatically reduces database load and improves performance, especially for large datasets.

### Nested Eager Loading
Doppar also supports nested eager loading, allowing you to preload relationships multiple levels deep in a single, efficient workflow.

Example:
```php
$posts = Post::query()->embed('comments.user')->get();
```
This query:
- Fetches all posts (1 query)
- Fetches all related comments (1 query)
- Fetches all users associated with the comments (1 query)

Total queries: 3

Without eager loading, lazy loading would trigger 1 query for posts + N queries for comments + M queries for users, which could easily run into hundreds or thousands of queries. You can debug this situation using [doppar insight](doppar-insight) package.

By strategically using `embed()` for eager loading and nesting related relationships, Doppar Entity ORM ensures your applications remain performant while maintaining clean, readable code.

## Many to Many / Bind To Many
Many-to-many relationships are a bit more involved than linkOne or linkMany relationships. A common example is the relationship between posts and tags. A single post can have multiple tags, and each tag can be associated with multiple posts. For instance, a blog post might be tagged with "Doppar" and "PHP", and those same tags might be used on other posts as well. So, a post has many tags, and a tag belongs to many posts.

### Table Structure
To define this relationship, you'll need three database tables: posts, tags, and post_tag. The post_tag table acts as a pivot or intermediate table that links posts and tags together. It is typically named by combining the singular form of both related models in alphabetical order—hence post_tag.

This pivot table contains two foreign keys: post_id and tag_id. These columns connect the posts and tags tables.

It’s important to note that because a tag can belong to many posts (and vice versa), you cannot simply add a post_id column to the tags table. Doing so would limit each tag to a single post, defeating the purpose of a many-to-many relationship. Instead, the post_tag table provides the flexibility needed for this shared association.

Here’s a quick summary of the table structure:
```markdown
posts
    id - integer
    title - string

tags
    id - integer
    name - string

post_tag
    post_id - integer
    tag_id - integer
```

## Define Many to Many Relationships
Doppar ORM supports many-to-many relationships through an intermediate pivot table. Here's how to implement and work with them. Assume we have a Post model and a Tag model, where a many-to-many relationship exists between them. This means one post can have multiple tags, and one tag can be associated with multiple posts.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Post extends Model
{
    public function tags()
    {
        return $this->bindToMany(
            Tag::class,  // Related model
            'post_id',   // Foreign key for posts in pivot table (post_tag.post_id)
            'tag_id',    // Foreign key for tags in pivot table (post_tag.tag_id)
            'post_tag'   // Pivot table name
        );
    }
}
```

Once the relationship is defined, you may access the post's tags using the tags dynamic relationship property:

```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}
```
You can easily retrieve all the related tag names of a post as a plain array using Entity’s pluck method:
```php
$tagNames = $post->tags->pluck('name')->toArray();
```

Or, you can achieve the same result using the map property shortcut:
```php
$tagNames = $post->tags->map->name->toArray();
```

This is a concise way to get just the tag names without loading entire models. Perfect for transforming data or passing it to views or APIs.

## Retrieving Pivot Table Columns
As you have already learned, working with many-to-many relations requires the presence of an pivot table. After accessing this relationship, we may access the intermediate table using the pivot attribute on the models as follows
```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    echo $tag->name;
    echo $tag->pivot->post_id;
}
```

## Defining the Inverse of the Relationship
To define the "inverse" of a many-to-many relationship, you should define a method on the related model which also returns the result of the `bindToMany` method. To complete our post / tag example, let's define the users method on the Tag model:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Entity\Model;

class Tag extends Model
{
    /**
     * The posts that belong to the tag.
     */
    public function posts()
    {
        return $this->bindToMany(
            Post::class, // Related model
            'tag_id',    // Foreign key for tags in pivot table
            'post_id',   // Foreign key for posts in pivot table
            'post_tag'   // Pivot table name
        );
    }
}
```

Now you can get the posts that belong to the tag like this way

```php
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
  //
}
```

Or you can load all the tags with their associated posts
```php
Tag::query()->embed('posts')->get();
```

## Many to Many Relationships
### link()
In Doppar, the `link()` method is used to associate records in a many-to-many relationship. This method adds entries to the pivot table, establishing a connection between related models
```php
$post = Post::find(1);

// Link tags with IDs 1, 2, and 3
$post->tags()->link([1, 2, 3]);
```

### unlink()
In Doppar, the `unlink()` method is used to unlink specific records from a many-to-many relationship. It removes the association between the current model (e.g., Post) and the related model (e.g., Tag) by removing the corresponding entries from the pivot table.
```php
$post = Post::find(1);

// Unlink tags with IDs 1, 2, and 3
$post->tags()->unlink([1, 2, 3]);
```

If you simply call unlink(), it will delete all the tags
```php
$post = Post::find(1);

// unlink all tags
$post->tags()->unlink();
```

### relate()
In Doppar, the `relate()` method is used to sync the relationships between models in a many-to-many relationship. The relate() method will attach the provided IDs and can optionally detach existing relationships, depending on the second argument passed.
```php
$post = Post::find(1);

$post->tags()->relate([1, 2, 3]);

$changes = $post->tags()->relate([1, 2, 4]); // 3 will be removed
```

#### link without unlinking
You can also use the `relate()` method without unlinking existing relationships by passing `false` as the second argument.
```php
$post->tags()->relate([1, 2, 3], false);
```

### Syncing with Pivot Data Using
In Doppar, the `relate()` method not only allows you to sync records in a many-to-many relationship, but also provides the ability to attach additional data to the pivot table. This is useful when you need to store extra attributes (such as timestamps or other metadata) along with the relationship between two models.
```php
$post = Post::find(1);

$post->tags()->relate([
    1 => ['created_at' => now()],
    2 => ['featured' => true],
    3 => ['meta' => ['color' => 'blue']]
]);
```

## Nested Relationship
In the Doppar framework, you can effortlessly retrieve deeply nested relationships using the powerful embed method. This allows you to preload related models at multiple levels, preventing the "N + 1" query issue and ensuring optimized performance across complex data structures.

For example, if you want to fetch all posts along with their comments, the replies to those comments, and the users who wrote the replies, you can use the following nested eager loading query:
```php
Post::query()->embed('comments.reply.user')->get();
```

Explanation:
- **comments:** Loads all comments related to each post.
- **reply:** Loads replies for each comment (assuming a reply is a nested relationship under a comment).
- **user:** Loads the user who authored each reply.

## Specific Column Selection
The `embed()` method in Doppar ORM allows you to eager load related models and even specify which columns to load for each relationship. This helps optimize queries by only selecting the necessary data.
```php
User::query()
    ->embed([
        'comments.reply',
        'posts' => function ($query) {
            $query->select('id', 'title', 'user_id');
        }
    ])
    ->get();
```

You can optimize your queries by fetching only the columns you need from related models. To do this, simply specify the desired columns after the relation name using the `relation:columns` syntax.

For example:
```php
User::query()
    ->embed([
        'comments.reply',
        'posts:id,title,status'
    ])
    ->get();
```
In this example:
- The `comments.reply` relationship will be fully loaded.
- The posts relationship will only retrieve the `id`, `title`, and `status` columns — reducing unnecessary data and improving query performance.

You can also select specific columns from `nested (deep) relationships` using the same syntax.
Simply include the column list after the full relation path.
```php
User::query()
    ->embed([
        'comments.reply:id,body',
        'posts:id,title,status',
    ])
    ->get();
```
In this example:
- `comments.reply:id,body`
  - Loads the reply relation of each comment, but only includes the `id` and `body` columns.
- `posts:id,title,status`
  - Loads each user's posts, selecting only the `id`, `title`, and `status` fields.

### Applying Conditions
You can also apply query constraints to your embedded relationships by passing a closure. This allows you to filter, sort, or otherwise modify the relationship query before it is executed.
```php
Post::query()
    ->embed([
        'comments:id,body' => function ($query) {
            $query->where('approved', true)
                  ->orderBy('created_at', 'DESC');
        },
    ])
    ->get();
```

In this example:
- The comments relation is embedded, but only the `id` and `body` columns are selected.
- The closure applies additional conditions:
  - Only `approved` comments are retrieved.
  - Comments are ordered by `created_at` in descending order (newest first).

### Embedding Relationships with Conditions, Limits, and Counts
You can combine multiple features — such as selecting specific columns, applying conditions, limiting results, and counting related records — all within the `embed()` and `embedCount()` methods.

For example:
```php
Post::query()
    ->where('id', 1)
    ->select('id', 'title', 'user_id', 'category_id')
    ->embed([
        'comments:id,body,created_at' => function ($query) {
            $query->where('status', true)
                  ->limit(2)
                  ->oldest('created_at');
        },
        'tags',
        'user:id,name',
        'category:id,name',
    ])
    ->embedCount('comments')
    ->where('status', true)
    ->first();
```
Explanation
 - `comments:id,body,created_at`
   - Loads only the `id`, `body`, and `created_at` columns from the comments relation.
   - The embedded query:
     - Filters comments where `status = true`
     - Limits the result to `2` comments
     - Orders them by `created_at` (oldest first)

- `tags` Loads all related tags without additional constraints.
- `user:id,name` Loads the `post’s author`, selecting only the `id` and `name` columns.
- `category:id,name` Loads the category relation, selecting only the `id` and `name`.
- `embedCount('comments')`
Adds a `comments_count` attribute with the total number of related comments.
- `Final filters:`
  - The main query filters `posts where id = 1 and status = true`, then retrieves the first matching record.

This approach offers precise control over both the parent and related models — letting you build rich, optimized data responses in a single query.

### Fetching Multiple Relationships
This query retrieves users along with their related articles and address using the embed method. By embedding multiple relationships, it ensures that all necessary data is fetched in a single query, improving efficiency and reducing additional database calls.
```php
User::query()
  ->embed(['posts','address'])
  ->get();
```

## Quering Relationship Existence
When fetching model records, there are times you might want to filter results based on whether a certain relationship exists. For example, suppose you need to retrieve only the blog posts that have at least one comment attached. In such cases, you can use the present method, passing in the name of the relationship to constrain the query to only those records that have related entries. This approach ensures your results are contextually relevant and tied to meaningful relationships.

In Doppar ORM, the present() method can be used to load relationships that are present (i.e., do not exist) in the model, or when you want to ensure related data is included, even if it is not empty.
```php
Post::query()->present('comments')->get();
```
This will retrieve all posts that have at least one comment.

The `present()` method can be used to load a relationship with custom query conditions. You can define specific conditions inside the closure passed to present() to filter the related data.
```php
Post::query()
    ->present('comments', function ($query) {
        $query->where('comment', 'doppar is awesome')
            ->where('created_at', NULL);
    })
    ->get();
```

You can also use `orPresent()` like this way
```php
User::query()
    ->whereYear('created_at', 2025)
    ->orPresent('posts', function ($query) {
        $query->where('status', true);
    })
    ->get();
```

You can use `orPresent()` to add an `OR EXISTS` condition to your query. This is helpful when combining relationship existence logic with other filters.

The `present()` methods let you filter parent records depending on whether a relationship (or chain of relationships) has related records matching specific conditions.

With nested relationship support, you can now query across multiple levels using dot notation (e.g. comments.reply.user).
```php
Post::query()
    ->present('comments.reply.user', function ($query) {
        $query->where('status', true);
    })
    ->get();
```

This returns all users that have at least one `post → comment → reply` where `status = true`.

You can do the same thing using `ifExists` method. Retrieve all posts that have at least one comment...
```php
Post::query()->ifExists('comments')->get();
```
The `ifExists()` method in Doppar is used as a conditional check to determine whether a related model (e.g., posts) exists in the database for a given parent model (e.g., users). This method is useful for filtering results based on the existence of related data without requiring explicit joins or additional queries

With conditions - find users who have at least one published post
```php
User::query()
    ->ifExists('posts', function ($query) {
        $query->where('status', '=', 'published');
    })
    ->get();
```

The `ifExists()` method works similarly, filtering results based on the existence of related nested records.
```php
User::query()
    ->ifExists('posts.comments.reply', function ($query) {
        $query->where('approved', true);
    })
    ->get();
```

This returns all users that have at least one `post → comment → reply` where `approved = true`.

## Quering Relationship Missing
The `absent()` method is used to fetch records where a particular relationship does not exist. This is useful when you want to retrieve records that are missing related data.

Retrieve all posts that has no comments:
```php
Post::query()->absent('comments')->get();
```

You can also use orAbsent() like this way.
```php
User::query()
    ->whereYear('created_at', 2025)
    ->orAbsent('posts')
    ->get();
```

In the Doppar framework, the `ifNotExists()` method works similarly to the `ifExists()` method but with the inverse logic. Instead of filtering posts who that has at least one comment, it retrieves posts that don't have any related comments. This can be useful when you want to find records without any associated data.

Find posts that don't have any comments:
```php
Post::query()->ifNotExists('comments')->get();
```
`ifNotExists('comments')` Filters the Post models to only those where the comments relationship is empty or doesn't exist.

## whereLinked()
Filters models based on the existence of related records that match a given condition. This method allows you to query models that are linked (via relationships) to other models with specific field values.

Find users who have at least one published post
```php
User::query()
    ->whereLinked('posts', 'status', true)
    ->orderBy('id', 'asc')
    ->get();
```

In the example above, only users who have at least one related Post with `status = true` will be returned.

You can also use nested relationships in your `whereLinked()` method to apply conditions on deeper relationship chains.

Find posts that have at least one comment with an approved reply:
```php
Post::query()
    ->whereLinked('comments.reply', 'status', true)
    ->pluck('id');
```

his will return all `Post` records that have at least one `Comment` whose related `Reply` has `status = true`.

## Relationship Count
The `embedCount()` method is used to count related records without loading all the details. This is useful when you only need to know how many related items exist, for example, how many posts a user has, without fetching all posts from the database. Using `embedCount()` can make your queries faster and more efficient.

For example count posts for each user:
```php
User::query()->embedCount('posts')->get();
```
Or count multiple relations at once:
```php
User::query()
    ->embedCount(['posts', 'comments', 'followers'])
    ->get();
// Access like this way
// $user->posts_count
// $user->comments_count
// $user->followers_count
```

### Conditional Counting
You can apply conditions to only count certain related records. This is useful if you want to count only “active” or “published” items.

Count only active posts for each user
```php
User::query()
    ->embedCount('posts', function ($query) {
        $query->where('status', true);
    })
    ->get();
```

Or count multiple relations with individual conditions. Count published posts and approved comments for each user
```php
User::query()->embedCount([
    'posts' => fn($q) => $q->where('published', true),
    'comments' => fn($q) => $q->where('approved', true),
])->get();
```

### Combining `embed()` and `embedCount()`
Sometimes you need to load full relations for some fields but just count others.

Load full posts and OTPs, but only count posts
```php
User::query()
    ->embed(['posts', 'otp'])
    ->embedCount(['posts'])
    ->paginate(15);
```

### Using Counts in a Foreach Loop
After counting relations, you can access the counts in your loop. This is helpful when showing counts in a UI or report.
```php
$users = User::query()->embedCount(['posts', 'comments'])->get();

foreach ($users as $user) {
    echo "User: {$user->name}\n";
    echo "Posts count: {$user->posts_count}\n";
    echo "Comments count: {$user->comments_count}\n";
}
```
`posts_count` and `comments_count` are automatically added by embedCount() and represent the number of related records.

### Nested Relationship Count
The `embedCount()` method also allows you to include the nested related models count when retrieving records. You can optionally apply conditions to filter which related records are counted.
```php
Post::embedCount('comments.reply')->get();
```
Each Post record will include a field comments data including `reply_count` representing the total number of reply records across all each comments.

Apply a condition to count only related records matching specific criteria.
```php
Post::embedCount('comments.reply', function ($query) {
    $query->where('status', false);
})->get();
```
Each Post record will include a count of only those reply records where `status = false`, under each related comments