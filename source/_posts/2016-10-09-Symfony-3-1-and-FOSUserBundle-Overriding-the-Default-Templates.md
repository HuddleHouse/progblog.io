---
title: 'Symfony 3.1 and FOSUserBundle: Overriding the Default Templates'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'FOSUserBundle handles powerful and otherwise mundane tasks for us with ease. Out of the box it looks pretty bare, but with some quick styling that can be fixed!'
tags:
  - FOSUserBundle
  - Symfony
date: 2016-10-08 16:17:47
---
## Introduction

FOSUserBundle handles powerful and otherwise mundane tasks for us with ease. Out of the box it looks pretty bare, but with some quick styling that can be fixed!

Before:

![FOSUserBundle login form after styling](before.png)

After:

![FOSUserBundle login form after styling](after.png)

## Prerequisites

This tutorial requires that you have a working Symfony dev environment. If you need help setting one up start with our other [tutorial](https://progblog.io/Symfony-3-1-Up-and-Running-in-60-Seconds/) that shows you how to get up and running with ease.

You will also need to have FOSUserBundle added to the project. We have another [tutorial](https://progblog.io/Symfony-3-1-and-FOSUserBundle-Setting-Up/) that will help you get that installed.
 
> The code for these tutorials is on my [Github](https://github.com/HuddleHouse/symfony-demo/branches) you can fork it to your own Github. Each branch is a different tutorial. The completed code for this tutorial is found on the `override-templates` branch. The branch `initial-setup` would put you right before this tutorial. This will require a more advanced knowledge of Git, but please ask me if you run into issues! :)
 
## Overview 

There are two ways that you can overide the templates of a bundle.

1. In the `app/Resources` directory define a new template
2. Create a new bundle that is a child of `FOSUserBundle` 

## Define new Template in `app/Resources`

This is the easiest way to overide a bundles template, but it is also the most limiting way. Personally I use a child theme. It is usful to know the other options available. 

To override the file `vender/friendsofsymfony/user-bundle/Resources/views/Security/login.html.twig` you would place your new template at `app/Resources/FOSUserBundle/views/Security/login.html.twig`

> Using this method will only allow you to overide the template. You cannot change the controller. If you want to be able to have control over the controller also, then you must make a child bundle.

## Child Bundle
 
This way is slightly more involved, but offers full control over FOSUserBundle. The first thing that we want to do is make `AppBundle` a child of FOSUserBundle. We can achieve that by editing `src/AppBundle/AppBundle.php` to look like this:

<button class="right copy btn" data-clipboard-target="#fos"><i class="fa fa-clipboard"></i></button>
<div id='fos'>
```
// src/AppBundle/AppBundle.php

namespace AppBundle;

use Symfony\Component\HttpKernel\Bundle\Bundle;

class AppBundle extends Bundle
{
    public function getParent()
    {
        return 'FOSUserBundle';
    }
}
```
</div>

Now that we have made AppBundle a child of FOSUserBundle we can do the same thing as above, but instead of replacing the filesdd in `app/Resources` we will replace them in `src/AppBundle`. We are not limited to only overriding things in the `Resources` folder! We can also override the controllers or anything else. 

> As far as I have tried with FOSUserBundle you can copy any file they have in the `vendor` folder and add it to AppBundle by matching the file structure and changing the namespace. Then you can customize as you see fit. Obviously there are some files in FOSUserBundle that should not be overwritten. 

### Create User

Before we can login we must have a user to be able to login with. FOSUserBundle comes with command line arguments that can achieve this. To see all commands available open a terminal window and type `php bin/console`. This will show us all available commands, and we can locate the FOSUserBundle commands:

![Listing the avaliable FOSUserBundle commands](fos.png)

Running the command below will prompt you with a few questions to answer in order to create the new user.

<button class="right copy btn" data-clipboard-target="#will"><i class="fa fa-clipboard"></i></button>
<div id='will'>
```
php bin/console fos:user:create
```
</div>

![Creating the user from the FOSUserBundle Command line](created_user.png)

### Create Base Template

In Symfony we can create a base template which every page can use. On this template it can hold the scaffolding each page requires and also include all the required css and js files. To do this edit the file `app/Resources/views/base.html.twig` to say:

<button class="right copy btn" data-clipboard-target="#page"><i class="fa fa-clipboard"></i></button>
<div id='page'>
```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
        <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">

</head>
<body>
<style>
    @import url(http://fonts.googleapis.com/css?family=Roboto);

    /****** LOGIN ******/
    .login-container {
        padding: 30px;
        max-width: 350px;
        width: 100% !important;
        background-color: #F7F7F7;
        margin: 100px auto;
        border-radius: 2px;
        box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
        overflow: hidden;
        font-family: roboto;
    }

    .login-container h1 {
        text-align: center;
        font-size: 1.8em;
        font-family: roboto;
    }

    .login-container input[type=submit] {
        width: 100%;
        display: block;
        margin-bottom: 10px;
        position: relative;
    }

    .login-container input[type=text], input[type=password] {
        height: 44px;
        font-size: 16px;
        width: 100%;
        margin-bottom: 10px;
        -webkit-appearance: none;
        background: #fff;
        border: 1px solid #d9d9d9;
        border-top: 1px solid #c0c0c0;
        /* border-radius: 2px; */
        padding: 0 8px;
        box-sizing: border-box;
        -moz-box-sizing: border-box;
    }

    .login-container input[type=text]:hover, input[type=password]:hover {
        border: 1px solid #b9b9b9;
        border-top: 1px solid #a0a0a0;
        -moz-box-shadow: inset 0 1px 2px rgba(0,0,0,0.1);
        -webkit-box-shadow: inset 0 1px 2px rgba(0,0,0,0.1);
        box-shadow: inset 0 1px 2px rgba(0,0,0,0.1);
    }

    .login-submit {
        /* border: 1px solid #3079ed; */
        border: 0px;
        color: #fff;
        text-shadow: 0 1px rgba(0,0,0,0.1);
        background-color: #4d90fe;
        padding: 17px 0px;
        font-family: roboto;
        font-size: 14px;
        /* background-image: -webkit-gradient(linear, 0 0, 0 100%,   from(#4d90fe), to(#4787ed)); */
    }

    .login-submit:hover {
        /* border: 1px solid #2f5bb7; */
        border: 0px;
        text-shadow: 0 1px rgba(0,0,0,0.3);
        background-color: #357ae8;
        /* background-image: -webkit-gradient(linear, 0 0, 0 100%,   from(#4d90fe), to(#357ae8)); */
    }

    .login-container a {
        text-decoration: none;
        color: #666;
        font-weight: 400;
        text-align: center;
        display: inline-block;
        opacity: 0.6;
        transition: opacity ease 0.5s;
    }

    .login-help{
        font-size: 12px;
    }
</style>

{% for type, messages in app.session.flashBag.all %}
    {% for message in messages %}
        <div class="{{ type }}">
            {{ message|trans({}, 'FOSUserBundle') }}
        </div>
    {% endfor %}
{% endfor %}

{% block fos_user_content %}{% endblock %}

<script   src="https://code.jquery.com/jquery-3.1.1.min.js"   integrity="sha256-hVVnYaiADRTO2PzUGmuLJr8BLUSjGIZsDYGmIJLv2b8="   crossorigin="anonymous"></script>
<script type="text/javascript" src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>

{% block javascripts %}{% endblock %}

</body>
</html>
```
</div>

This file is very basic. I including some custom styling for the login page. I also included [Bootstrap](http://getbootstrap.com/) and [jQuery](https://jquery.com/). Bootstrap is a responsive HTML framework that comes with a lot of core components.

Notice `block fos_user_content`. That is the section where each page will import it's own html.

The `block javascripts` section is so that custom js can be added on each page and it will load in the correct order.

### Updating the Styling on the Login Form

Now we are ready to make the login form look the way that we want. First I open the `vendor` folder and go into `friendsofsymfony` to locate to login form. I need to know the file structure to match it in AppBundle.

In AppBundle make the folders `Resources/views/Security`. Copy both `login.html.twig` and `login_content.html.twig` from FOSUserBundle and put them in the Security folder that we just made. 

> Tip: If you are using [PHPStorm](https://www.jetbrains.com/phpstorm/) you can highlight the files and right click them and hit copy. You can then click the desired folder and paste the files there!

I personally do not like how FOSUserBundle has a `login.html.twig` and `login_content.html.twig`. So I took the `trans_default_domain 'FOSUserBundle'` from it and put it into `login.html.twig`. Then I moved the form into the `block fos_user_content` section and then deleted `login_content.html.twig`.

Now we should still see the same login form as before. But now lets change the contents of `login.html.twig` to make this login form look awesome:

<button class="right copy btn" data-clipboard-target="#lets"><i class="fa fa-clipboard"></i></button>
<div id='lets'>
```
{% trans_default_domain 'FOSUserBundle' %}
{% extends "::base.html.twig" %}

{% block fos_user_content %}
    {% if error %}
        <div>{{ error.messageKey|trans(error.messageData, 'security') }}</div>
    {% endif %}

    <form action="{{ path("fos_user_security_check") }}" method="post" class="form-signin">
        <input type="hidden" name="_csrf_token" value="{{ csrf_token }}" />
        <div class="login-container">
            <h1>Login to Your Account</h1><br>
            <form>
                <label for="username">{{ 'security.login.username'|trans }}</label>
                <input type="text" id="username" name="_username" value="{{ last_username }}" required="required" />
                <label for="password">{{ 'security.login.password'|trans }}</label>
                <input type="password" id="password" name="_password" required="required" />
                <input type="checkbox" id="remember_me" name="_remember_me" value="on" />
                <label for="remember_me">{{ 'security.login.remember_me'|trans }}</label>
                <input type="submit" class="login login-submit" id="_submit" name="_submit" value="{{ 'security.login.submit'|trans }}" />
            </form>
            <div class="login-help">
                <a href="{{ path('fos_user_registration_register') }}">Register</a> - <a href="{{ path('fos_user_resetting_request') }}">Forgot Password</a>
            </div>
        </div>
    </form>
{% endblock fos_user_content %}
```
</div>

This will display the formatted login form!

> In twig you can get the path to other routes by calling `path('fos_user_registration_register')`

![Our formatted FOSUserBundle login form!](success.png)

Now lets test the login form and see if we can login. 

> Note: When developing in Symfony it's important to add `/app_dev.php` to every url. If it is not added to the url then Symfony treats it as production and Caches a lot. Thus you don't see your changes without clearing the cache. It also gives you a handy toolbar at the bottom.

By entering the credentials of the user that we just made and submitting it we are taken to a blank page. That is a good sign that it worked! We can see this by hovering over the bottom toolbar (if you don't have the bottom toolbar add `/app_dev.php` to the url.

![We can see that we are logged in through our FOSUserBUndle Login form](logged-in.png)

To logout add `/logout` to the url. This will take us to a blank screen again, but by hovering over the person on the bottom toolbar we see that we are no longer logged in.

![We can see that we are logged out](logged-out.png)


### Redirect to Login Screen

It is a personal preference of mine that when not logged in it should redirect us to the login screen. So lets make it do that! We can do that by editing our `app/config/security.yml` by adding this line under `access_control`.

<button class="right copy btn" data-clipboard-target="#pref"><i class="fa fa-clipboard"></i></button>
<div id='pref'>
```
- { path: ^/, role: IS_AUTHENTICATED_FULLY }
```
</div>

This line says that anyone at the root url has to be fully authenticated. Since we have our login provider set to FOSUserBundle it automatically redirects to the login page.

## Conclusion

We have now seen how to fully override a FOSUserBundle template and make it our own. We also saw how to create a user and adjust the security settings to fit our needs. 

In this application we did not need to override the controller. Doing so is as easy as copying the one in the `vendor` folder and moving it to AppBundle in the same spot (you also need to change the namespace in the copied file). Once you have copied it, Symfony will automatically use yours over the one in the `vendor` folder.

The code for this projec is on [Github](https://github.com/HuddleHouse/symfony-demo/tree/override-templates). If you have any questions please ask me in the comments :) 

We are looking for more ideas on tutorials so if there is anything you would like us to do a tutorial on let us know! The material does not have to be Symfony related.