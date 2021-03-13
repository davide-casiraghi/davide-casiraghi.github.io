---
last_modified_on: "2021-01-27"
title: Laravel - Eloquent Eager Loading
layout: page
description: Explain the purpose of Eager Loading and the N+1 query problem
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Eager Loading

Eager Loading means to preload the specified model that is related with the one we are loading from the db with an eloquent query.

In this example we load from the db a Collection with all the users, and their profile data using just one query.
``` php
// Relationship defined in the User model
public function profile()
{
    return $this->hasOne(UserProfile::class);
}

// Load all users with their profile
$users = User::with('profile')->get();
```

## What is the purpose of Eager Loading?
When accessing Eloquent relationships as properties, the relationship data is lazy loaded.

This means the relationship data is not actually loaded until you first access the property.   
However, Eloquent can "eager load" relationships at the time you query the parent model.  
Eager loading alleviates the N + 1 query problem.

## What is the N+1 query problem ?
The N+1 query problem happens when the data access framework executed N additional SQL statements to fetch the same data that could have been retrieved when executing the primary SQL query.  
The larger the value of N, the more queries will be executed, the larger the performance impact.

If you select the post_comments using the following query:   
And, later in a blade file, you decide to fetch the associated post title for each post_comment, you are going to trigger the N+1 query issue because, instead of one SQL query, you executed 5 (1 + 4):  
``` php
$posts = Post::all();
```
``` php
@foreach($posts as $post)
    {{$post->comment->title}}
@endforeach
```

Fixing the N+1 query issue is very easy.
All you need to do is to get all the data you need in the original SQL query.

This time, only one SQL query is executed to fetch all the data we are further interested in using.  
``` php
$posts = Post::with('comments')->get();
```

## What is the purpose of Lazy Eager Loading?

Sometimes you may need to eager load a relationship after the parent model has already been retrieved.
For example, this may be useful if you need to dynamically decide whether to load related models:

``` php
$books = App\Book::all();

// Lazy eager load the book author
if ($someCondition) {
    $books–>load('author']);
}
```

I can also load multiple relations in the one time.

``` php
// Load all the users comments and posts
$users->load(['comments', 'posts']);
```

Or you even set additional query constraints on the eager loading query using a closure.
``` php
// Load the books and order them by published_date
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```
