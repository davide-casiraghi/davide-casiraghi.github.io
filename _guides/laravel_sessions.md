---
layout: post
last_modified_on: "2021-04-13"
title: Laravel - Sessions
description: Explain laravel sessions
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---


# Sessions


## What are used the sessions for?
- store the data on server side.
- to store data when I go from a page to another one.
- User login and authentication.

## Where are the session data stored ?
You can specify it at `config/session.php`
It can be:
- file - sessions are stored in storage/framework/sessions.
- cookie - sessions are stored in secure, encrypted cookies.
- database - sessions are stored in a relational database.
- memcached / redis - sessions are stored in one of these fast, cache based stores.
- dynamodb - sessions are stored in AWS DynamoDB.
- array - sessions are stored in a PHP array and will not be persisted.



## Example
https://www.youtube.com/watch?v=idw3k9EvmcE
- Create a login form
- Store data in session
- Get data from session
- Delete data from session


views/login.blade.php
``` html
<form action="user" method="post">
    @csrf
    <input type="text" name="username" placeholder="enter user name"> <br>
    <input type="text" name="password" placeholder="enter user password"> <br>
    <button type="submit">Login</button>
</form>
``` 

routes/web.php
``` php
Route::view('login', 'login');
``` 

Create the controller:
``` 
php artisan make:controller UserAuth
``` 
app/Http/Controllers/UserAuth.php
``` php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserAuth extends Controller{
	function userLogin(Request $request){
		return $request->input();
	}
}
``` 

I add it to the routes.   
routes/web.php
``` php
Route::post("user", [UserAuth::class, 'userLogin']);
Route::view('login', 'login');
``` 

I can test the form. And it will show the data on submit.

### Store the data in the session

Put the data in a variable `$data` and add it to the session with a key named `user`, then delete it.

app/Http/Controllers/UserAuth.php
``` php
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserAuth extends Controller{
	function userLogin(Request $request){
		$data = $request->input();
		$request->session->put('user', $data['user]);
		
		// To test
		//echo session('user');
		
		// Redirect to the profile route
		return redirect('profile');
	}
}
```

Now, I create a profile view to show the data.
views/profile.blade.php
```
<h1>Profile Page</h1>
<h2>Hello, {{session('user') }}</h2>
```

I add it to the routes.   
routes/web.php
``` php
Route::post("user", [UserAuth::class, 'userLogin']);
Route::view('login', 'login');
Route::view('profile', 'profile');
``` 

### Delete data from the session

Now we want to allow the user to logout.
To do that we will delete the user data from the session.

I add a logout link to the profile view.
views/profile.blade.php
```
<h1>Profile Page</h1>
<h2>Hello, {{session('user') }}</h2>

<a href="/logout">Logout</a>
```

Than write a route for that.  
When user click on logout, if the session has the user key, remove it.

routes/web.php
``` php
Route::post("user", [UserAuth::class, 'userLogin']);
Route::view('login', 'login');
Route::view('profile', 'profile');

Route::get('/logout', function(){
    if(session->had('user')){
        session()->pull('user');
    }
    return redirect('login');
});
``` 

And this logout already work.

Then we want to not allow the user to go to the login page when we are logged in.

routes/web.php
``` php
Route::post("user", [UserAuth::class, 'userLogin']);
//Route::view('login', 'login');
Route::view('profile', 'profile');

Route::get('/login', function(){
    if(session->had('user')){
        return redirect('profile');
    }
    return view('login');
});


Route::get('/logout', function(){
    if(session->had('user')){
        session()->pull('user');
    }
    return redirect('login');
});
``` 