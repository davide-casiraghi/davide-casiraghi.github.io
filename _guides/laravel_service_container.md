---
layout: post
last_modified_on: "2021-01-27"
title: Laravel - Service Container
description: Explain how service container works in Laravel
series_position: null
author_github: https://github.com/davide-casiraghi/
author_name: Davide Casiraghi
tags: ["language: php", "framework: laravel"]
---

# Service Container

## What is a service?
- A Service is any PHP object that performs some sort of "global" task.
- Services are generic classes that are built to perform a specific task.
- For instance, on might have a mailer service or a user creation service. These are tasks that need to be repeated over and over in various places and thus don't fit into any one place that well in particular. That makes them good candidates to become a service.


## What is the Service Container in Laravel?

- It’s the bed rock of the entire framework, very important!
- It's a container for services.
- It’s a place where to store and retrieve services
    - Any kind of data: string, number, object, collection, etc.
- It’s a service that Laravel provides us that allow you to tell Laravel how an object or class needs to be constructed.
    - So then Laravel can figure it out from there.
- It’s a way to manage our class dependencies.
    - Every class has some dependencies with it, and you can inject them automatically.
  
Basic idea: You bind something in the container and then later you can resolve it.

```
Learning to code with Laravel is learning the philosophy of Laravel, its elegance and its beautiful syntax
A major part of Laravel’s philosophy is the Service Container or IoCcontainer.   
As a Laravel developer, understanding and using the Service Container properly is a crucial part in mastering your craft, as it is the core of any Laravel application.
```

- Basically the IoC Container is just an ordinary PHP class, but I like to think of it as my “Bag of tricks”.
- This “Bag” is where we will place or “Bind” everything we need to run a Laravel application smoothly, from interfaces implementations to directories paths and so on. Yes you can place pretty much anything you want in it !
- Now, since we have a single Object ($app) that contains all of our various bindings, it is very easy to retrieve them back or “resolve” them at any point in our code.


## Automatic Injection
https://laravel.com/docs/8.x/container#automatic-injection

### When we use Automatic Injection ?
We use Automatic Injection to include class dependencies in the constructors of our controllers, event listeners, middleware, and more.

### How to do Automatic Injection ?
Type-hinting the dependency in the constructor of a class.  
In this way the repository/model will automatically be resolved and injected into the class.

``` php
<?php
namespace App\Http\Controllers;
use App\Models\Users\Repository as UserRepository;


class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;


    /**
     * Create a new controller instance.
     *
     * @param  UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    /**
     * Show the user with the given ID.
     *
     * @param  int  $id
     * @return Response
     */
    public function show($id)
    {
        //
    }
}
```

----

## Manual injection

### When do you need manual injection?

So, when would you ever manually interact with the container?

1. When you write a class that implements an interface and you wish to type-hint that interface on a route or class constructor, you must tell the container how to resolve that interface. 

``` php
class AppServiceProvider extends ServiceProvider
{

    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->bind(CountryRepositoryInterface::class, CountryRepository::class);
        $this->app->bind(PostRepositoryInterface::class, PostRepository::class);
        $this->app->bind(VenueRepositoryInterface::class, VenueRepository::class);
        $this->app->bind(UserRepositoryInterface::class, UserRepository::class);
        ..
    }
```

2. When you write a Laravel package that you plan to share with other Laravel developers, you may need to bind your package's services into the container.
```
Add here example of a package service provider
```
----

## Example of Service Container use

From: https://www.youtube.com/watch?v=_z9nzEUgro4

1. I want to create a controller and inject a specific class.   
   
    - this is the regular way, in the next steps we will refactor to make it better.

/app/Http/Controllers/PayOrderController.php

``` php
namespace App\Http\Controllers;
use App\Billing\PaymentGateway;

class PayOrderController extends Controller{  
...
public function store(){
    $payentGateway = new PaymentGateway;
    dd(payentGateway->charge(2500)); // I charge 2500 cents
}
```

We create a payment gateway to have several payment systems in one application, but we always call the same PaymentGateway class.

/app/Billing/PaymentGatway.php
``` php
namespace App\Billing;
use Illuminate\Support\Str;

class PaymentGateway{    
    public function charge($amount){
        
        // charge the bank
        
         return [
            'amount' => $amount,
            'confirmation_number' => Str::random(),
        ]
    }
}
```
We create the route for this store.

Routes/web.php
``` php
Route::get(‘pay’, 'PayOrderController@store’);
```

So I can then visit:   
www.nameofmywebsite.com/pay

So we get back an array:
``` php
[
    'amount' => 2500,
    'confirmation_number' => 'k23h4kgjh3f76d6',
]
``` 
### Injecting the class

This works.   

Then I can get rid of `$payentGateway = new PaymentGateway;` injecting it. (4:20)

/app/Http/Controllers/PayOrderController.php
```
...
use App\Billing\PaymentGateway;
...
Public function store($payentGateway PaymentGateway){
    dd($payentGateway->charge(2500);
}
```

