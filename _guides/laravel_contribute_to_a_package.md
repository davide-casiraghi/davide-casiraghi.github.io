---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Contribute to a package
description: Explain how to contribute to a Laravel open source package
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Contribute to a Laravel open source package



## How to contribute to a package?
After stumbling upon the internet for a while the method expained here is the way I found more simple and effective.  

You can use a fresh Laravel installation or a project you are working on and where you are using the package you want to contribute.

The package I choose is [AGILEDROP/laravel-telnyx](https://github.com/AGILEDROP/laravel-telnyx)

## Uninstall the package

If you previously installed the composer package you want to contribute uninstall it.
```
composer remove agiledrop/laravel-telnyx
```
## Clone the repo
- In the root folder of your laravel installation create a directory called `packages`.
- Add this directory to your `.gitignore` file.

``` 
# Local contributing composer packages
/packages
```

## Fork the repo
- Fork the repo on Github
- Clone the repo from your fork in the packages dir.   
  `git clone git@github.com:AGILEDROP/laravel-telnyx.git`
  

## Install the package
-  Add the repo to the composer.json of the Laravel project.

```
"repositories":[
    {
        "type": "path",
        "url": "./packages/laravel-telnyx",
        "options": {
            "symlink": true
        }
    }
],
"require": {
        ...
        "agiledrop/laravel-telnyx": "dev-develop",
        ...
    },
   ```
-  Then run
   `composer install`


## Edit the files
Now everything you edit on the package affect immediately the functionality offered by the package on your website.

## Run the tests
Before committing and pushing always remember to run the tests.

## Commit and push
Commit and push your code.

## Merge request
Ask for a merge request to the package maintainer. 

....add details here...

## Install the official package
When your edits will be merged by the maintainer and included in a new release of the package you can then remove the code you previously added for your local repository from the composer.json. 
Then run a:  
```
composer install
```

Then install the package in the usual way with a:
```
composer require agiledrop/laravel-telnyx
```