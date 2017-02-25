---
title: >-
  Custom Form Validation in Symfony 3.1.4: Symfony Form Component
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: https://huddlehouse.io
excerpt: "This is a more advanded tutorial and assumes you have a working form built using Symfony's Form Component. If you need a good reference to get you started building a form Symfony's website offers a great reference here."
tags:
  - Symfony3
  - Symfony Form Component
  - Callbacks
  - Form Validation
date: 2016-09-30 12:38:39
---
## Prerequisites

This is a more advanded tutorial and assumes you have a working form built using Symfony's Form Component. If you need a good reference to get you started building a form Symfony's website offers a great reference [here](http://symfony.com/doc/current/forms.html). 

This tutorial also uses [Callbacks](http://symfony.com/doc/current/reference/constraints/Callback.html). They were not intruduced in Symfony until 3.1. If you do not have Symfony 3.1 you can always [upgrade](http://symfony.com/doc/current/setup/upgrade_major.html).

> Note: If your version of Symfony is 3.0.* the upgrade should be very smooth and easy. It was as simple as opening my composer.json and changing "symfony/symfony": "3.0.*" to "symfony/symfony": "3.1.*" and then in the terminal I ran `composer update`  

## Background

Hands down, my least favorite thing to do as a web developer is handling form validation. It is very time consuming, tedius and overall it's not rewarding when you finish. At the same time my feelings aren't going to make it go away anytime soon, so lets make it easier. :)

Symfony's Form Component (FC) was hard to learn at first (I wish I could say it made sense from the start!). I could do basic forms, but that was all. The really nice thing about the FC is that when you save the form Symfony automatically persists the data to the DB. 

The FC starts to get more complex as your entities grow in complexity. For example when you have One-to-Many relationships (Many-to-one's surprisingly aren't hard). Custom validation has always been hard with the FC, until now! You can do incredibly specific cases incredibly easy.

## Getting Started

For this tutorial I am inside of a Symfony application that is using FOSUserBundle for user management. I have a User entity already made that I will be editing.

Here is what part of my form looks like. If a User has `Retailer` checked they can't be in 2 `Channels`

![Check if retailer is checked.](retailer.png)
<br>
![Can't have two channels selected](channel.png)

 
To start lets set up our `validate` function inside of the User Entity.

<button class="right copy btn" data-clipboard-target="#validate"><i class="fa fa-clipboard"></i></button>
<div id='validate'>
```
// src/AppBundle/Entity/User.php
namespace AppBundle\Entity;

use Symfony\Component\Validator\Constraints as Assert;
use Symfony\Component\Validator\Context\ExecutionContextInterface;

class User
{
    /**
     * @Assert\Callback
     */
    public function validate(ExecutionContextInterface $context, $payload)
    {
        // ...
    }
}
```
</div>

Now we have that set up, we can check our data to see if it matches our specefic criteria. If it does not we trigger an error. 

> Note: In the code at `->atPath('user_channels')` that is reffering to the name of the element that was declared on the UserFormType class, that has incorrect data. 

<button class="right copy btn" data-clipboard-target="#user"><i class="fa fa-clipboard"></i></button>
<div id='user'>
```
// src/AppBundle/Entity/User.php

    /**
     * @Assert\Callback
     */
    public function validate(ExecutionContextInterface $context, $payload)
    {
        if($this->hasRole('ROLE_RETAILER')) {
            $count = 0;
            foreach($this->getUserChannels() as $channel)
                $count++;
            if ($count > 1) {
                $context->buildViolation('A retailer can only be on one Channel.')
                    ->atPath('user_channels')
                    ->addViolation();
            }
        }
    }
```
</div>

This code checks to see if the User had the role Retailer added, if so it counts the number of channels they are in. If it is more than one then it sends an error back to the screen and does not allow the form tobe submitted. 

To display the error to the user you have to add a place to show the errors, it is very similar to how the form gets displayed in TWIG.

> Note: To get your error to display in a red error box you have to wrap it in HTML and add CSS classes to make the box red. I included those in my code. 

In your view add this before your form:

<button class="right copy btn" data-clipboard-target="#form"><i class="fa fa-clipboard"></i></button>
<div id='form'>
```php
	{% if not form.vars.valid %}
        <div class="row">
            <div class="col-md-12">
                <div class="alert alert-danger ">
                    {{ form_errors(form.user_channels) }}
                </div>
            </div>
        </div>
    {% endif %}
```
</div>

This is what my error looks like: 

![This is what my error looks like.](error.png)

## Conclusion

I hope this was a clear example of how easy it is to add custom form validation to a form built with Symfony's Form Component.

If you have any questions please ask us in the comments :)