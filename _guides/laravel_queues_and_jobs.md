---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Queues and Jobs
description: Explain how queues and jobs works
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Queues and Jobs


Some tasks can take too long to perform during a typical web request. 
Thankfully, Laravel allows you to easily create queued jobs that may be processed in the background. 
Some example of long tasks can be:
- parsing and storing an uploaded CSV file.
- send a lot of emails.

## Create a job

```php artisan make:job ProcessPodcast```

The job will be created in `app/Jobs`

## Class Structure

Like for event listeners, job classes are very simple, normally containing only a **handle** method that is invoked when the job is processed by the queue.
The **handle** method is invoked when the job is processed by the queue.

In this example, note that we were able to pass an Eloquent model directly into the queued job's constructor. Because of the **SerializesModels** trait that the job is using,
So, If your queued job accepts an Eloquent model in its constructor, only the identifier for the model will be serialized onto the queue.   
When the job is actually handled, the queue system will automatically re-retrieve the full model instance and its loaded relationships from the database.

```
<?php

namespace App\Jobs;

use App\Models\Podcast;
use App\Services\AudioProcessor;
use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * The podcast instance.
     *
     * @var \App\Models\Podcast
     */
    protected $podcast;

    /**
     * Create a new job instance.
     *
     * @param  App\Models\Podcast  $podcast
     * @return void
     */
    public function __construct(Podcast $podcast)
    {
        $this->podcast = $podcast;
    }

    /**
     * Execute the job.
     *
     * @param  App\Services\AudioProcessor  $processor
     * @return void
     */
    public function handle(AudioProcessor $processor)
    {
        // Process uploaded podcast...
    }
}
```

## Dispatching Jobs

Once you have written your job class, you may dispatch it using the dispatch method on the job itself. The arguments passed to the dispatch method will be given to the job's constructor:

``` php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $podcast = Podcast::create(...);

        // ...

        ProcessPodcast::dispatch($podcast);
    }
}
```
If you would like to conditionally dispatch a job, you may use the dispatchIf and dispatchUnless methods:
``` php
ProcessPodcast::dispatchIf($accountActive, $podcast);

ProcessPodcast::dispatchUnless($accountSuspended, $podcast);
```


## Delayed Dispatching
If you would like to specify that a job should not be immediately available for processing by a queue worker, you may use the delay method when dispatching the job. For example, let's specify that a job should not be available for processing until 10 minutes after it has been dispatched:

``` php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Jobs\ProcessPodcast;
use App\Models\Podcast;
use Illuminate\Http\Request;

class PodcastController extends Controller
{
    /**
     * Store a new podcast.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $podcast = Podcast::create(...);

        // ...

        ProcessPodcast::dispatch($podcast)
                    ->delay(now()->addMinutes(10));
    }
}
```

## Example 1

https://gistlog.co/JacobBennett/e60d6a932db98985f160146b09455988


Let's say that you create a new job called **SendWelcomeEmail** that will be dispatched
from your **RegistrationController** each time a new user signs up for your application.    

Ideally we don't want our user to have to wait for the welcome email to be sent when they sign up, 
so we make our **SendWelcomeEmail job** push onto the **queue** instead.    

To accomplish this, Laravel magic only requires that our job implements the **ShouldQueue** interface.   
Great!

Here is where the paths diverge.

### Scenario 1
We already have the email address of our user, and we pass through the users email to the job's constructor. 

Something like: 
```
$user = User::find(1);
dispatch(new SendWelcomeEmail($user->email));
```

### Scenario 2
We decide to pass the entire User model into our constructor and use the SerializesModels trait in our SendWelcomeEmail job. Something like
```
$user = User::find(1);
dispatch(new SendWelcomeEmail($user));
```
The difference in implementation is nearly indistinguishable, but let's play devils advocate for a minute and see how this might play out.

In Scenario 1, a user signs up, and our job is queued to run. Your application was featured on product hunt this morning however, and you have 100s of emails waiting to be sent in your queue. Your user upon signing up realizes they mistyped their email address, and promptly corrects it in the settings of your app. By the time your queue gets around to the SendWelcomeEmail for this user, the email that was serialized with the job is no longer valid. The user never gets their welcome email and the world stops spinning.

In Scenario 2, all the same steps take place, but when the queued SendWelcomeEmail job finally gets its turn to run, Laravel pulls the latest info for the passed in User, and the email gets sent to the updated email address! Praise be to you almighty dev.

