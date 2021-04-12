---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Authorization and Middlewares
description: Explain laravel authorization and middlewares
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Laravel - Authorization and Middlewares

Limit Access to Authorized Users.   
https://laracasts.com/series/laravel-6-from-scratch/episodes/50


## Authorize in Blade Views

**How can I conditionally display something in a blade view?**

``` php
@can('update-post', $post)
    <a class="btn btn-primary float-right" href="{{ route('posts.edit',$post->id) }}">Edit</a>
@endcan
```

**How we define update-post ?**   

I have to define a **Gate**  in the AuthServiceProvider.
- In the gate I can check conditions and return true if the user is authorized.
- The following code is true when we have an authenticated users, otherwise is false (for guests)
- In this case I pass to the callback the $post.

app/Providers/AuthServiceProvider.php
``` php
public function boot()
{
    $this->registerPolicies();
    Passport::routes();

    Gate::define('update-post', function(User $user, Post $post){
        return true;
    });
}
```

In case we need to authorise also guests, we need to make $user optional.
- In this way it’s not mandatory. So also guests can see that.
``` php
Gate::define('update-post', function(?User $user, Post $post){
```


So if I have a belongsTo relationship defined in the post model I can do:
``` php
Gate::define('update-post', function(User $user, Post $post){
    return $post->user->is($user); //true if the authenticated $user is the post creator ($post->user)
});
```


---

## Authorise in Controllers methods


First way

Now we are authorising the display of a button, but this doesn’t prevent anybody to submit a push request!
https://laravel.com/docs/8.x/authorization#via-middleware
- Laravel provides a helpful authorize method to any of your controllers which extend the App\Http\Controllers\Controller base class.
- Like the can method we use for views, this method accepts the name of the action you wish to authorize and the relevant model.
- If the action is not authorized, the authorize method will throw an Illuminate\Auth\Access\AuthorizationException,
  - which the default Laravel exception handler will convert to an HTTP response with a 403 status code:$this->authorize

9:56- 11.24
``` php
public function update(Request $request, Post $post)
    {
        $this->authorize(‘update-post', $post);   // Check if user has permissions to update-post
        
    }
```
Or
``` php
public function store(Request $request)
{
    $this->authorize(‘create-post', Post::class);  // Check if user has permissions to create-post

}
```

BEST WAY!


```
What can we do to void to store all the Authorization logic in the AuthServiceProvider?
```
- app/Providers/AuthServiceProvider.php
- Since it get a mess soon!
- Better to use dedicate policy classes!!!

Create for each model the relative policy!

Check the documentation:
``` php
php artisan
php artisan help make:policy     (create a new policy class)
```

So I do:
``` php
php artisan make:policy PostPolicy --model=Post
```

- We can see it as a class that encapsulate an authorisation policy for a model

This creates:
app/Policies/PostPolicy.php
``` php
public function create(User $user)
{
    return true;  // True if the user is authenticated
}
public function update(User $user, Post $post)
{
    return $post->user->is($user); // True if the authenticated $user is the post creator ($post->user)
}
```

Here it generates automatically all the policies for any possible standard case!!
- We can then call any method we want here.
- Or delete the one that are automatically created.
- Or rename the one automatically created.
- What if I need only one method, because in my controller I have just one method?
  - I can delete all the rest..
  - Do we have permission to work with this model.

So I can return to my service provider and remove the two Gate I have created.
app/Providers/AuthServiceProvider.php
``` php
Gate::define('update-post', function(User $user, Post $post){
    return $post->user->is($user); //true if the authenticated $user is the post creator ($post->user)
});


Gate::define('create-post', function(User $user){
    return true; // True if the user is authenticated
});
```

And we update also the controller with the names that have been created in the PostPolicy.php
So from:
``` php
$this->authorize('create-post', Post::class);
$this->authorize(‘update-post', Post::class);
```
To:
``` php
$this->authorize('create', Post::class);
$this->authorize(‘update', Post::class);
```

And also I can change the name of the authorisation in the @can directive I used in the blade view!!