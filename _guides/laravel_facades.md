---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Facades
description: Explain how facades works
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Facades

## Resources of this tutorial
- https://www.sitepoint.com/how-laravel-facades-work-and-how-to-use-them-elsewhere/
- https://www.youtube.com/watch?v=zR6JnwH7MSQ&t=13s++++++
    - In this tutorial it uses the splat operator ...
    - https://stackoverflow.com/questions/41124015/meaning-of-three-dot-in-php
- https://stackoverflow.com/questions/40784181/how-to-access-a-packages-methods
- https://stackoverflow.com/questions/25289124/package-facade-not-found

## What is a Facade in Laravel?

As you probably know, every service inside the container has a unique name.

In a Laravel application, to access a service directly from the container, we can use the App::make() method or the app() helper function.

```
<?php

App::make('some_service')->methodName();
```

Laravel uses facade classes to make services available to the developer in a more readable way.
By using a facade class, we would only need to write the following code to do the same thing:

```
// ...
someService::methodName();
// ...
```

In Laravel, all services have a facade class.
These facade classes extend the base Facade class which is part of the Illuminate/Support package. The only thing that they need to implement is the **getFacadeAccessor()** method, which returns the service name inside the container.

In the above syntax:
- someService refers to the facade class
- The methodName is in fact a method of the original service in the container


## Practical example

From:
- https://www.youtube.com/watch?v=zR6JnwH7MSQ&t=13s++++++
  - In this tutorial it uses the splat operator ...
  - https://stackoverflow.com/questions/41124015/meaning-of-three-dot-in-php


1. Create a service (17:18)
- This service sends an email and dump a message.

app/Services/PostcardSendingService.php
 ``` php
 <?php
 
 namespace App/Services;
 
 class PostcardSendingService{
  private $country;
  private $width;
  private $height;
 
  public static function __construct($country, $width, $height){
    $this->country = $country;
    $this->width = $width;
    $this->height = $height;
  }

  public function hello(){
    Mail::raw($message, function($message) use($email){
      $message->to($email);
    });

    dump("A postcard was sent with the message: " . $message );
  }
}
```

2. Register in the App service provider a singleton.  
- This singletone has going to be referenced as **Postcard**.
- Here what we are outputting is how to generate the PostcardSendingService.

app/Providers/AppServiceProvider.php
``` php
public function boot(){
  $this->app->singleton('Postcard', function($app){
    return new PostcardSendingService('us', 4, 6);
  });
}
```

3. Define the facade
...add details here...
   17:00....
 ``` php
 <?php
 
  namespace App/Services;
 
  class Postcard{
    protected static function resolveFacade($name){
      return $app()[$name];
    }
    
    public static function __callStatic($method, $arguments){
      return self::resolveFacade('Postcard'))
        ->method(...$arguments);
    } 
  }
 ```

4. Call the service using the facade from a Route.
16:26
routes/web.php
``` php
Route::get('/send-a-postcard', function(){
    Postcard::hello('Hello from facade', 'info@test.com');
});
```


