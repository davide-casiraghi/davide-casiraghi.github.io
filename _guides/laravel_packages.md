---
last_modified_on: "2021-01-28"
title: Useful Laravel Packages
description: A collection of useful Laravel packages
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Useful Laravel Packages

## Laravel Excel
Easily export collections to Excel and CSV format.   
[maatwebsite/excel](https://github.com/Maatwebsite/Laravel-Excel) 

## Laravel Model Status
Assign statuses to Eloquent models.   
[spatie/laravel-model-status](https://github.com/spatie/laravel-model-status)

## Laravel Model Status
Associate files with Eloquent models.   
[spatie/laravel-medialibrary](https://github.com/spatie/laravel-medialibrary)

## Laravel Permission
Associate users with permissions and roles.   
[spatie/laravel-permission](https://github.com/spatie/laravel-permission)

## Laravel Model Status
This package makes it easy to get structured search from a variety of sources.  
It can search through many models at the same time.  
Very useful for a global search functionality.    
[spatie/laravel-searchable](https://github.com/spatie/laravel-searchable)

## Location
Retrieve a user's location from their IP address using external web services, or through a flat-file database hosted on your server.
There are many packages to retrieve the user IP, this one is the easiest I found and also for free many of the others change a fee.   
[stevebauman/location](https://github.com/stevebauman/location)

## Model Versioning
This package offer auding functionality. 
It's better than [Laravel Auditing](http://www.laravel-auditing.com/) since it offers also the possiblity to version Many-to-many relations.  
[altek/accountant](https://altek.gitlab.io/accountant/)

To track many to many relationships require also this package.   
[altek/eventually](https://altek.gitlab.io/eventually/installation.html)

## Larastan
Code static analysys for Laravel, is a PHPStan wrapper for Laravel.  
Finds errors in your code without actually running it. 
It catches whole classes of bugs even before you write tests for the code.     
[nunomaduro/larastan](https://github.com/nunomaduro/larastan)

### Migrate DB
A package to migrate one database to another in Laravel apps.

[Laravel news about](https://laravel-news.com/laravel-migrate-db-package)  
[andrey-helldar/migrate-db](https://github.com/andrey-helldar/migrate-db)



## Commercial packages

### Laravel Ray
Allows a better debugging experience then dump() and dd().  
Especially to debug AJAX stuffs.  
[spatie/laravel-ray](https://myray.app/)


### Laravel Tinkerwell
It's a PHP code editor on steroids.
Allows to run any PHP code locally or even directly in production.

Once set the work directory as the one of the project we are working on,
allows interacting directly with the db data and to see the output of **Eloquent** and **Collection** methods.

I found out that it helps to speed up my dev flow since I can play with the data checking the output immediately, and without writing code in the project I'm working on.

[laravel/tinker](https://tinkerwell.app/)


