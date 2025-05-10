  - [One To One (linkOne)](#one-to-one)
  - [Reverse One To One (bindTo)](#reverse-one-to-one)
  - [One To Many (linkMany)](#one-to-many)
  - [Many To Many (bindToMany)](#many-to-many)
  - [Nested Relationship](#nested-relationship)
  - [Querying Relationship Existence](#relationship-existance)
  - [Querying Relationship Missing](#quering-missing-relationships)

<a name="getting-started"></a>

## Eloquent: Relationships
### Introduction
The Doppar relationship is a term that refers to a specialized type of relationship in data modeling, often used in the context of web development.

In many database systems, tables are often related to one another. For example, a blog post may have many comments or an order could be related to the user who placed it. Eloquent makes managing and working with these relationships easy, and supports a variety of common relationships.

Let’s dive into the specifics of Doppar relationships, their benefits, and how they compare to more conventional relationship types like one-to-one, one-to-many, and many-to-many.

  - [One To One (linkOne)](#one-to-one)
  - [Reverse One To One (bindTo)](#reverse-one-to-one)
  - [One To Many (linkMany)](#one-to-many)
  - [Many To Many (bindToMany)](#many-to-many)

### Defining Relationships
Eloquent relationships are defined as methods on your Eloquent model classes. Since relationships also serve as powerful query builders, defining relationships as methods provides powerful method chaining and querying capabilities. For example, we may chain additional query constraints on this posts relationship:
```php
$user = User::find(1);
$user->posts()->where('status', '=', true)->get();
```

But, before diving too deep into using relationships, let's learn how to define each type of relationship supported by Eloquent.

<a name="one-to-one"></a>

### One to One / Link One
A one-to-one relationship is one of the simplest types of database associations. For instance, a `User` model may be linked to a single `Otp` model. To define this relationship, you can add an otp method to the User model. This method should invoke the `linkOne` method and return its result. The `linkOne` method is provided by the `Phaseolies\Database\Eloquent\Model` base class, making it easily accessible within your model.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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

<a name="reverse-one-to-one"></a>

### One To One Inverse / Bind To
So, we can access the Otp model from our User model. Next, let's define a relationship on the Otp model that will let us access the user that owns the otp. We can define the inverse of a `linkOne` relationship using the `bindTo` method:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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
    ->where('status', '=', 1)
    ->first();
```

<a name="one-to-many"></a>

### One to Many / Link Many
A one-to-many relationship is ideal for scenarios where a single model serves as the parent to multiple related models. For example, a single blog post might have countless comments attached to it. To define this type of relationship in your model, simply create a method that represents the connection. As with all relationships in Eloquent, this method encapsulates the logic and structure of the association, making your code expressive and intuitive.

```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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

Once the relationship method has been defined, we can access the collection of related comments by accessing the comments property. Remember, since Eloquent provides "dynamic relationship properties", we can access relationship methods as if they were defined as properties on the model:

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
Post::query()->embed('comments')->get();
```
This approach ensures that all related articles for each User are loaded in a single query, By retrieving the posts and their associated comments in one go, you significantly improve performance, especially when dealing with large datasets or nested relationships.


### One to Many Inverse / Bind To
Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `LinkMany` relationship, define a relationship method on the child model which calls the belongsTo method:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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

return $comment->post?->title ?? 'default';
```
In the example above, Eloquent will attempt to find a Post model that has an id which matches the post_id column on the Comment model.

### Many to Many Relationships /  Bind To Many
Many-to-many relationships are a bit more involved than linkOne or linkMany relationships. A common example is the relationship between posts and tags. A single post can have multiple tags, and each tag can be associated with multiple posts. For instance, a blog post might be tagged with "Doppar" and "PHP", and those same tags might be used on other posts as well. So, a post has many tags, and a tag belongs to many posts.

#### Table Structure
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
<a name="many-to-many"></a>

### Define Many to Many Relationships
Doppar ORM supports many-to-many relationships through an intermediate pivot table. Here's how to implement and work with them. Assume we have a Post model and a Tag model, where a many-to-many relationship exists between them. This means one post can have multiple tags, and one tag can be associated with multiple posts.
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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

### Retrieving Pivot Table Columns
As you have already learned, working with many-to-many relations requires the presence of an pivot table. After accessing this relationship, we may access the intermediate table using the pivot attribute on the models as follows
```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    $tag->pivot->post_id;
}
```

Since all relationships also serve as query builders, you may add further constraints to the relationship query by calling the tags method and continuing to chain conditions onto the query as like this:
```php
Post::find(1)->tags()->orderBy('name')->get();
```

### Defining the Inverse of the Relationship
To define the "inverse" of a many-to-many relationship, you should define a method on the related model which also returns the result of the `bindToMany` method. To complete our post / tag example, let's define the users method on the Tag model:
```php
<?php

namespace App\Models;

use Phaseolies\Database\Eloquent\Model;

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

### Many to Many Relationships
#### link() with bindToMany
In Doppar, the link() method is used to associate records in a many-to-many relationship. This method adds entries to the pivot table, establishing a connection between related models
```php
// Link tags to a post
$post = Post::find(1);
$post->tags()->link([1, 2, 3]); // Link tags with IDs 1, 2, and 3
```

#### unlink() with bindToMany
In Doppar, the unlink() method is used to unlink specific records from a many-to-many relationship. It removes the association between the current model (e.g., Post) and the related model (e.g., Tag) by removing the corresponding entries from the pivot table.
```php
$post = Post::find(1);
$post->tags()->unlink([1, 2, 3]); // Unlink tags with IDs 1, 2, and 3
```

If you simply call unlink(), it will delete all the tags
```php
$post = Post::find(1);
$post->tags()->unlink(); // unlink all tags
```

#### relate() with bindToMany
In Doppar, the relate() method is used to sync the relationships between models in a many-to-many relationship. The relate() method will attach the provided IDs and can optionally detach existing relationships, depending on the second argument passed.
```php
$post = Post::find(1);
$post->tags()->relate([1, 2, 3]);
$changes = $post->tags()->relate([1, 2, 4]); // 3 will be removed
$post->tags()->relate([1, 2, 3], false); // link tithout unlinking
```

#### Syncing with Pivot Data Using
In Doppar, the relate() method not only allows you to sync records in a many-to-many relationship, but also provides the ability to attach additional data to the pivot table. This is useful when you need to store extra attributes (such as timestamps or other metadata) along with the relationship between two models.
```php
$post = Post::find(1);
$post->tags()->relate([
    1 => ['created_at' => now()],
    2 => ['featured' => true],
    3 => ['meta' => ['color' => 'blue']]
]);
```
<a name="nested-relationship"></a>

### Nested Relationship
In the Doppar framework, you can effortlessly retrieve deeply nested relationships using the powerful embed method. This allows you to preload related models at multiple levels, preventing the "N + 1" query issue and ensuring optimized performance across complex data structures.

For example, if you want to fetch all posts along with their comments, the replies to those comments, and the users who wrote the replies, you can use the following nested eager loading query:
```php
Post::query()->embed('comments.reply.user')->get();
```

Explanation:
- **comments:** Loads all comments related to each post.
- **reply:** Loads replies for each comment (assuming a reply is a nested relationship under a comment).
- **user:** Loads the user who authored each reply.

#### Specific Column Selection
The embed() method in Doppar ORM allows you to eager load related models and even specify which columns to load for each relationship. This helps optimize queries by only selecting the necessary data.
```php
User::query()
    ->embed([
        'comments.reply',
        'posts' => function ($query) {
            $query->select(['id', 'title', 'user_id']);
        }
    ])
    ->get();
```

#### Fetching Multiple Relationships
This query retrieves users along with their related articles and address using the embed method. By embedding multiple relationships, it ensures that all necessary data is fetched in a single query, improving efficiency and reducing additional database calls.
```php
User::query()
  ->embed(['posts','address'])
  ->get();
```

<a name="relationship-existance"></a>

### Querying Relationship Existence
When fetching model records, there are times you might want to filter results based on whether a certain relationship exists. For example, suppose you need to retrieve only the blog posts that have at least one comment attached. In such cases, you can use the present method, passing in the name of the relationship to constrain the query to only those records that have related entries. This approach ensures your results are contextually relevant and tied to meaningful relationships.

In Doppar ORM, the present() method can be used to load relationships that are present (i.e., do not exist) in the model, or when you want to ensure related data is included, even if it is not empty.
```php
use App\Models\Post;

// Retrieve all posts that have at least one comment...
$posts = Post::query()->present('comments')->get();
```

### present() with callback
The present() method can be used to load a relationship with custom query conditions. You can define specific conditions inside the closure passed to present() to filter the related data.
```php
return Post::query()
    ->present('comments', function ($query) {
        $query->where('comment', '=', 'Mrs.')
            ->where('created_at', '=', NULL);
    })
    ->get();
```

You can do the same thing using `ifExists` method
```php
use App\Models\Post;

// Retrieve all posts that have at least one comment...
Post::query()->ifExists('comments')->get();
```
The ifExists() method in Doppar is used as a conditional check to determine whether a related model (e.g., posts) exists in the database for a given parent model (e.g., users). This method is useful for filtering results based on the existence of related data without requiring explicit joins or additional queries

With conditions - find users who have at least one published post
```php
User::query()
    ->ifExists('posts', function ($query) {
        $query->where('status', '=', 'published');
    })
    ->get();
```

<a name="quering-missing-relationships"></a>

### Querying with Missing Relationships
The absent() method is used to fetch records where a particular relationship does not exist. This is useful when you want to retrieve records that are missing related data.
```php
// Retrieve all posts that has no comments
Post::query()->absent('comments')->get();
```

In the Doppar framework, the `ifNotExists()` method works similarly to the `ifExists()` method but with the inverse logic. Instead of filtering posts who that has at least one comment, it retrieves posts that don't have any related comments. This can be useful when you want to find records without any associated data.
```php
// Find posts that don't have any comments
Post::query()->ifNotExists('comments')->get();
```
`ifNotExists('comments')` Filters the Post models to only those where the comments relationship is empty or doesn't exist.