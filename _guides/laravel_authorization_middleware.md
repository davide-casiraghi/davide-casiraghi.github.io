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

I have to define a **Gate** in the AuthServiceProvider.
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

In the previous example we authorized the display of a button in a blade view, 
but this doesn't prevent anybody to submit a push request!

https://laravel.com/docs/8.x/authorization#via-middleware
- Laravel provides a helpful authorize method to any of your controllers which extend the App\Http\Controllers\Controller base class.
- Like the can method we use for views, this method accepts the name of the action you wish to authorize and the relevant model.
- If the action is not authorized, the authorize method will throw an Illuminate\Auth\Access\AuthorizationException,
  - which the default Laravel exception handler will convert to an HTTP response with a 403 status code:$this->authorize

So, in our controller methods I can use it like this:
``` php
public function update(Request $request, Post $post)
    {
        $this->authorize(‘update-post', $post);   // Check if user has permissions to update-post
    }
```
Or:
``` php
public function store(Request $request)
{
    $this->authorize(‘create-post', Post::class);  // Check if user has permissions to create-post
}
```


### What can we do to void to store all the Authorization logic in the AuthServiceProvider?

BEST WAY!

- app/Providers/AuthServiceProvider.php
- Since it get a mess soon!
- Better to use dedicate policy classes!!

Create for each model the relative policy!

Check the documentation:
``` php
php artisan
php artisan help make:policy     (create a new policy class)
```

So I do:
```
php artisan make:policy PostPolicy --model=Post
```

- We can see it as a class that encapsulate an authorisation policy for a model

This creates:
app/Policies/PostPolicy.php
``` php
public function create(User $user)
{
    return true;  // True for any user that is authenticated.
}
public function update(User $user, Post $post)
{
    return $post->user->is($user); // True if the authenticated $user is the post creator ($post->user).
}
```

Here it generates automatically all the policies for any possible standard case!!
- We can then call any method we want here.
- Or delete the one that are automatically created.
- Or rename the one automatically created.
- What if I need only one method, because in my controller I have just one method?
  - I can delete all the rest.
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

And, we update also the controller with the names that have been created in the PostPolicy.php
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

----

















## How to manage user roles?
- Eg. Administrator, manager etc..
- And check Eg. isAdmin()

Laravel with the authorisation plugin doesn't provide roles out of the box when we install.

I can use a spatie package for that:
https://spatie.be/docs/laravel-permission/v3/introduction
```
composer require spatie/laravel-permission
```

With this package I can assign roles to users.
``` php
  $user->assignRole('Admin');
```

I can add to the user model a function to check if the user is an admin or super admin.   

app/Models/User.php
``` php

/**
 * Return true if the user is an administrator
 *
 * @return bool
 */
public function isAdmin(): bool
{
    return $this->hasRole(['Super Admin', 'Admin']);
}
```


Now:
In the CONTROLLERS I can check like this:
```
Auth::user()->isAdmin()
```

Then in your BLADE VIEWS I can call it:
```
@if(Auth::user()->isAdmin())
    enter code here
@endif
```


Now I can allow the administrators to do whatever action to the posts?
- I can add to the Post policy a method before()
  - This fires before any other policy method

app/Policies/PostPolicy.php
``` php
...
/**
 * Give the admin full authorization to this resource
 *
 * @param  \App\Models\User  $user
 * @return mixed
 */
public function before(User $user)
{
    if($user->isAdmin()){
        return true;
    }
}
...
```

Or even better!
- We can give to the admins full access to any resource of the website
- So we handle globally defining it in the AuthServiceProvider.php

app/Providers/AuthServiceProvider.php
``` php
public function boot()
{
    $this->registerPolicies();

    // Give the admin full authorization to this resource
    Gate::before(function (User $user){
        if($user->isAdmin()){
            return true;
        }
    });
}
```


### Authentication at the Route level

https://laracasts.com/series/laravel-6-from-scratch/episodes/53?autoplay=true

We can also authenticate at the route level using a middleware.

``` php 
Route::get(‘/reports’, function (){
    Return ’the secret reports’;
})->middleware(‘can:view_reports');
```

### Diving deeper in the middleware

#### Example 1
In this example we see two routes that require two middleware to be verified to access them.

routes/web.php
``` php 
Route::group(['middleware' => ['is_admin', 'phone_verified']], function() {
  Route::get('/users', [UserController::class, 'index'])->name('users.index');
  Route::get('/events', [EventController::class, 'index'])->name('events.index');
});
``` 

app/Http/Kernel.php
``` php 
protected $routeMiddleware = [
  ...
  'phone_verified' => \App\Http\Middleware\EnsurePhoneIsVerified::class,
  ...
];
``` 


app/Http/Middleware/EnsurePhoneIsVerified.php
``` php 
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Support\Facades\Auth;


/**
 * When the user still has to verify the phone number,
 * This middleware shows on any page the flash message 'Please verify your phone number'
 * with a link to the 'verify phone number' page.
 */
class EnsurePhoneIsVerified
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if (
            !empty($request->user()->profile->phone_verification_code)
            && empty($request->user()->profile->phone_verified_at))
        {
            session()->flash('warning', "<a href='/verifyPhoneNumber'>Please verify your phone number</a> <br> If you haven't got the verification code via SMS <a href='#'>send it again</a>.");
        }

        return $next($request);
    }
}
``` 

#### Example 2
In this other middleware we redirect the user that is not authorized to the homepage.

app/Http/Kernel.php
``` php
protected $routeMiddleware = [
...
'is_admin' => \App\Http\Middleware\EnsureUserIsAdmin::class,
...
];
``` 

app/Http/Middleware/EnsureUserIsAdmin.php
``` php
<?php

namespace App\Http\Middleware;

use App\Providers\RouteServiceProvider;
use Closure;

class EnsureUserIsAdmin
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if (!$request->user()->isAdmin()) {
            return redirect(RouteServiceProvider::HOME)
                ->with('success','This section of the website is accessible just by admins');
        }

        return $next($request);
    }
}
``` 

