---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - View composer
description: Explain View composers
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# View composers


## What is a View Composer ?

View composers are callbacks or class methods that are called when a view is rendered.
- We can use them to avoid passing data to the views in the controllers.
- So to shorten the code in the controllers.
- It bounds data to a view each time that view is rendered, organizing that logic into a single location.

---

## Practical example

https://www.youtube.com/watch?v=7QWZxjgvEQc&ab_channel=Coder%27sTape

We have 2 models:
- Channel
- Post (a post can belong to a channel)

List of channels, we display them to a user whenever he creates a post.
- So user can associate a post to a channel.

In our UI, we have a list of channels and we allow the user to filer trough them.


---

### What is the problem?
In some cases we need to pass the same variable to multiple views, and for that we waste many line of code.

``` php
class ChannelCrontroller extends Controller{
    public function create(){
        $channels = Channel::orderBy(’name’)->get();
        return view(‘channel.index’, compact(‘channels’);
    }
}
```

``` php
class PostCrontroller extends Controller{
    public function create(){
        $channels = Channel::orderBy(’name’)->get();
        return view(‘post.create’, compact(‘channels’);
    }
}
```

---

### Can I share a variable across any view ? (Option 1)

Yes, I can share the data across any view!
- Maybe we don’t use it often but can be useful.
- ALERT: 
  - We have to use it few times, because we are creating a query!!
  - We are hitting the DB every single time a view is loaded. Not overuse this! Don’t share unless it’s absolutely necessary.

We will do our edit now in the AppServiceProvider.php, but we can also create our own service provider just for this.

app/Providers/AppServiceProvider.php
``` php
...
use Illuminate\Support\Facades\View;
use App\Models\Channel;
...
public function boot()
{
    View::share('channels', Channel::orderBy('name')->get());
}
```
- Now every singe view will have a channels variable!

Then I can shorten my controllers because the variable is already accessible:
``` php
class PostCrontroller extends Controller{
    public function create(){
        return view(‘post.create);
    }
}
```

---

### Can I share a variable just with specific views ? (Option 2)
Yes, with View Composers!!

View composer attaches to specific views some data.
- First argument:
  - The views to which we want to pass this variable
  - string with view name or array of views
- Second argument:
  - Callback that take as parameter $view

app/Providers/AppServiceProvider.php
``` php
...
use Illuminate\Support\Facades\View;
use App\Models\Channel;
...
public function boot()
{
    View::composer(['post.create', 'channel.create'], function($view){
        $view->with('channels', Channel::orderBy('name')->get());
    });
}
```
- In this case we are much more granular.
- Just 2 views are receiving this data: post.create and channel.create.

---

### How do I share with all the post views without typing all their names?

Using wild cards!

``` php
View::composer([‘post.*', 'channel.create'], function($view){
        $view->with('channels', Channel::orderBy('name')->get());
    });
```
- All the views inside resources/post will get this $channel variable.

---

### What if we have a method that requires a lot of calculation ?  (Option 3)

We can use a dedicated class. (Composer class)

This class is not specified in the Laravel documentation but according to this video they should live in:
- app/Http/View/Composers
- So I create this dir!

Any composer class requires at least one method in it, called compose()
- The view that accept as parameter is not the facade, it’s Illuminate\View
  - So it’s different from the one we were using in the AppServiceProvider.php

app/Http/View/Composer/ChannelsComposer.php
``` php
<?php

namespace App\Http\View\Composers;
use App\Models\Channel;

use Illuminate\View\View;

class ChannelsComposer {
    public function compose(View $view){
        $view->with('channels', Channel::orderBy('name')->get());
    }
}
```
- Here we can do the same that we were doing in the app service provider with the normal view composer.
- But now we have encapsulated it in a class.
- So here we can add as Many methods as we need to calculate whatever the data the we want to pass to the view needs.

Then we have to import it again in the AppServiceProvider.php
- But instead of the callback we are gonna reference our View Composer Class
  app/Providers/AppServiceProvider.php
``` php
...
use Illuminate\Support\Facades\View;
use App\Models\Channel;
use App\Http\View\Composer\ChannelsComposer;
...
public function boot()
{
    View::composer([‘post.*', 'channel.create'], ChannelsComposer::class);
}
```



--- 

### Other example, injecting a repo


app/Http/View/Composer/CategoryComposer.php
``` php
<?php
namespace App\ViewComposers;

use Illuminate\View\View;
use App\Repositories\CategoryRepository;

class CategoryComposer{

protected CategoryRepository $categoryRepository;


/**
* Create a new categories composer.
*
* @param CategoryRepository $categories
* @return void
*/
public function __construct(CategoryRepository $categoryRepository)
{
    $this->categoryRepository = $categoryRepository;
}


/**
* Bind data to the view.
*
* @param View $view
* @return void
*/
public function compose(View $view)
{
    $view->with('categories', $this->categoryRepository->all());
}
}
```

---

### We can do one last refactor using partials.
Since the channels variable is used just to print lists and foreach loops.
- I can create two partials views and pass just that to the view composer first argument !!!!

resources/views/partials/channels/list.blade.php
``` php
<ul>
@foreach ($channels as $channel)
    <li>{{$channel->name}}</Ii>
@endforeach
<ul>
```

resources/views/partials/channels/dropdown.blade.php
``` php
<select name=“channel_id”  id=“channel_id”>
@foreach ($channels as $channel)
       <option value=“{{{$channel->id}}">{{$channel->name}}</option>
@endforeach
</select>
```


Then I can include in any form this partial:
``` php
@include(partials.channels.dropdown);
```


Then I will change my view composer to optimise even more!!
- p.s. This works with both Composer Callback(option 2) or Composer Class(option 3)
  app/Providers/AppServiceProvider.php
``` php
...
public function boot()
{
    View::composer('partial.channels.*', ChannelsComposer::class);
}
```
- We can pass just the dir with the partials related to the channels
- We don’t need an array anymore.

---

### Last last refactor, if we want to allow the id and name customisation wherever the dropdown is rendered.

resources/views/partials/channels/dropdown.blade.php
``` php
<select name=“{{$field_name ?? ‘channel_id’}}”  id=“{{$field_name ?? ‘channel_id’}}”>
@foreach ($channels as $channel)
       <option value=“{{{$channel->id}}">{{$channel->name}}</option>
@endforeach
</select>
```

Then I can include in any form this partial:
``` php
@include(partials.channels.dropdown, [‘field_name’ => ‘my_channels’]);
```