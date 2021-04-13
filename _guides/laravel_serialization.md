---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Serialization
description: Explain how serialization works in Laravel
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Serialization in Laravel


https://laravel.com/docs/8.x/eloquent-serialization#introduction


## What is serialization?
- It's a way to store the values of an object into a text string format.
- To convert our models, and our relationships to **arrays** or **JSON**.

## When is it used?
- Usually when we build JSON APIs, we will often need to convert our models and our relationships to arrays or JSON.

## How does Laravel address this?
- Eloquent includes convenient methods for:
  - making these conversions
  - controlling which attributes are included in your serializations.

---

## Serializing To Arrays

### How to convert to array just the loaded model attributes?

With the attributesToArray method:
``` php
$user = App\Models\User::first();
return $user->attributesToArray();
```

### How to convert to array the loaded model’s attributes and its loaded relationships (eager loading)?
With the toArray method:
``` php
$user = App\Models\User::with('roles')->first();
return $user->toArray();
```
- This method is recursive, so all attributes and loaded relations (eager loading) (including the relations of relations) will be converted to arrays.

### Can I convert the entire collections of models to array?
yes
``` php
$users = App\Models\User::all();
return $users->toArray();
```

---

## Serializing To JSON

### How to convert to JSON the loaded model’s attributes and its loaded relationships?
With the toJson method:
``` php
$user = App\Models\User::find(1);
return $user->toJson();
```
- This method is recursive, so all attributes and all relations (including the relations of relations) will be converted to JSON.
- Relationships
    - When an Eloquent model is converted to JSON, its loaded relationships will automatically be included as attributes on the JSON object.
    - Also, though Eloquent relationship methods are defined using "camel case", a relationship's JSON attribute will be "snake case".

### Can I specify the Json encoding options supported in PHP? Yes
``` php
return $user->toJson(JSON_PRETTY_PRINT);
```
- https://www.php.net/manual/en/function.json-encode.php

### How to automatically call the toJson method?
Casting a model or collection to a string.  
This will automatically call the toJson method on the model or collection.
``` php
$user = App\Models\User::find(1);
return (string) $user;
```

### What happens when I return an Eloquent object directly from your application's routes or controllers? (I didn’t fully understand this one…)
- Since models and collections are converted to JSON when cast to a string, we can return Eloquent objects directly from your application's routes or controllers.
- I didn’t fully understand this one...
``` php
Route::get('users', function () {
    return App\Models\User::all();
});
```

---

## How to hide attributes from the array / JSON?

Sometimes you may wish to limit the attributes, such as passwords, that are included in your model's array or JSON representation.

To do so, add a $hidden property to your model:
``` php
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * The attributes that should be hidden for arrays and JSON.
     *
     * @var array
     */
    protected $hidden = ['password'];
}
```

Alternatively, you may use the visible property to define just the attributes that should be included in your model's array and JSON representation

``` php
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;


class User extends Model
{
    /**
     * The attributes that should be visible in arrays and JSON
     *
     * @var array
     */
    protected $visible = ['first_name', 'last_name'];
}
```

---

## Can I append extra custom attributes To JSON or Array when I cast my model even if they are not part of my model?

Yes, to do this in the model:
1.  first define an accessor for the value.
2. then add the attribute name to the appends property.

``` php
<?php
namespace App\Models;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    /**
     * Get the administrator flag for the user. (1)
     *
     * @return bool
     */
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] === 'yes';
    }

    /**
     * The accessors to append to the model's array form. (2)
     *
     * @var array
     */
    protected $appends = ['is_admin'];
}
```
- Note that attribute names are typically referenced in "snake case", even though the accessor is defined using "camel case":

Instead of using the $append property in the model to add the accessors and attributes.
I can alternatively add them in the controller at runtime.
Using one of this two methods:
- append
    - This add just one attribute/property.
``` php
return $user->append('is_admin')->toArray();
```
- setAppends
    - This override the entire array of appended properties for a given model instance:
``` php
return $user->setAppends(['is_admin'])->toArray();
```

---

## How to customize the format of the DATES that get serialised?

Change format of all the dates
Overriding the serializeDate method. (in the model I guess)
``` php
class MyModel extends Eloquent {

    protected $dates = [
        'do',
        'date_closed',
        'file_stale_date'
    ];

    protected function serializeDate(\DateTimeInterface $date) {
        return $date->format('Y-m-d');
    }

}
```
- This format the values of all the dates of the model when I retrieve the model like this:
    - $post->toArray()
    - $post->toJson()

Change format of specific dates
Alternatively,
You may customize the serialization format of individual Eloquent date attributes by specifying the date format in the cast declaration:

``` php
class MyModel extends Eloquent {

    protected $casts = [
        'birthday' => 'date:Y-m-d',
        'joined_at' => 'datetime:Y-m-d H:00',
    ];

}
```

------
