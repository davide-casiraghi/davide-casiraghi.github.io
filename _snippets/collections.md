---
id: collections
title: Collections
description: List of collection methods
---


[What are Laravel Collections?](../../guides/beginner/laravel_introduction_to_collections/)


### map()
The map() method iterates through the collection and passes each value to the given closure/callback.   
The callback is free to modify the item and return it, thus forming a new collection of modified items.
``` php
$collection = collect(['taylor', 'abigail']);
$collection->map(function ($name) { 
    return strtoupper($name);
});
$collection->toArray(); 
// ['Taylor', 'Abigail', null]
```


### reject()
The reject method filters the collection using the given closure/callback.  
When the closure return true, the item is removed from the resulting collection.

``` php
$collection = collect(['taylor', 'abigail']);
$collection->reject(function ($name) {
    return empty($name);
});
$collection->toArray(); 
// ['taylor', 'abigail']
```

