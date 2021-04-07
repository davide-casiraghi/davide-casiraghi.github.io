---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Service Providers
description: Explain service providers
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Service Providers


From: https://programmingpot.com/laravel/laravel-service-provider/

## What Is a Service Provider in Laravel?
- If the service container is something that allows you to define bindings and inject dependencies,
- Then the service provider is the place where it happens.

Service providers are the central place of all Laravel application bootstrapping. Your own application, as well as all of Laravel’s core services, are bootstrapped via service providers.

## But, what do we mean by “bootstrapped”?
In general, we mean registering things, including registering service container bindings, event listeners, middleware, and even routes.

## Where are defined the Service Providers?
`app/Providers/`

## How to Use Laravel Service Provider ?
There are two important methods, boot, and register, that your service provider may implement.

Let’s have a quick look at one of the core service providers to understand what it does.
Go ahead and open the:
vendor/laravel/framework/src/Illuminate/Cache/CacheServiceProvider.php
``` php
public function register()
{
    $this->app->singleton('cache', function ($app) {
        return new CacheManager($app);
    });
  
    $this->app->singleton('cache.store', function ($app) {
        return $app['cache']->driver();
    });
  
    $this->app->singleton('memcached.connector', function () {
        return new MemcachedConnector;
    });
}
```
The important thing to note here is:
- the register method, which allows you to define service container bindings.
  As you can see, there are three bindings for the cache, cache.store and memcached.connector services.



---

From: https://programmingpot.com/laravel/laravel-service-provider/


## How to Create Your Service Provider ?
``` php
php artisan make:provider ProgrammingPotServiceProvider
```

It gets created:
app/Providers/ProgrammingPotServiceProvider.php
``` php
<?php
namespace App\Providers;
  
use Illuminate\Support\ServiceProvider;
  
class EProgrammingPotServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap the application services.
     *
     * @return void
     */
    public function boot()
    {
        //
    }
  
    /**
     * Register the application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }
}
```

Then we need to register it.

 ## How to Register a Service Provider ?
To register your service provider, you just need to add an entry to the array of service providers in the config/app.php file.

config/app.php
``` php
'providers' => [
       ...
        /*
         * Application Service Providers...
         */
        ...
        App\Providers\ProgrammingPotServiceProvider::class,
],
```

But the service provider we’ve created is almost a blank template and of no use at the moment.
In the next section, we’ll go through a couple of practical examples to see what you could do with the register and boot methods.

## What to put in the boot and register methods?
If we want to register a view composer within our service provider this should be done within the boot method.
- The boot method is called after all other service providers have been registered, meaning you have access to all other services that have been registered by the framework!

``` php
public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
```
