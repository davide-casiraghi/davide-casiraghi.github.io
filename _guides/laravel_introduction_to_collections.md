---
last_modified_on: "2021-01-28"
title: Laravel - Introduction to Laravel Collections
layout: page
description: A brief introduction to Collections
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Introduction to Laravel Collections


A **collection** is an object that contains an array and lets us perform operations on it by calling methods on the collection instead of passing the array into functions.

### Two kind of collections
Laravel have two types of collections:

- The **Illuminate\Support\Collection** class provides a fluent, convenient wrapper to work with arrays of data.

- All **Eloquent methods** that return more than one model result, return instances of the **Illuminate\Database\Eloquent\Collection** class, including results retrieved via the get method or accessed via a relationship.

I will refer in the rest of the article to this two collections as **Base Collection** and **Eloquent Collection**.

We will discuss [later](../../guides/beginner/laravel_introduction_to_collections/#two-types-of-collection) in this article how to deal with this 2 kinds.


## Start using collections

The first two methods we should learn to use collections should be **collect()** and **toArray()**.

### collect()

``` php
// Create a collection from an array
$collection = collect(['one', 'two', 'three']);

// Create a collection from a multidimensional array
$collection = collect([
     ['id'=>1, 'name'=>'Hardik', 'age'=> 29],
     ['id'=>2, 'name'=>'Harsukh', 'age'=> 42],
     ['id'=>3, 'name'=>'Bhagat', 'age'=> 23],
]);
```

### toArray()

``` php
// Converts the collection into a plain PHP array.  
$collection = collect(['name' => 'Desk', 'price' => 200]);
$collection->toArray();

// [['name' => 'Desk', 'price' => 200]]
```

If the collection's values are Eloquent models, **toArray()** will also convert them to arrays.
``` php
// Collection containing 2 posts. 
$posts = Post::where('id', 2)->orWhere('id', 3)->get();
$posts->toArray();

/*
[
   [
     "id" => 2,
     "title" => "Harum placeat molestiae.",
     ..
     "created_at" => "2021-01-23T07:15:41.000000Z",
     "updated_at" => "2021-01-23T07:15:41.000000Z",
   ],
   [
     "id" => 3,
     "title" => "Dicta veritatis et.",
     ...
     "created_at" => "2021-01-23T07:15:41.000000Z",
     "updated_at" => "2021-01-23T07:15:41.000000Z",
   ],
 ]
*/
```

### Example of collection's use

The following example uses the collection methods [map()](../../../docs/laravel/collections#map) and [reject()](../../../docs/laravel/collections#reject).  
They both accept callbacks as parameter to execute their tasks.

``` php
// Create a new collection from the specified array
$collection = collect(['taylor', 'abigail', null]);

// Capitalize each element
$collection->map(function ($name) { 
    return strtoupper($name);
});

// Remove all empty elements
$collection->reject(function ($name) {
    return empty($name);
});

$collection->toArray();
// ['Taylor', 'Abigail']

```


## Two types of collection

Laravel have two types of collections: 
- Illuminate\Support\Collection
- Illuminate\Database\Eloquent\Collection

They are different even if they share many common methods.

### The key is whether we are using eloquent data or not

The **collect()** method returns an instance of **Illuminate\Support\Collection**. **Illuminate\Database\Eloquent\Collection** is returned only when getting data from eloquent methods.

Because **Illuminate\Database\Eloquent\Collection** extends **Illuminate\Support\Collection**, eloquent collection of course has more methods than the base collection (e.g., find, getKey). 

Some methods are also overwritten (e.g., contains, unique). Thus, we need to be aware of the collection functions to avoid misusing them.

So, if we will use **find()** on a method created with **collect()** we will get an error.


### Make the return type consistent
Since the available functions between these two collections are different, it may be confusing for when we can or can not use some methods (as described above). 
It can be also confusing when we need to type hint the functions return values.

The better way to make this simple is to return the results all in one type.   
It can be an **array** or a **base collection**.

Once we unify the return type, we will not be bothered by the concern that what methods can be used when developing.


### How to transform an Eloquent Collection into a Base Collection?
Laravel provides a function toBase() to help transform the eloquent collection into base collection.

``` php
// Illuminate\Database\Eloquent\Collection
$event = Event::where('id',2)->orWhere('id',3)->get();

// Illuminate\Support\Collection
$event = Event::where('id',2)->orWhere('id',3)->get();
$event->toBase();

// Illuminate\Support\Collection
$users = User::toBase()->get();
``` 

### Collection methods

The collection class offer a lot of convenient methods to interact with the data.
You can find the list of all the methods [here](../../../docs/laravel/collections).


### Performances

In terms of performance, **Base Collections** saves almost 50% of the memory compared to **Eloquent collections**.   
You can find more in the article written by Mitul Golakiya quoted below in the references.

### Tools

While coding I like to play with collections using [Tinkerwell](https://tinkerwell.app/).  
It's a commercial tool written by Beyond Code that, once set the work directory as the one of the project we are working on, 
allows to interact directly to the database and to see the output of **Eloquent** and **Collection** methods.  

I found out that it helps to speed up my dev flow since I can play with the data checking the output immediately, and without writing code in the project I'm working on.

#### References:  
- [Two Types of Collections in Laravel ](https://medium.com/@lynnlin827/two-types-of-collections-in-laravel-888d43858c4e) by Lynn Lin  
- [toBase function in Laravel Eloquent](https://www.infyom.com/blog/tobase-function-in-laravel-eloquent
) by Mitul Golakiya