This works thanks to the Reflection class.


### Problem: pass a parameter while injecting

What if we have to specify the currency when we instantiate the object?

/app/Billing/PaymentGateway.php

``` php
<?php
namespace App\Billing;
use Illuminate\Support\Str;

class PaymentGateway{  
    private $currency;
  
    public function __construct($currency){
        $this->currency = $currency;
    }
    public function charge($amount){
         return [
            ‘amount’ => $amount,
            ‘confirmation_number’ => Str::random(),
            ‘currency’ => $this->currency,
        ]
    }
}
```

But, now the automatic injection return an error!   
And there is no way to pass the parameter there.    

To do that we can use the Service Provider!

Our application bootstrap everything that it needs in:
`app/Providers/AppServiceProvider.php`

app/Providers/AppServiceProvider.php
``` php
...
use App\Billing\PaymentGateway;
...
public function register(){

    //$this->app represent our entire app!
    $this->app->bind(PaymentGateway::class), function($app){
        return new PaymentGateway(‘usd’);
    });
}
...
```

We have explained: 
- whenever somebody requires a PaymentGateway, this is how you construct it.
  - we do it with a callback.
  - the callback actually get as parameter our entire $app.
- Now it works again! The parameter is coming from our AppServiceProvider
    - Now Laravel knows how to construct the payment gateway.

###  Why this is so important ?
- Because the controllers are not in charge anymore about how to create a payment gateway!
- And if in future we change our payment gateway the controller doesn't need to change, only our payment gateway needs to change.
  - The controller should not know so much about how our payment is processed.
- You have to try to encapsulate the logic to you don’t have to change many files because a small change
- If 50 controllers are instantiating the PaymentGateway, to make this change of currency would be a bloody hell!


###  Apply a discount -  Problem: the gateway is re-initialized all the times.

Now I want to give the possibility to create orders and to apply a discount.
If I create an order, I can pass also here the PaymentGateway in the constructor.

/app/Orders/OrderDetails.php
``` php
<?php
namespace App\Orders;
use App\Billing\PaymentGateway;

class orderDetails{

    private $paymentGateway;
    public function __construct(PaymentGateway $paymentGateway){
        $this->paymentGateway = $paymentGateway;
    }
    
    // Return all the order details
    public function all(){
        $this->paymentGateway->setDiscount(500);  //5 dollars off, we are expressing in cent.

        return [
            ’Name’ => ‘Victor’,
            ‘Address ‘ => ‘123 St James road, London'
        ];
    }
}
```

I add the function setDiscount() to my PaymentGateway.

/app/Billing/PaymentGatway.php
``` php
<?php
namespace App\Billing;
use Illuminate\Support\Str;

class PaymentGateway{  
    private $currency;
    private $discount = 0;
  
    public function __construct($amount){
        $this->currency = $currency;
    }
    public function charge($amount){
        
        // Charge the bank
    
         return [
            ‘amount’ => $amount - $this->discount,
            ‘confirmation_number’ => Str::random(),
            ‘currency’ => $this->currency,
            ‘discount’ => $this->discount,
        ]
    }
   public function setDiscount($amount){
        $this->discount = $amount;
   }
}
```

Now I can inject also the OrderDetails in the store method.
- We can do as many dependency injection as we need.
- Now I can get an order from OrderDetails calling the all() method.
  - This will also set the discount in the payment gateway.

**note:** i didn't bind OrderDetails in the ServiceProvider since it doesn't have any parameter.   
So it's enough the Automatic Injection.

/app/Http/Controllers/PayOrderController.php
``` php
...
Use App\Billing\PaymentGateway;
Use App\Orders\OrderDetails;
…
Public function store(OrderDetails $orderDetails, PaymentGateway $payentGateway){
    $order = $orderDetails->all();

    dd($payentGateway->charge(2500));
}
```

If I refresh the array returned by PaymentGateway has discount = 0, why?   
It should show 5$

``` php
[
    'amount' => 2500,
    'confirmation_number' => 'k23h4kgjh3f76d6',
    'currency' => 'usd',
    'discount' => 0,
]
``` 

Because whatever we bind into the container, every time that the class get asked for, we get a brand new object!
- Laravel invoke the binded function in the AppServiceProvider, every time a $payentGateway gets called.
- And the instances don’t know anything about each other.
- Usually the payment Gateway it’s just one instance, and you will use that instance in the entire request.


To solve this, we can use instead of **bind**, a **singleton**.
- It’s a way to tell Laravel that there is gonna be just one of this payment gateway!
- This means that the fist time that somebody request a PaymentGateway, Laravel invokes this callback, intanciating it.
- Every subsequent time after the first one, you get the same instance! 😍

app/Providers/AppServiceProvider.php
``` php
...
use App\Billing\PaymentGateway;
...
public function register(){
    $this->app->singleton(PaymentGateway::class), function($app){
        return new PaymentGateway(‘usd’);
    });
}
...
```

Now if we refresh we get the discount:

