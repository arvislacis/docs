# Database: Relationships

## Introduction

Database tables are often related to one another. For example, a blog post may have many comments, or an order could be related to the user who placed it. Winter makes managing and working with these relationships easy and supports several different types of relationships.

## Defining relationships

Winter provides two methods of defining model relationships. Both provide the same level of functionality - which one you use is entirely up to your own preferences.

- Property-based array configuration in the model class (Property style)
- Relation methods defined in the model class, [synonymous with Laravel](https://laravel.com/docs/9.x/eloquent-relationships#defining-relationships) (Method style)

### Property style relationship definition

Model relationships can be defined as properties on your model classes. An example of defining relationships:

```php
class User extends Model
{
    public $hasOne = [
        'profile' => \Acme\Blog\Models\Profile::class,
    ];

    public $hasMany = [
        'posts' => \Acme\Blog\Models\Post::class,
    ];
}
```

Winter CMS automatically converts relations defined in this way into method endpoints on the model. For example, the above `posts` relation can be accessed by calling the `posts()` method on an instance of the `User` model.

```php
$user->posts()
```

Relationships, like the models themselves, also serve as powerful [query builders](query) to allow accessing relationships as functions provides powerful method chaining and querying capabilities. For example:

```php
$user->posts()->where('is_active', true)->get();
```

Accessing a relationship as a property is also possible. Retrieving the relation as a property will give you the "value" of the relation. This means that a single-record relation (ie. `hasOne`) will provide you with the related model directly. In the case of a multiple-record relation (ie. `hasMany`), a collection is usually returned that contains all related records.

```php
$user->profile; // The "Acme\Blog\Models\Profile" record attached to this user
$user->posts; // A collection containing every "Acme\Blog\Models\Post" record attached to this user
```

> **NOTE**: All relationship queries have [in-memory caching enabled](../database/query#in-memory-caching) by default. The `load($relation)` method won't force cache to flush. To reload the memory cache use the `reloadRelations()` or the `reload()` methods on the model object.

### Detailed property style definitions

Each definition can be an array where the key is the relation name and the value is a detail array. The detail array's first value is always the related model class name and all other values are parameters that must have a key name.

```php
public $hasMany = [
    'posts' => ['Acme\Blog\Models\Post', 'delete' => true]
];
```

The following are parameters that can be used with all relations:

Argument | Description
------------- | -------------
`order` | sorting order for multiple records.
`conditions` | filters the relation using a raw where query statement.
`scope` | filters the relation using a supplied scope method.
`push` | if set to false, this relation will not be saved via `push`, default: true.
`delete` | if set to true, the related model will be deleted if the primary model is deleted or relationship is destroyed, default: false.
`detach` | if set to false the related model will not be automatically detached if the primary model is deleted or the relationship is destroyed. Used by `belongsToMany`, `morphToMany` and `morphedByMany` relationships only, default: true.
`count` | if set to true, the result contains a `count` column only, used for counting relations, default: false.

Example filter using **order** and **conditions**:

```php
public $belongsToMany = [
    'categories' => [
        'Acme\Blog\Models\Category',
        'order'      => 'name desc',
        'conditions' => 'is_active = 1'
    ]
];
```

Example filter using **scope**:

```php
class Post extends Model
{
    public $belongsToMany = [
        'categories' => [
            'Acme\Blog\Models\Category',
            'scope' => 'isActive'
        ]
    ];
}

class Category extends Model
{
    public function scopeIsActive($query)
    {
        return $query->where('is_active', true)->orderBy('name', 'desc');
    }
}
```

Example filter using **count**:

```php
public $belongsToMany = [
    'users' => ['Backend\Models\User'],
    'users_count' => ['Backend\Models\User', 'count' => true]
];
```

### Method style relation definition

> **NOTE:** Method style relation definition is available from Winter v1.2.7.

Model relations can also be defined as methods in a model class, similar to the base Laravel framework.

```php
class User extends Model
{
    public function profile(): HasOne
    {
        return $this->hasOne('Acme\Blog\Models\Profile');
    }

    public function posts(): HasMany
    {
        return $this->hasMany('Acme\Blog\Models\Post');
    }
}
```

This way provides a more familiar syntax for Laravel users, and allows for code editors and IDEs to provide syntax support for relations.

One key difference between property style definitions and method style definitions is that a relation method must *explicitly* be defined as a relation method in order for it to be picked up in certain circumstances, for example, when determining all available relations for cascading deletions.

In order to define a relation method, the method must have a single return type that matches one of the `Winter\Storm\Database\Relations` classes:

```php
use Winter\Storm\Database\Relations\HasMany;

public function posts(): HasMany
{
    return $this->hasMany('Acme\Blog\Models\Post');
}
```

Alternatively, you may also use the `Relation` attribute on the method to define it as a relation method, and include the relation type in the attribute:

```php
use Winter\Storm\Database\Attributes\Relation;

#[Relation('hasMany')]
public function posts()
{
    return $this->hasMany('Acme\Blog\Models\Post');
}
```

Since the relation method is already a method, you may call the method to retrieve the relation, similar to the property style relations. This returns a [query builder](query) for the relation and allows powerful chaining capabilities.

```php
$user->posts()->where('is_active', true)->get();
```

As with the property style relations, you can also call the relation as a property of the model, which returns the relation "value". This means that a single-record relation (ie. `hasOne`) will provide you with the related model directly. In the case of a multiple-record relation (ie. `hasMany`), a collection is usually returned that contains all related records.

```php
$user->profile; // The "Acme\Blog\Models\Profile" record attached to this user
$user->posts; // A collection containing every "Acme\Blog\Models\Post" record attached to this user
```

> **WARNING:** If you define a relation in both the relation properties of a class, and define a method relation with the same name, an exception will be thrown. You must only use one style to define a single relation. You can, however, define multiple unique relations in both the property style and the method style.

### Detailed relation methods

Another key difference between property style and method style relation definitions lies in the additional parameters that may be applied to the relation. With the property style, you can configure the relation by providing additional keys and values in the relation configuration array. With the method style, you define these parameters by using chained methods, for example:

```php
public function posts(): HasMany
{
    return $this->hasMany('Acme\Blog\Models\Post')->dependent(true)->pushable();
}
```

The following chained methods are available to define additional parameters about the relation:

Method | Description
------ | -----------
`->dependent(true/false)` | Defines if this relation is "dependent" on the primary model. The related model records will be deleted when the primary model is deleted. This is only available for the following relation types: `attachOne`, `attachMany`, `hasOne`, `hasMany`, `morphOne` and `morphMany`. Default: `false`.
`->detachable(true/false)` | Defines if this relation detaches from the primary model if the primary model is deleted or the relationship is broken. This is only available for the following relation types: `belongsToMany`, `morphToMany` and `morphedByMany`. Default: `true`.
`->pushable(true/false)` | Sets this relation to save when `push()` is run on the primary model. Default: `true`.

You might have noticed that there are no chain methods for handling "constraint" options like the `order`, `conditions` and `scope` options available in the property style relations. That is because relations defined in this format are already [query builders](query) - you can simply add the constraints directly to the relation!

```php
#[Relation('hasMany')]
public function posts()
{
    return $this->hasMany('Acme\Blog\Models\Post')->dependent()->published()->where('free_article', true);
}
```

> **NOTE:** If your relation is constrained in this fashion, the object returned will be a query builder, not the original relation class. You will need to use the `Relation` attribute to mark methods that return query builders as a relation method.

## Relationship types

The following relations types are available:

- [One To One](#one-to-one)
- [One To Many](#one-to-many)
- [Many To Many](#many-to-many)
- [Has Many Through](#has-many-through)
- [Polymorphic relations](#polymorphic-relations)
- [Many To Many Polymorphic relations](#many-to-many-polymorphic)

### One To One

A one-to-one relationship is a very basic relation. For example, a `User` model might be associated with one `Phone`. To define this relationship, we add a `phone` entry to the `$hasOne` property on the `User` model.

```php
<?php namespace Acme\Blog\Models;

use Model;

class User extends Model
{
    // Property style
    public $hasOne = [
        'phone' => 'Acme\Blog\Models\Phone'
    ];

    // Method style
    public function phone(): HasOne
    {
        return $this->hasOne('Acme\Blog\Models\Phone');
    }
}
```

Once the relationship is defined, we may retrieve the related record using the model property of the same name. These properties are dynamic and allow you to access them as if they were regular attributes on the model:

```php
$phone = User::find(1)->phone;
```

The model assumes the foreign key of the relationship based on the model name. In this case, the `Phone` model is automatically assumed to have a `user_id` foreign key. If you wish to override this convention, you may pass the `key` parameter to the definition:

```php
// Property style
public $hasOne = [
    'phone' => ['Acme\Blog\Models\Phone', 'key' => 'my_user_id']
];

// Method style
public function phone(): HasOne
{
    return $this->hasOne('Acme\Blog\Models\Phone', 'my_user_id');
}
```

Additionally, the model assumes that the foreign key should have a value matching the `id` column of the parent. In other words, it will look for the value of the user's `id` column in the `user_id` column of the `Phone` record. If you would like the relationship to use a value other than `id`, you may pass the `otherKey` parameter to the definition:

```php
// Property style
public $hasOne = [
    'phone' => ['Acme\Blog\Models\Phone', 'key' => 'my_user_id', 'otherKey' => 'my_id']
];

// Method style
public function phone(): HasOne
{
    return $this->hasOne('Acme\Blog\Models\Phone', 'my_user_id', 'my_id');
}
```

#### Defining the inverse of a One To One relation

Now that we can access the `Phone` model from our `User`. Let's do the opposite and define a relationship on the `Phone` model that will let us access the `User` that owns the phone. We can define the inverse of a `hasOne` relationship using the `$belongsTo` property:

```php
class Phone extends Model
{
    // Property style
    public $belongsTo = [
        'user' => 'Acme\Blog\Models\User'
    ];

    // Method style
    public function user(): BelongsTo
    {
        return $this->belongsTo('Acme\Blog\Models\User');
    }
}
```

In the example above, the model will try to match the `user_id` from the `Phone` model to an `id` on the `User` model. It determines the default foreign key name by examining the name of the relationship definition and suffixing the name with `_id`. However, if the foreign key on the `Phone` model is not `user_id`, you may pass a custom key name using the `key` parameter on the definition:

```php
// Property style
public $belongsTo = [
    'user' => ['Acme\Blog\Models\User', 'key' => 'my_user_id']
];

// Method style
public function user(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\User', 'my_user_id');
}
```

If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass the `otherKey` parameter to the definition specifying your parent table's custom key:

```php
// Property style
public $belongsTo = [
    'user' => ['Acme\Blog\Models\User', 'key' => 'my_user_id', 'otherKey' => 'my_id']
];

// Method style
public function user(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\User', 'my_user_id', 'my_id');
}
```

#### Default models

The `belongsTo`, `hasOne`, `hasOneThrough`, and `morphOne` relationships allow you to define a default model that will be returned if the given relationship is `null`. This pattern is often referred to as the [Null Object pattern](https://en.wikipedia.org/wiki/Null_Object_pattern) and can help remove conditional checks in your code. In the following example, the `user` relation will return an empty `Acme\Blog\Models\User` model if no `user` is attached to the post:

```php
public $belongsTo = [
    'user' => ['Acme\Blog\Models\User', 'default' => true]
];
```

To populate the default model with attributes, you may pass an array to the `default` parameter:

```php
public $belongsTo = [
    'user' => [
        'Acme\Blog\Models\User',
        'default' => ['name' => 'Guest']
    ]
];
```

If you have defined the relation as a method, you may use the `withDefault()` method to define a default model:

```php
public function user(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\User')->withDefault();
}

// With attributes

public function user(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\User')->withDefault([
        'name' => 'Guest',
    ]);
}
```

### One To Many

A one-to-many relationship is used to define relationships where a single model owns any amount of other models. For example, a blog post may have an infinite number of comments. Like all other relationships, one-to-many relationships are defined adding an entry to the `$hasMany` property on your model:

```php
class Post extends Model
{
    // Property style
    public $hasMany = [
        'comments' => 'Acme\Blog\Models\Comment'
    ];

    // Method style
    public function comments(): HasMany
    {
        return $this->hasMany('Acme\Blog\Models\Comment');
    }
}
```

Remember, the model will automatically determine the proper foreign key column on the `Comment` model. By convention, it will take the "snake case" name of the owning model and suffix it with `_id`. So for this example, we can assume the foreign key on the `Comment` model is `post_id`.

Once the relationship has been defined, we can access the collection of comments by accessing the `comments` property. Remember, since the model provides "dynamic properties", we can access relationships as if they were defined as properties on the model:

```php
$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    //
}
```

Of course, since all relationships also serve as query builders, you can add further constraints to which comments are retrieved by calling the `comments` method and continuing to chain conditions onto the query:

```php
$comments = Post::find(1)->comments()->where('title', 'foo')->first();
```

Like the `hasOne` relation, you may also override the foreign and local keys by passing the `key` and `otherKey` parameters on the definition respectively:

```php
// Property style
public $hasMany = [
    'comments' => ['Acme\Blog\Models\Comment', 'key' => 'my_post_id', 'otherKey' => 'my_id']
];

// Method style
public function comments(): HasMany
{
    return $this->hasMany('Acme\Blog\Models\Comment', 'my_post_id', 'my_id');
}
```

#### Defining the inverse of a One To Many relation

Now that we can access all of a post's comments, let's define a relationship to allow a comment to access its parent post. To define the inverse of a `hasMany` relationship, define the `$belongsTo` property on the child model:

```php
class Comment extends Model
{
    // Property style
    public $belongsTo = [
        'post' => 'Acme\Blog\Models\Post'
    ];

    // Method style
    public function post(): BelongsTo
    {
        return $this->belongsTo('Acme\Blog\Models\Post');
    }
}
```

Once the relationship has been defined, we can retrieve the `Post` model for a `Comment` by accessing the `post` "dynamic property":

```php
$comment = Comment::find(1);

echo $comment->post->title;
```

In the example above, the model will try to match the `post_id` from the `Comment` model to an `id` on the `Post` model. It determines the default foreign key name by examining the name of the relationship  and suffixing it with `_id`. However, if the foreign key on the `Comment` model is not `post_id`, you may pass a custom key name using the `key` parameter:

```php
// Property style
public $belongsTo = [
    'post' => ['Acme\Blog\Models\Post', 'key' => 'my_post_id']
];

// Method style
public function post(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\Post', 'my_post_id');
}
```

If your parent model does not use `id` as its primary key, or you wish to join the child model to a different column, you may pass the `otherKey` parameter to the definition specifying your parent table's custom key:

```php
// Property style
public $belongsTo = [
    'post' => ['Acme\Blog\Models\Post', 'key' => 'my_post_id', 'otherKey' => 'my_id']
];

// Method style
public function post(): BelongsTo
{
    return $this->belongsTo('Acme\Blog\Models\Post', 'my_post_id', 'my_id');
}
```

### Many To Many

Many-to-many relations are slightly more complicated than `hasOne` and `hasMany` relationships. An example of such a relationship is a user with many roles, where the roles are also shared by other users. For example, many users may have the role of "Admin". To define this relationship, three database tables are needed: `users`, `roles`, and `role_user`. The `role_user` table is derived from the alphabetical order of the related model names, and contains the `user_id` and `role_id` columns.

Below is an example that shows the [database table structure](../plugin/updates#migration-and-seed-files) used to create the join table.

```php
Schema::create('role_user', function ($table)
{
    $table->integer('user_id')->unsigned();
    $table->integer('role_id')->unsigned();
    $table->primary(['user_id', 'role_id']);
});
```

Many-to-many relationships are defined adding an entry to the `$belongsToMany` property on your model class. For example, let's define the `roles` method on our `User` model:

```php
class User extends Model
{
    // Property style
    public $belongsToMany = [
        'roles' => 'Acme\Blog\Models\Role'
    ];

    // Method style
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany('Acme\Blog\Models\Role');
    }
}
```

Once the relationship is defined, you may access the user's roles using the `roles` dynamic property:

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    //
}
```

Of course, like all other relationship types, you may call the `roles` method to continue chaining query constraints onto the relationship:

```php
$roles = User::find(1)->roles()->orderBy('name')->get();
```

As mentioned previously, to determine the table name of the relationship's joining table, the model will join the two related model names in alphabetical order. However, you are free to override this convention. You may do so by passing the `table` parameter to the `belongsToMany` definition:

```php
// Property style
public $belongsToMany = [
    'roles' => ['Acme\Blog\Models\Role', 'table' => 'acme_blog_role_user']
];

// Method style
public function roles(): BelongsToMany
{
    return $this->belongsToMany('Acme\Blog\Models\Role', 'acme_blog_role_user');
}
```

In addition to customizing the name of the joining table, you may also customize the column names of the keys on the table by passing additional parameters to the `belongsToMany` definition. The `key` parameter is the foreign key name of the model on which you are defining the relationship, while the `otherKey` parameter is the foreign key name of the model that you are joining to:

```php
// Property style
public $belongsToMany = [
    'roles' => [
        'Acme\Blog\Models\Role',
        'table'    => 'acme_blog_role_user',
        'key'      => 'my_user_id',
        'otherKey' => 'my_role_id'
    ]
];

// Method style
public function roles(): BelongsToMany
{
    return $this->belongsToMany('Acme\Blog\Models\Role', 'acme_blog_role_user', 'my_user_id', 'my_role_id');
}
```

#### Defining the inverse of the relationship

To define the inverse of a many-to-many relationship, you simply place another `$belongsToMany` property on your related model. To continue our user roles example, let's define the `users` relationship on the `Role` model:

```php
class Role extends Model
{
    // Property style
    public $belongsToMany = [
        'users' => 'Acme\Blog\Models\User'
    ];

    // Method style
    public function users(): BelongsToMany
    {
        return $this->belongsToMany('Acme\Blog\Models\User');
    }
}
```

As you can see, the relationship is defined exactly the same as its `User` counterpart, with the exception of simply referencing the `Acme\Blog\Models\User` model. Since we're reusing the `$belongsToMany` property, all of the usual table and key customization options are available when defining the inverse of many-to-many relationships.

#### Retrieving intermediate table columns

As you have already learned, working with many-to-many relations requires the presence of an intermediate join table. Models provide some very helpful ways of interacting with this table. For example, let's assume our `User` object has many `Role` objects that it is related to. After accessing this relationship, we may access the intermediate table using the `pivot` attribute on the models:

```php
$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}
```

Notice that each `Role` model we retrieve is automatically assigned a `pivot` attribute. This attribute contains a model representing the intermediate table, and may be used like any other model.

By default, only the model keys will be present on the `pivot` object. If your pivot table contains extra attributes, you must specify them when defining the relationship:

```php
// Property style
public $belongsToMany = [
    'users' => [
        'Acme\Blog\Models\User',
        'pivot' => ['column1', 'column2']
    ]
];

// Method style
public function users(): BelongsToMany
{
    return $this->belongsToMany('Acme\Blog\Models\User')->withPivot('column1', 'column2');
}
```

If you want your pivot table to have automatically maintained `created_at` and `updated_at` timestamps, use the `timestamps` parameter on the relationship definition:

```php
// Property style
public $belongsToMany = [
    'users' => ['Acme\Blog\Models\User', 'timestamps' => true]
];

// Method style
public function users(): BelongsToMany
{
    return $this->belongsToMany('Acme\Blog\Models\User')->withTimestamps();
}
```

These are the parameters supported for `belongsToMany` relations:

Argument | Description
------------- | -------------
`table` | the name of the join table.
`key` | the key column name of the defining model (inside pivot table). Default value is combined from model name and `_id` suffix, i.e. `user_id`
`parentKey` | the key column name of the defining model (inside defining model table). Default: id
`otherKey` | the key column name of the related model (inside pivot table). Default value is combined from model name and `_id` suffix, i.e. `role_id`
`relatedKey` | the key column name of the related model (inside related model table). Default: id
`pivot` | an array of pivot columns found in the join table, attributes are available via `$model->pivot`.
`pivotModel` | specify a custom model class to return when accessing the pivot relation. Defaults to `Winter\Storm\Database\Pivot`. Note: `pivot` still needs to be defined in order to include the pivot columns in any database queries.
`timestamps` | if true, the join table should contain `created_at` and `updated_at` columns. Default: false

### Has Many Through

The has-many-through relationship provides a convenient short-cut for accessing distant relations via an intermediate relation. For example, a `Country` model might have many `Post` models through an intermediate `User` model. In this example, you could easily gather all blog posts for a given country. Let's look at the tables required to define this relationship:

```
countries
    id - integer
    name - string

users
    id - integer
    country_id - integer
    name - string

posts
    id - integer
    user_id - integer
    title - string
```

Though `posts` does not contain a `country_id` column, the `hasManyThrough` relation provides access to a country's posts via `$country->posts`. To perform this query, the model inspects the `country_id` on the intermediate `users` table. After finding the matching user IDs, they are used to query the `posts` table.

Now that we have examined the table structure for the relationship, let's define it on the `Country` model:

```php
class Country extends Model
{
    // Property style
    public $hasManyThrough = [
        'posts' => [
            'Acme\Blog\Models\Post',
            'through' => 'Acme\Blog\Models\User'
        ],
    ];

    // Method style
    public function posts(): HasManyThrough
    {
        return $this->hasManyThrough('Acme\Blog\Models\Post', 'Acme\Blog\Models\User');
    }
}
```

The first argument passed to the `$hasManyThrough` relation is the name of the final model we wish to access, while the `through` parameter is the name of the intermediate model.

Typical foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the `key`, `otherKey` and `throughKey` parameters to the `$hasManyThrough` definition. The `key` parameter is the name of the foreign key on the intermediate model, the `throughKey` parameter is the name of the foreign key on the final model, while the `otherKey` is the local key.

```php
// Property style
public $hasManyThrough = [
    'posts' => [
        'Acme\Blog\Models\Post',
        'key'        => 'my_country_id',
        'through'    => 'Acme\Blog\Models\User',
        'throughKey' => 'my_user_id',
        'otherKey'   => 'my_id'
    ],
];

// Method style
public function posts(): HasManyThrough
{
    return $this->hasManyThrough('Acme\Blog\Models\Post', 'Acme\Blog\Models\User', 'my_country_id', 'my_user_id', 'my_id');
}
```

### Has One Through

The `hasOneThrough` relationship links models through a single intermediate relation. For example, if each supplier has one user, and each user is associated with one user history record, then the supplier model may access the user's history through the user. Let's look at the database tables necessary to define this relationship:

```
users
    id - integer
    supplier_id - integer

suppliers
    id - integer

history
    id - integer
    user_id - integer
```

Though the `history` table does not contain a `supplier_id` column, the `hasOneThrough` relation can provide access to the user's history to the supplier model. Now that we have examined the table structure for the relationship, let's define it on the `Supplier` model:

```php
class Supplier extends Model
{
    // Property style
    public $hasOneThrough = [
        'userHistory' => [
            'Acme\Supplies\Model\History',
            'through' => 'Acme\Supplies\Model\User'
        ],
    ];

    // Method style
    public function userHistory(): HasOneThrough
    {
        return $this->hasOneThrough('Acme\Supplies\Model\History', 'Acme\Supplies\Model\User');
    }
}
```

The first array parameter passed to the `$hasOneThrough` property is the name of the final model we wish to access, while the `through` key is the name of the intermediate model.

Typical foreign key conventions will be used when performing the relationship's queries. If you would like to customize the keys of the relationship, you may pass them as the `key`, `otherKey`, `throughKey`, and `secondOtherKey` parameters to the `$hasOneThrough` definition.

- `key`: The name of the foreign key on the intermediate model that links to the parent model.
- `through`: The intermediate model class name.
- `throughKey`: The name of the foreign key on the final model that links to the intermediate model.
- `otherKey`: The local key on the parent model.
- `secondOtherKey`: The local key on the intermediate model that should be matched against the final model. This is useful when the join to the final model does not use the intermediate model’s primary key.

```php
// Property style
public $hasOneThrough = [
    'userHistory' => [
        'Acme\Supplies\Model\History',
        'key'        => 'supplier_id',
        'through'    => 'Acme\Supplies\Model\User',
        'throughKey' => 'user_id',
        'otherKey'   => 'id'
    ],
];

// Method style
public function userHistory(): HasOneThrough
{
    return $this->hasOneThrough('Acme\Supplies\Model\History', 'Acme\Supplies\Model\User', 'supplier_id', 'user_id', 'id');
}
```

#### Using `secondOtherKey`

Consider the following table structure:

```
departments
    id - integer
    code - string

employees
    id - integer
    department_code - string

records
    id - integer
    employee_id - integer
```

In this case, each department has many employees identified by a `department_code`, and each employee has one record. To define a `hasOneThrough` relationship from a department to a record through the employee, where the join to `employees` uses `department_code` rather than the default `id`, use the `secondOtherKey`:

```php
public $hasOneThrough = [
    'record' => [
        'App\Models\Record',
        'key' => 'id',
        'through' => 'App\Models\Employee',
        'throughKey' => 'employee_id',
        'otherKey' => 'code',
        'secondOtherKey' => 'department_code',
    ],
];
```

This results in a query that joins `employees` to `records` using `employee_id`, and joins `departments.code` to `employees.department_code` instead of using the default `id`.

```sql
select `records`.*
from `records`
inner join `employees` on `employees`.`id` = `records`.`employee_id`
where `employees`.`department_code` = `departments`.`code`
```

This setup is useful in cases where intermediate relationships are keyed using non-standard or business-specific identifiers instead of primary keys.

### Polymorphic relations

Polymorphic relations allow a model to belong to more than one other model on a single association.

### One To One (Polymorphic)

A one-to-one polymorphic relation is similar to a simple one-to-one relation; however, the target model can belong to more than one type of model on a single association. For example, imagine you want to store photos for your staff members and for your products. Using polymorphic relationships, you can use a single `photos` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

```
staff
    id - integer
    name - string

products
    id - integer
    price - integer

photos
    id - integer
    path - string
    imageable_id - integer
    imageable_type - string
```

Two important columns to note are the `imageable_id` and `imageable_type` columns on the `photos` table. The `imageable_id` column will contain the ID value of the owning staff or product, while the `imageable_type` column will contain the class name of the owning model. The `imageable_type` column is how the ORM determines which "type" of owning model to return when accessing the `imageable` relation.

Next, let's examine the model definitions needed to build this relationship:

```php
class Photo extends Model
{
    // Property style
    public $morphTo = [
        'imageable' => []
    ];

    // Method style
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Staff extends Model
{
    // Property style
    public $morphOne = [
        'photo' => ['Acme\Blog\Models\Photo', 'name' => 'imageable']
    ];

    // Method style
    public function photo(): MorphOne
    {
        return $this->morphOne('Acme\Blog\Models\Photo', 'imageable');
    }
}

class Product extends Model
{
    // Property style
    public $morphOne = [
        'photo' => ['Acme\Blog\Models\Photo', 'name' => 'imageable']
    ];

    // Method style
    public function photo(): MorphOne
    {
        return $this->morphOne('Acme\Blog\Models\Photo', 'imageable');
    }
}
```

#### Retrieving Polymorphic relations

Once your database table and models are defined, you may access the relationships via your models. For example, to access the photo for a staff member, we can simply use the `photo` dynamic property:

```php
$staff = Staff::find(1);

$photo = $staff->photo;
```

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the `morphTo` relationship. In our case, that is the `imageable` definition on the `Photo` model. So, we will access it as a dynamic property:

```php
$photo = Photo::find(1);

$imageable = $photo->imageable;
```

The `imageable` relation on the `Photo` model will return either a `Staff` or `Product` instance, depending on which type of model owns the photo.

### One To Many (Polymorphic)

A one-to-many polymorphic relation is similar to a simple one-to-many relation; however, the target model can belong to more than one type of model on a single association. For example, imagine users of your application can "comment" on both posts and videos. Using polymorphic relationships, you may use a single `comments` table for both of these scenarios. First, let's examine the table structure required to build this relationship:

```
posts
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

Next, let's examine the model definitions needed to build this relationship:

```php
class Comment extends Model
{
    // Property style
    public $morphTo = [
        'commentable' => []
    ];

    // Method style
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    // Property style
    public $morphMany = [
        'comments' => ['Acme\Blog\Models\Comment', 'name' => 'commentable']
    ];

    // Method style
    public function comments(): MorphMany
    {
        return $this->morphMany('Acme\Blog\Models\Comment', 'commentable');
    }
}

class Product extends Model
{
    // Property style
    public $morphMany = [
        'comments' => ['Acme\Blog\Models\Comment', 'name' => 'commentable']
    ];

    // Method style
    public function comments(): MorphMany
    {
        return $this->morphMany('Acme\Blog\Models\Comment', 'commentable');
    }
}
```

#### Retrieving The Relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the comments for a post, we can use the `comments` dynamic property:

```php
$post = Author\Plugin\Models\Post::find(1);

foreach ($post->comments as $comment) {
    //
}
```

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the `morphTo` relationship. In our case, that is the `commentable` definition on the `Comment` model. So, we will access it as a dynamic property:

```php
$comment = Author\Plugin\Models\Comment::find(1);

$commentable = $comment->commentable;
```

The `commentable` relation on the `Comment` model will return either a `Post` or `Video` instance, depending on which type of model owns the comment.

You are also able to update the owner of the related model by setting the attribute with the name of the `morphTo` relationship, in this case `commentable`.

```php
$comment = Author\Plugin\Models\Comment::find(1);
$video = Author\Plugin\Models\Video::find(1);

$comment->commentable = $video;
$comment->save();
```

### Many To Many (Polymorphic)

In addition to "one-to-one" and "one-to-many" relations, you may also define "many-to-many" polymorphic relations. For example, a blog `Post` and `Video` model could share a polymorphic relation to a `Tag` model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. First, let's examine the table structure:

```
posts
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

Next, we're ready to define the relationships on the model. The `Post` and `Video` models will both have a `tags` relation defined in the `$morphToMany` property on the base model class:

```php
class Post extends Model
{
    // Property style
    public $morphToMany = [
        'tags' => ['Acme\Blog\Models\Tag', 'name' => 'taggable']
    ];

    // Method style
    public function tags(): MorphToMany
    {
        return $this->morphToMany('Acme\Blog\Models\Tag', 'taggable');
    }
}
```

#### Defining the inverse of a Many To Many (Polymorphic) relationship

Next, on the `Tag` model, you should define a relation for each of its related models. So, for this example, we will define a `posts` relation and a `videos` relation:

```php
class Tag extends Model
{
    // Property style
    public $morphedByMany = [
        'posts'  => ['Acme\Blog\Models\Post', 'name' => 'taggable'],
        'videos' => ['Acme\Blog\Models\Video', 'name' => 'taggable']
    ];

    // Method style
    public function posts(): MorphedByMany
    {
        return $this->morphedByMany('Acme\Blog\Models\Post', 'taggable');
    }
    public function videos(): MorphedByMany
    {
        return $this->morphedByMany('Acme\Blog\Models\Video', 'taggable');
    }
}
```

#### Retrieving the relationship

Once your database table and models are defined, you may access the relationships via your models. For example, to access all of the tags for a post, you can simply use the `tags` dynamic property:

```php
$post = Post::find(1);

foreach ($post->tags as $tag) {
    //
}
```

You may also retrieve the owner of a polymorphic relation from the polymorphic model by accessing the name of the relationship defined in the `$morphedByMany` property. In our case, that is the `posts` or `videos` methods on the `Tag` model. So, you will access those relations as dynamic properties:

```php
$tag = Tag::find(1);

foreach ($tag->videos as $video) {
    //
}
```

#### Custom Polymorphic types

By default, the fully qualified class name is used to store the related model type. For instance, given the example above where a `Photo` may belong to `Staff` or a `Product`, the default `imageable_type` value is either `Acme\Blog\Models\Staff` or `Acme\Blog\Models\Product` respectively.

Using a custom polymorphic type lets you decouple your database from your application's internal structure. You may define a relationship "morph map" to provide a custom name for each model instead of the class name:

```php
use Winter\Storm\Database\Relations\Relation;

Relation::morphMap([
    'staff' => 'Acme\Blog\Models\Staff',
    'product' => 'Acme\Blog\Models\Product',
]);
```

The most common place to register the `morphMap` in the `boot` method of a [Plugin registration file](../plugin/registration#supported-methods).

## Querying relations

Since all types of Model relationships can be called via functions, you may call those functions to obtain an instance of the relationship without actually executing the relationship queries. In addition, all types of relationships also serve as [query builders](query), allowing you to continue to chain constraints onto the relationship query before finally executing the SQL against your database.

For example, imagine a blog system in which a `User` model has many associated `Post` models:

```php
class User extends Model
{
    public $hasMany = [
        'posts' => ['Acme\Blog\Models\Post']
    ];
}
```

### Access via relationship method

You may query the **posts** relationship and add additional constraints to the relationship using the `posts` method. This gives you the ability to chain any of the [query builder](query) methods on the relationship.

```php
$user = User::find(1);

$posts = $user->posts()->where('is_active', 1)->get();

$post = $user->posts()->first();
```

### Access via dynamic property

If you do not need to add additional constraints to a relationship query, you may simply access the relationship as if it were a property. For example, continuing to use our `User` and `Post` example models, we may access all of a user's posts using the `$user->posts` property instead.

```php
$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}
```

Dynamic properties are "lazy loading", meaning they will only load their relationship data when you actually access them. Because of this, developers often use [eager loading](#eager-loading) to pre-load relationships they know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

### Querying relationship existence

When accessing the records for a model, you may wish to limit your results based on the existence of a relationship. For example, imagine you want to retrieve all blog posts that have at least one comment. To do so, you may pass the name of the relationship to the `has` method:

```php
// Retrieve all posts that have at least one comment...
$posts = Post::has('comments')->get();
```

You may also specify an operator and count to further customize the query:

```php
// Retrieve all posts that have three or more comments...
$posts = Post::has('comments', '>=', 3)->get();
```

> **NOTE**: MySQL strict mode can sometimes complain when using a `GROUP` column without a `GROUP BY` clause (in the above case, `COUNT()`). You can set the `strict` option to `false` in your database's connection configuration (i.e. `database.connections.mysql.strict`) to ignore this warning message.

Nested `has` statements may also be constructed using "dot" notation. For example, you may retrieve all posts that have at least one comment and vote:

```php
// Retrieve all posts that have at least one comment with votes...
$posts = Post::has('comments.votes')->get();
```

If you need even more power, you may use the `whereHas` and `orWhereHas` methods to put "where" conditions on your `has` queries. These methods allow you to add customized constraints to a relationship constraint, such as checking the content of a comment:

```php
// Retrieve all posts with at least one comment containing words like foo%
$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
```

## Eager loading

When accessing relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, models can "eager load" relationships at the time you query the parent model. Eager loading alleviates the N + 1 query problem. To illustrate the N + 1 query problem, consider a `Book` model that is related to `Author`:

```php
class Book extends Model
{
    public $belongsTo = [
        'author' => ['Acme\Blog\Models\Author']
    ];
}
```

Now let's retrieve all books and their authors:

```php
$books = Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
```

This loop will execute 1 query to retrieve all of the books on the table, then another query for each book to retrieve the author. So, if we have 25 books, this loop would run 26 queries: 1 for the original book, and 25 additional queries to retrieve the author of each book.

Thankfully we can use eager loading to reduce this operation to just 2 queries. When querying, you may specify which relationships should be eager loaded using the `with` method:

```php
$books = Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
```

For this operation only two queries will be executed:

```sql
select * from books

select * from authors where id in (1, 2, 3, 4, 5, ...)
```

> **NOTE:** If you are selecting specific columns in your query and want to load relationships as well, you need to make sure that the columns that contain the keying data (i.e. `id`, `foreign_key`, etc) are included in your select statement. Otherwise, Winter cannot connect the relations.

### Eager loading multiple relationships

Sometimes you may need to eager load several different relationships in a single operation. To do so, just pass additional arguments to the `with` method:

```php
$books = Book::with('author', 'publisher')->get();
```

### Nested eager loading

To eager load nested relationships, you may use "dot" syntax. For example, let's eager load all of the book's authors and all of the author's personal contacts in one statement:

```php
$books = Book::with('author.contacts')->get();
```

### Constraining eager loads

Sometimes you may wish to eager load a relationship, but also specify additional query constraints for the eager loading query. Here's an example:

```php
$users = User::with([
    'posts' => function ($query) {
        $query->where('title', 'like', '%first%');
    }
])->get();
```

In this example, the model will only eager load posts if the post's `title` column contains the word `first`. Of course, you may call other [query builder](query) methods to further customize the eager loading operation:

```php
$users = User::with([
    'posts' => function ($query) {
        $query->orderBy('created_at', 'desc');
    }
])->get();
```

### Lazy eager loading

Sometimes you may need to eager load a relationship after the parent model has already been retrieved. For example, this may be useful if you need to dynamically decide whether to load related models:

```php
$books = Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

If you need to set additional query constraints on the eager loading query, you may pass a `Closure` to the `load` method:

```php
$books->load([
    'author' => function ($query) {
        $query->orderBy('published_date', 'asc');
    }
]);
```

## Inserting related models

Just like you would [query a relationship](#querying-relations), Winter supports defining a relationship using a method or dynamic property approach. For example, perhaps you need to insert a new `Comment` for a `Post` model. Instead of manually setting the `post_id` attribute on the `Comment`, you may insert the `Comment` directly from the relationship.

### Insert via relationship method

Winter provides convenient methods for adding new models to relationships. Primarily models can be added to a relationship or removed from a relationship. In each case the relationship is associated or disassociated respectively.

#### Add method

Use the `add` method to associate a new relationship.

```php
$comment = new Comment(['message' => 'A new comment.']);

$post = Post::find(1);

$comment = $post->comments()->add($comment);
```

Notice that we did not access the `comments` relationship as a dynamic property. Instead, we called the `comments` method to obtain an instance of the relationship. The `add` method will automatically add the appropriate `post_id` value to the new `Comment` model.

If you need to save multiple related models, you may use the `addMany` method:

```php
$post = Post::find(1);

$post->comments()->addMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another comment.']),
]);
```

#### Remove method

Comparatively, the `remove` method can be used to disassociate a relationship, making it an orphaned record.

```php
$post->comments()->remove($comment);
```

In the case of many-to-many relations, the record is removed from the relationship's collection instead.

```php
$post->categories()->remove($category);
```

In the case of a "belongs to" relationship, you may use the `dissociate` method, which doesn't require the related model passed to it.

```php
$post->author()->dissociate();
```

#### Adding with pivot data

When working with a many-to-many relationship, the `add` method accepts an array of additional intermediate "pivot" table attributes as its second argument as an array.

```php
$user = User::find(1);

$pivotData = ['expires' => $expires];

$user->roles()->add($role, $pivotData);
```

The second argument of the `add` method can also specify the session key used by [deferred binding](#deferred-binding) when passed as a string. In these cases the pivot data can be provided as the third argument instead.

```php
$user->roles()->add($role, $sessionKey, $pivotData);
```

#### Create method

While `add` and `addMany` accept a full model instance, you may also use the `create` method, that accepts a PHP array of attributes, creates a model, and inserts it into the database.

```php
$post = Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
```

Before using the `create` method, be sure to review the documentation on attribute [mass assignment](model#mass-assignment) as the attributes in the PHP array are restricted by the model's "fillable" definition.

### Insert via dynamic property

Relationships can be set directly via their properties in the same way you would access them. Setting a relationship using this approach will overwrite any relationship that existed previously. The model should be saved afterwards like you would with any attribute.

```php
$post->author = $author;

$post->comments = [$comment1, $comment2];

$post->save();
```

Alternatively you may set the relationship using the primary key, this is useful when working with HTML forms.

```php
// Assign to author with ID of 3
$post->author = 3;

// Assign comments with IDs of 1, 2 and 3
$post->comments = [1, 2, 3];

$post->save();
```

Relationships can be disassociated by assigning the NULL value to the property.

```php
$post->author = null;

$post->comments = null;

$post->save();
```

Similar to [deferred binding](#deferred-binding), relationships defined on non-existent models are deferred in memory until they are saved. In this example the post does not exist yet, so the `post_id` attribute cannot be set on the comment via `$post->comments`. Therefore the association is deferred until the post is created by calling the `save` method.

```php
$comment = Comment::find(1);

$post = new Post;

$post->comments = [$comment];

$post->save();
```

### Many To Many relations

#### Attaching / Detaching

When working with many-to-many relationships, Models provide a few additional helper methods to make working with related models more convenient. For example, let's imagine a user can have many roles and a role can have many users. To attach a role to a user by inserting a record in the intermediate table that joins the models, use the `attach` method:

```php
$user = User::find(1);

$user->roles()->attach($roleId);
```

When attaching a relationship to a model, you may also pass an array of additional data to be inserted into the intermediate table:

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

Of course, sometimes it may be necessary to remove a role from a user. To remove a many-to-many relationship record, use the `detach` method. The `detach` method will remove the appropriate record out of the intermediate table; however, both models will remain in the database:

```php
// Detach a single role from the user...
$user->roles()->detach($roleId);

// Detach all roles from the user...
$user->roles()->detach();
```

For convenience, `attach` and `detach` also accept arrays of IDs as input:

```php
$user = User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([1 => ['expires' => $expires], 2, 3]);
```

#### Syncing For convenience

You may also use the `sync` method to construct many-to-many associations. The `sync` method accepts an array of IDs to place on the intermediate table. Any IDs that are not in the given array will be removed from the intermediate table. So, after this operation is complete, only the IDs in the array will exist in the intermediate table:

```php
$user->roles()->sync([1, 2, 3]);
```

You may also pass additional intermediate table values with the IDs:

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

### Touching parent timestamps

When a model `belongsTo` or `belongsToMany` another model, such as a `Comment` which belongs to a `Post`, it is sometimes helpful to update the parent's timestamp when the child model is updated. For example, when a `Comment` model is updated, you may want to automatically "touch" the `updated_at` timestamp of the owning `Post`. Just add a `touches` property containing the names of the relationships to the child model:

```php
class Comment extends Model
{
    /**
     * All of the relationships to be touched.
     */
    protected $touches = ['post'];

    /**
     * Relations
     */
    public $belongsTo = [
        'post' => ['Acme\Blog\Models\Post']
    ];
}
```

Now, when you update a `Comment`, the owning `Post` will have its `updated_at` column updated as well:

```php
$comment = Comment::find(1);

$comment->text = 'Edit to this comment!';

$comment->save();
```

## Dynamically defining a relation

Using Winter's powerful [extension capabilities](../services/behaviors), you can dynamically add additional relations to models at run-time, in both the property style and the method style, allowing you to extend the functionality of models in other plugins, or in the core of Winter CMS.

Within a [plugin boot method](../plugin/registration#registration-file), you can extend a given model to add additional relations like the following:

```php
<?php

namespace Acme\Blog;

use Winter\Storm\Database\Relations\HasMany;

class Plugin extends \System\Classes\PluginBase
{
    // ...
    public function boot()
    {
        \Acme\Blog\Post::extend(function ($model) {
            // Property-style
            $model->addHasManyRelation(\Acme\Blog\Comment::class, [
                'delete' => true,
            ]);

            // Method-style
            $model->addDynamicMethod('comments', function (): HasMany {
                return $model->hasMany(\Acme\Blog\Comment::class)->dependent();
            });
        });
    }
}
```

When using the method style of defining a dynamic relation, you must ensure that the callback function has a return type of one of the applicable relation classes in order for it to be identified as a relation method.

Winter provides helper methods to dynamically add relations. The following methods can be used to create relations:

- `addHasOneRelation()`
- `addHasManyRelation()`
- `addBelongsToRelation()`
- `addBelongsToManyRelation()`
- `addHasOneThroughRelation()`
- `addHasManyThroughRelation()`
- `addAttachOneRelation()`
- `addAttachManyRelation()`
- `addMorphOneRelation()`
- `addMorphManyRelation()`
- `addMorphToRelation()`
- `addMorphToManyRelation()`
- `addMorphedByManyRelation()`

In all methods above, the first parameter defines the related class, as a class string, and the second parameter provides the relation config as an array. Please note that using these methods results in the relation being defined in the relation properties.

## Deferred binding

Deferred bindings allows you to postpone model relationships binding until the master record commits the changes. This is particularly useful if you need to prepare some models (such as file uploads) and associate them to another model that doesn't exist yet.

You can defer any number of **slave** models against a **master** model using a **session key**. When the master record is saved along with the session key, the relationships to slave records are updated automatically for you. Deferred bindings are supported in the backend [Form behavior](../backend/forms) automatically, but you may want to use this feature in other places.

### Generating a session key

The session key is required for deferred bindings. You can think of a session key as of a transaction identifier. The same session key should be used for binding/unbinding relationships and saving the master model. You can generate the session key with PHP `uniqid()` function. Note that both [backend forms](../backend/forms) and the [frontend `form()` function](/docs/v1.2/markup/functions/form) generates a hidden field containing the session key automatically.

```php
$sessionKey = uniqid('session_key', true);
```

### Defer a relation binding

The comment in the next example will not be added to the post unless the post is saved.

```php
$comment = new Comment;
$comment->content = "Hello world!";
$comment->save();

$post = new Post;
$post->comments()->add($comment, $sessionKey);
```

> **NOTE**: the `$post` object has not been saved but the relationship will be created if the saving happens.

### Defer a relation unbinding

The comment in the next example will not be deleted unless the post is saved.

```php
$comment = Comment::find(1);
$post = Post::find(1);
$post->comments()->remove($comment, $sessionKey);
```

### List all bindings

Use the `withDeferred` method of a relation to load all records, including deferred. The results will include existing relations as well.

```php
$post->comments()->withDeferred($sessionKey)->get();
```

### Cancel all bindings

It's a good idea to cancel deferred binding and delete the slave objects rather than leaving them as orphans.

```php
$post->cancelDeferred($sessionKey);
```

### Commit all bindings

You can commit (bind or unbind) all deferred bindings when you save the master model by providing the session key with the second argument of the `save` method.

```php
$post = new Post;
$post->title = "First blog post";
$post->save(null, $sessionKey);
```

The same approach works with the model's `create` method:

```php
$post = Post::create(['title' => 'First blog post'], $sessionKey);
```

### Lazily commit bindings

If you are unable to supply the `$sessionKey` when saving, you can commit the bindings at any time using the the next code:

```php
$post->commitDeferred($sessionKey);
```

### Clean up orphaned bindings

Destroys all bindings that have not been committed and are older than 1 day:

```php
Winter\Storm\Database\Models\DeferredBinding::cleanUp(1);
```

> **NOTE:** Winter automatically destroys deferred bindings that are older than 5 days. It happens when a backend user logs into the system.

### Disable Deferred Binding

Sometimes you might need to disable deferred binding entirely for a given model, for instance if you are loading it from a separate database connection. In order to do that, you need to make sure that the model's `sessionKey` property is `null` before the pre and post deferred binding hooks in the internal save method are run. To do that, you can bind to the model's `model.saveInternal` event:

```php
public function __construct(array $attributes = [])
{
    $result = parent::__construct($attributes);
    $this->bindEvent('model.saveInternal', function () {
        $this->sessionKey = null;
    });
    return $result;
}
```

> **NOTE:** This will disable deferred binding entirely for any model's you apply this override to.
