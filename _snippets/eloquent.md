---
layout: post
id: eloquent
title: Eloquent
layout: page
description: List of eloquent methods
---

## Retrieving models

### find()

``` php
// Get a single user
$user = User::find(1);
```
``` php
// Get an instance of User model with just the fields name and surname.
$user = User::find(1, ['name', 'surname']);

/*
=> App\User {
     name: "Michael",
     surname: "Red",
   }
*/
```

### findOrFail()

``` php
// Get a single user, and throw a ModelNotFoundException exception if a model is not found.
$user = User::findOrFail(1);
```

### get()

``` php
// Get the collection item with the key = 'name'
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
$value = $collection->get('name');
// taylor
```

``` php
// Default value

// Get the collection item with the key = 'name', 
// and return 34 as default value if the key don't exist.
$collection = collect(['name' => 'taylor', 'framework' => 'laravel']);
$value = $collection->get('age', 34);
// 34
```


NOTICE:   
The get() Collection method is different from the get() Eloquent method.

## Saving / Updating models

### create()
Create a new model and add it to the database.
- It accept an array of attributes as input
- In addition to the save and saveMany methods, you may also use the create method, to add models to the DB.
``` php
// Create a new comment
$post = App\Post::find(1);
$comment = $post–>comments()–>create([
    'message' => 'A new comment.',
]);
```

### save()
Store a new model to the DB.  
The save method may also be used to update models that already exist in the database.
``` php
// Store a new post
$post = new Post;
$post->title = 'New post title';
$post–>save();

/* 
 * Store a new comment, and relate it to the post 
 * setting also the comment's post_id.
*/
$comment = new Comment(['message' => 'A new comment.']);
$post = Post::find(1);
$post->comments()->save($comment);

// Update a post
$post = Post::find(1);
$post->title = 'Another post title';
$post–>save();
```

### What is the difference between save() & create() methods?
- Save: accepts a full Eloquent model instance.
- Create: accepts a plain PHP array.
``` php
// Two new posts: one using save() and one using create()
$post = new Post;
$post->title = 'New post title';
$post–>save();

$post–>create(['title' => 'A new post.']);
```


### saveMany()
Save multiple related models.
``` php
// Save two comments related to the same post.
$post = Post::find(1);

$post–>comments()–>saveMany([
    new Comment(['message' => 'A new comment.']),
    new Comment(['message' => 'Another comment.']),
]);
```

### update()
Update models that already exist in the database.
``` php
// Update a post (same result that same)
$post = Post::find(1);
$post->title = 'Another post title';
$post–>update();

// Mass update
// Set all the flyights to San Diego as delayed 
Flight::where('active', 1)
      ->where('destination', 'San Diego')
      ->update(['delayed' => 1]);
```



### Touches property
Updates the parent's timestamp when every time the child model is updated.
The parameter specified is an array with one or more names of the relationships to the child model.

Relationship: Post > hasMany > Comment
``` php
// Defined in the Comment model
// update the updated_at property of post when a comment is updated.
protected $touches = ['post'];
```

## Eager loading