``` php
[
    'amount' => 2500,
    'confirmation_number' => 'k23h4kgjh3f76d6',
    'currency' => 'usd',
    'discount' => 500,
]
``` 

### If I have 2 paymentGateways, how can I dynamically chose ?

What if I have a **BankPaymentGateway** and a **creditCard paymentGateway**.
How can I chose which one I want to use for a specific order?

1. I transform my PaymentGateway to BankPaymentGateway
2. I extract an interface selecting just the 2 methods
- charge()
- setDiscount()

``` php
<?php
namespace App\Billing;

interface PaymentGatewayContract{
  public function setDiscount($amount);
  
  public function charge($amount);
}
```

An **iteraface** is a roadmap about how a class has to be constructed.


/app/Billing/BankPaymentGatway.php
``` php
<?php
namespace App\Billing;
use Illuminate\Support\Str;


class BankPaymentGatway implements PaymentGatewayContract{  
    private $currency;
    private $discount = 0;
  
    public function __construct($amount){
        $this->currency = $currency;
    }
    public function charge($amount){
         return [
            ‘amount’ => $amount - $this->discount,
            ‘confirmation_number’ => Str::random(),
            ‘currency’ => $this->currency,
            ‘discount’ => $this->discount,
        ]
    }
   public function setDiscount($amount){
        $this->discount = $amount;
   }
}
```
Now in my controller instead of requesting a BankPaymentGateway I just pass a PaymentGatewayContract.
I use the interface.  
- Now the controller doesn't care if the user use a credit card or a bank account.
- This because with the interface I’m sure that all the classes that implements the PaymentGatewayContract interface will provide the two methods setDiscount and charge.

/app/Http/Controllers/PayOrderController.php
```
...
use App\Billing\PaymentGatewayContract;
use App\Orders\OrderDetails;
...
Public function store(OrderDetails $orderDetails, PaymentGatewayContract $payentGateway){
    $order = $orderDetails->all();

    dd($payentGateway->charge(2500);
}
```

But if we go back to Chrome we get an error.
This because we need a concrete implementation of this interface, we cannot generate an interface.

So we solve it in the AppServiceProvider.
In the singleton instead of requesting as parameter the BankPaymentGateway we require PaymentGatewayContract.
- so whenever somebody requires a PaymentGatewayContract, we return the concrete impelmenaton of it (BankPaymentGateway).

app/Providers/AppServiceProvider.php
```
...
use App\Billing\PaymentGatewayContract;
...
public function register(){
    $this->app->singleton(PaymentGatewayContract::class), function($app){
        return new BankPaymentGateway('usd');
    });
}
...
```

To avoid an error, we have also to change in our OrderDetails.
/app/Orders/OrderDetails.php
from
``` php
public function __construct(BankPaymentGateway $paymentGateway){
``` 
to
``` php
public function __construct(PaymentGatewayContract $paymentGateway){
```  

### Why is it better than before?

I can create now a new PaymentGateway: **CreditPaymentGateway**
- This will represent my credit card processing.
- The credit card has a fee that has to be charged when the credit card is processed.

/app/Billing/CreditPaymentGateway.php
``` php
<?php
 namespace App\Billing;
use Illuminate\Support\Str;

class CreditPaymentGatway implements PaymentGatewayContract{  
    private $currency;
    private $discount = 0;
  
    public function __construct($amount){
        $this->currency = $currency;
    }
    public function charge($amount){

        $fees =  $amount * 0.03; // fee charged when the credit card is processed
        // charge the bank

         return [
            ‘amount’ => ($amount - $this->discount) + $fees,
            ‘confirmation_number’ => Str::random(),
            ‘currency’ => $this->currency,
            ‘discount’ => $this->discount,
            ‘fees’ => $fees,
        ]
    }
   public function setDiscount($amount){
        $this->discount = $amount;
   }
}
```

Now if I want to get payments from CreditCard instead of from Bank account I just need to switch in the AppServiceProvider.php
- from BankPaymentGateway to CreditPaymentGateway
  
  app/Providers/AppServiceProvider.php
``` php
...
use App\Billing\PaymentGatewayContract;
...
public function register(){
    $this->app->singleton(PaymentGatewayContract::class), function($app){
        // return new BankPaymentGateway(‘usd’);
         return new CreditPaymentGatway(‘usd’);
    });
}
...
```

Even better, we can allow the choice based on user input. 🙂
- Or we can check anything else, maybe a parameter in the configuration.
  app/Providers/AppServiceProvider.php
  
``` php
...
use App\Billing\PaymentGatewayContract;
...
public function register(){
    $this->app->singleton(PaymentGatewayContract::class), function($app){
        if (request()->has(‘credit’){
            return new CreditPaymentGatway(‘usd’);
        }
        else{
            return new BankPaymentGateway(‘usd’);
        }
         
    });
}
...
```

So I can then visit the url passing the parameter credit = true to use the **CreditPaymentGatway**
www.nameofmywebsite.com/pay?credit=true
