---
layout: post
last_modified_on: "2021-04-13"
title: Laravel - Middleware
description: Explain laravel middlewares
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---


# Middlewares

Middleware acts as a bridge between a request and a response.
- It is a type of filtering mechanism.
- This chapter explains you the middleware mechanism in Laravel.

The middleware of your application are stored in the `app/Http/Middleware` directory.

## How to create a middleware ?

1. Create a middleware
```
   php artisan make:middleware EnsurePhoneIsVerified
```
2. Then we have to register it before using it.
    - there are two types of middleware in Laravel
        - Global middleware
            - run on every HTTP request of the application
        - Route middleware
            - will be assigned to a specific route.


The middleware can be registered at app/Http/Kernel.php.

This file contains two properties $middleware and $routeMiddleware.
- $middleware property is used to register Global Middleware
- $routeMiddleware property is used to register route specific middleware.


### Example 1
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

### Example 2
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