[What is the purpose of Eager Loading?](../../guides/beginner/laravel_purpose_of_eager_loading/#what-is-the-purpose-of-eager-loading)

### width() - Eager loading

As parameter, with take the name of the relationship.

Relationship: User > hasOne > Post
``` php
// Relationship defined in the user model
public function profile()
{
    return $this->hasOne(UserProfile::class);
}

// Collection of all users with their profiles
$users = User::with('profile')->get();
```

**Eager Load multiple relationships**
``` php
// Collection of all users with their profiles and their posts
$users = User::with('profile', 'posts')->get();
```

**Nested Eager Loading**
``` php
// Collection of all users and the comments they got to their posts.
$users = User::with('posts.comments')->get();
```


### load() - Lazy Eager Loading

Load a relationship after the parent model has already been retrieved.

[What is the purpose of Lazy Eager Loading?](../../guides/beginner/laravel_purpose_of_eager_loading/#what-is-the-purpose-of-lazy-eager-loading)

``` php

// Collection with all books 
$books = App\Book::all();

// Add to the each book of the collection it's authors and publishers
if ($someCondition) {
    $books–>load('author', 'publisher');
}
```
## Querying Relationship Existence

### has()

Filter the selecting model based on a relationship.
It acts very similarly to a normal WHERE condition.  

When you need to get only the models that have at least one related model in this relation.

Relationship: Post > hasMany > Comment
``` php
// Collection of all posts that have at least one comment
$posts = Post::has('comments')->get();

// Collection of all posts that have three or more comments
$posts = Post::has('comments', '>=', 3)->get();

// Retrieve posts that have at least one comment with images
$posts = Post::has('comments.images')->get();
```

### orHas()

Relationship: Post > hasMany > Like  
Relationship: Post > hasMany > Comment

``` php
// Collection of all the posts that have at least one comment or one like
$posts = Post::has('comments')->orHas('likes')->get();
```

### whereHas()

Similar to **has()**, with the advantage that I can specify filtering conditions with **whereHas()**.  
Whereas allow to specify a where clause on the table that is in the relation, specified with **with()**.

``` php 
/* 
 *  Collection of all the users with their profiles.
 *  Get just the one that in their profile data 
 *  have surname = 'Schulist'. 
 *  Load also their whole profile. (with)
 **/
$user = User::with('profile')
             ->whereHas('profile', function($q){
                  $q->where('surname', 'Schulist');
             })->get();
```


``` php
/* 
 *  Collection of all the users that have published 
 *  a post in the last 7 week.
 **/
$users = User::whereHas('posts', function($q){
    $q->where('created_at', '>=',Carbon\Carbon::now()->subDays(7));
})->get();
```


Note also that:
- **with('relation')** will include the related table's data in the returned collection.
- **has('relation')** and **whereHas('relation')** will not include the related table's data, they just filter the collection of the parent model based on the related model.
- So you may need to call both with('relation') as well as has() or whereHas()


### orWhereHas()

Similar to **orHas()**, with the advantage that I can specify filtering conditions with **orWhereHas()**.

**Relationship:** User > hasMany > Post  
**Relationship:** Post > hasMany > Comment  
**Relationship:** User > hasManyThrough(Post) > Comment  

``` php
// Collection of all the users that made posts OR comments last 7 days.
$users = User::query()
    ->whereHas('posts', function ($q) {
        $q->where('created_at', '>=', Carbon\Carbon::now()->subDays(7));
    })
    ->orWhereHas('comments', function ($q) {
        $q->where('created_at', '>=', Carbon\Carbon::now()->subDays(7));
    })
    ->get();
```

## Affecting scopes

### withoutGlobalScopes()


[What are Anonymous Global Scopes?](https://laravel.com/docs/8.x/eloquent#global-scopes)


``` php
// Collection of all the users, without being affected by the global scopes.
// if we have a global scope that hides the soft deleted users we will get also them.
User::withoutGlobalScopes()->get();
```


You can remove just some global scopes, specifying them as parameters.
``` php
// Collection of all the users, without deleted and disabled users.
User::withoutGlobalScopes([
    DeletedUsers::class, 
    DisabledUsers::class,
])->get();
```

---

## Methods to update one-to-many relationships

### associate()

Updates a belongsTo (one to many) relationship.  
It sets the foreign key on the child model.
``` php
// Associate an account to the user
// If we are following the naming standard, set the value of account_id to 10.
$account = App\Account::find(10);
$user–>account()–>associate($account);
$user–>save();
```

### dissociate()

Removes a belongsTo (one to many) relationship.  
It set the relationship's foreign key to null.
``` php
// Remove the account from the user 
// If we are following the naming standard, set the value of account_id to null.
$user–>account()–>dissociate();
$user–>save();
```


## Methods to update many-to-many relationships

### attach()

Adds one or more records in the intermediate (pivot) table of a BelongsToMany (many to many) relationship.

**Relationship:** User > belongsToMany > Role  
**Relationship:** Role > belongsToMany > User  
``` php
// Attach a role to a user
$user = User::find(1);
$user–>roles()–>attach($roleId);
```

``` php
// Attach mutiple roles to a user
$user = User::find(1);
$user->roles()->attach([1, 2, 3]);
```

You may also pass additional intermediate table values with the IDs:
``` php
/* 
 *  Attach a role to a user
 *  In the user_roles pivot table set the attribute expires to true.
 **/
$user->roles()->attach($roleId, ['expires' => true]);
```

### sync()

Works the same as attach(), and also removes from the intermediate table the relations not specified in parameter.

``` php
// Remove all the roles previously assigned and attach the one specified in the array
$user–>roles()–>sync([1, 2, 3]);
```

You may also pass additional intermediate table values with the IDs:
``` php
/* 
 *  Remove all the roles previously assigned and attach the three specified in the array
 *  In the user_roles pivot table set the attribute expires to true.
 **/
$user–>roles()–>sync([1 => ['expires' => true], 2, 3]);
```

### syncWithoutDetaching()

``` php
// Sync without detaching existing IDs
$user–>roles()–>syncWithoutDetaching([1, 2, 3]);
```
The result on the pivot table is the same.  
What does it change from **attach()** to **syncWithoutDetaching()** since it seems they do something similar?

What changes is the return value.   
attach() returns null in any case.    
syncWithoutDetaching() returns an array with the ids of the field attached, detached and updated.
```
[ "attached" => [], "detached" => [], "updated" => [], ]
```  
Detached will always be empty in this case. 


### updateExistingPivot()
Updates an extra attribute on a pivot table.
This method accepts the pivot record foreign key and an array of attributes to update.

**Relationship:** User > belongsToMany > Message  
**Relationship:** Message > belongsToMany > User

``` php
/* 
 *  We want to record the status of each message if it has been delivered to the user or not.
 *  So we had defined a 'status' attribute in the pivot table  
 **/
 
$user = User::find(1);
$messageId = 2;
// Update the pivot table row with user_id=1 and message_id=2 setting the status attribute to 'sent'.
$user->messages()->updateExistingPivot($messageId, ['status' => 'sent']);
```  


### detach()

Removes one or more records in the pivot table of a BelongsToMany (many to many) relationship.   
However, both models will remain in the database.

``` php
// Detach a single role from the user
$user–>roles()–>detach($roleId);

// Detach all roles from the user
$user–>roles()–>detach();
```

### toggle()

Toggles the attachment status of the given IDs for a many–to–many relationship.
- If the given ID is currently attached, it will be detached.
- Likewise, if it is currently detached, it will be attached.
``` php
/* 
 *  Add or remove from the pivot table post_tags the keys specified,  
 *  depending if they were already present or not.
 **/

$post = Post::find(1);
$post->tags()->toggle([4, 7, 2]);
```


-----

## Various 

### withCount()


Count the number of results from a relationship without actually loading them.
It places a {relation}_count column on your resulting models.

``` php
// Collection of all the posts, adding a column tags_count to each model.
$posts = Post::withCount('tags')->get();

// Print the number of tags of each post
foreach ($posts as $post) {
    echo $post->title.": ".$post->tags_count." tags";
}
```
