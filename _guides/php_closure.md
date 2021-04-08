---
layout: post
last_modified_on: "2021-01-27"
title: PHP - Closures
description: Explains Closures in PHP
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php"]
---

# Closures in PHP


## What is a closure ?

A closure is an object representation of an anonymous function. We can see that the anonymous function in the above code actually returns an object of closure which is assigned to and called using the variable $string.

So when we instance $string assigning an anonymus function it become a closure. They is an object. And the variables declared inside this object are private can be edited just buy the closure function.

The counter is protected by the scope of the anonymous function, and can only be changed using the add function!

## In which version of PHP have been introduced? Year?
Anonymous functions, implemented in PHP 5.3.

## What is an anonymous function?
An anonymous function is simply a function with no name.
Sso you would define it like this:
``` php
// Anonymous function  
function () {  
    return "Hello world";  
}  
```

## What is a lambda function?
A Lambda is an anonymous function that can be assigned to a variable or passed to another function as an argument.

Because the function has no name, you can’t call it like a regular function. Instead you must either assign it to a variable or pass it to another function as an argument.

To so use the anonymous function, we assign it to a variable and then call that variable as a function.

Lambdas are useful because they are throw away functions that you can use once. Often, you will need a function to do a job, but it doesn’t make sense to have it within the global scope or to even make it available as part of your code.

So to keep the code in order.
- Syntactic sugar… (here I can put the syntactic sugar definition)
- Maybe I can put as last slide, or second slide.
    - To motivate the talk, to keep the code in order and clean.


## What is a Closure?
A Closure is essentially the same as a Lambda apart from it can access variables outside the scope that it was created.

For example:

``` php
// Create a user  
$user = "Philip";


// Create a Closure  
$greeting = function() use ($user) {  
   echo "Hello $user";  
};


// Greet the user  
$greeting(); // Returns "Hello Philip"  
```


##  Why in this closure it passes variables in different ways? 😄

Because:
- $path is a required parameter when the function is invoked.
- &$deleteDirectory is a variable we want to include in the scope of the function.

Small little trick. You can use a closures in itself via reference.
Example to delete a directory with all subdirectories and files:
``` php
<?php
$deleteDirectory = null;
$deleteDirectory = function($path) use (&$deleteDirectory) {
    $resource = opendir($path);
    while (($item = readdir($resource)) !== false) {
        if ($item !== "." && $item !== "..") {
            if (is_dir($path . "/" . $item)) {
                $deleteDirectory($path . "/" . $item);
            } else {
                unlink($path . "/" . $item);
            }
        }
    }
    closedir($resource);
    rmdir($path);
};
$deleteDirectory("path/to/directoy");
?>
```


##  Why to use them ?
I use them when I find a monster function that I want to refactor.
  - I start grouping things that at the end should return a variable.
  - And I make them an anonymous function.
  - Then if I need to use them in different functions I make them a function.