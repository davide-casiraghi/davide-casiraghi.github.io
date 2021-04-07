---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Events and Listeners
description: Explain how events and listeners works
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Events and Listeners

## What is an event?
- Represent an event that can happen in your application
- Event classes are typically stored in app/Events

## What is a listener?
- Some actions triggered when the event happen.
- Listeners classes are stored in app/Listeners.
- An event listener is a procedure or function in a computer program that waits for an event to occur.


## How to generate the events and listeners with Artisan?

1. First you add events or listeners to your EventServiceProvider.
2. Then you run:
```
php artisan event:generate
```
- This command will generate any events or listeners that are listed in your EventServiceProvider.
- Events and listeners that already exist will be left untouched.


## How to register the Events and Listeners in the EventServiceProvider?
- EventServiceProvider.php is the place to register all of your application's event listeners.
- You may add as many events to this array as your application requires
- The `listen` property contains an array of all events (keys) and their listeners (values).
    - You may add as many events to this array as your application requires.
    - For example, let's add a OrderShipped event:

app/Providers/EventServiceProvider.php
``` php
/**
 * The event listener mappings for the application.
 *
 * @var array
 */
protected $listen = [
    'App\Events\OrderShipped' => [   //Event
        'App\Listeners\SendShipmentNotification’,    //Listener
    ],
];
```

## Why the directories are not present in my Laravel vanilla installation?  
This directories are created for you as you generate events and listeners using Artisan console commands.

## Can one event have multiple listeners?
- Yes a single event can have multiple listeners that do not depend on each other.
- This is great to decouple (disaccoppiare) various aspects of your application.
- eg
    - Event: OrderShipped
    - Listener:
        - Slack notification
        - etc..


## Why do I have to manually register an event? Does it have any advantage?
https://laravel.com/docs/8.x/events#manually-registering-events

## Event Discovery

Instead of registering events and listeners manually in the $listen array of the EventServiceProvider, you can enable automatic event discovery.
- When event discovery is enabled, Laravel will automatically find and register your events and listeners by scanning your application's Listeners directory.
    - app/Listeners
- In addition, any explicitly defined events listed in the EventServiceProvider will still be registered.
    - app/Providers/EventServiceProvider.php
        - protected $listen[]


## How to enable Event Discovery ?

Event discovery is disabled by default, but you can enable it by overriding the shouldDiscoverEvents method of your application's EventServiceProvider.

app/Providers/EventServiceProvider.php
``` php
/**
 * Determine if events and listeners should be automatically discovered.
 *
 * @return bool
 */
public function shouldDiscoverEvents()
{
    return true;
}
```


##  How to define and event?

- Eg.OrderShipped
- Let's assume our generated OrderShipped event receives an Eloquent ORM object.
``` php
<?php

namespace App\Events;

use App\Models\Order;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $order;

    /**
     * Create a new event instance.
     *
     * @param  \App\Models\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```

- As you can see, this event class contains no logic.
    - It is a container for the Order instance that was purchased.
    - The SerializesModels trait used by the event will gracefully serialize any Eloquent models if the event object is serialized using PHP's serialize function.
        - Check Laravel serialisation in Evernote
            - So if I format the dates in the model I will get the formatted dates.

## How to define a Listener?
https://laravel.com/docs/8.x/events#defining-listeners
- Event listeners receive the event instance in their handle() method.
    - In our case we Access the order using $event->order.
- Within the handle method, you may perform any actions necessary to respond to the event
- The event:generate command will automatically import the proper event class and type-hint the event on the handle method.

``` php
<?php
namespace App\Listeners;
use App\Events\OrderShipped;

class SendShipmentNotification
{
    /**
     * Create the event listener.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }


    /**
     * Handle the event.
     *
     * @param  \App\Events\OrderShipped  $event
     * @return void
     */
    public function handle(OrderShipped $event)
    {
        // Access the order using $event->order...
    }
}
```
- Your event listeners may also type-hint any dependencies they need on their constructors.
    - All event listeners are resolved via the Laravel service container, so dependencies will be injected automatically.

    
## Is it possible to add listeners that are not in the app/Listeners dir?
Yes overriding the discoverEventsWithin() method.