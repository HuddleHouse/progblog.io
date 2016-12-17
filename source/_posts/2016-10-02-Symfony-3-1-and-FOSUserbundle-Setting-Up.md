---
title: 'Symfony 3.1 and FOSUserBundle: Setting Up'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
excerpt: "FOSUserBundle adds a flexible database backed user system. It includes basic ways to create users, login/logout, password reset and registration support, with an optional confirmation email. It is easily expandable to fit any needs."
tags:
  - Symfony
  - FOSUserBundle
date: 2016-10-03 14:52:58
author_link: https://huddlehouse.io
---
## Introduction

FOSUserBundle adds a flexible database backed user system. It includes basic ways to create users, login/logout, password reset and registration support, with an optional confirmation email. It is easily expandable to fit any needs.

There are many reasons why I have always used FOSUserBundle instead of writing the code for this functionality instead. In college there was a principle that we learned called [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself), Don't Repeat Yourself! There have been loads of people use FOSUserBundle and it has been unit tested. It would take me a long time to write the same functionality, and hopefully it would work as well. 

For this reason I learned to use FOSUserBundle a few years ago and I still use it in multiple projects today. 

The code for this tutorial is found on my [Github](https://github.com/HuddleHouse/symfony-demo) under the branch `initial-setup`. Feel free to fork it and use it!

## Prerequisites

This tutorial requires you have a working Symfony project. If you need help getting set up we have a quick tutorial that shows you how to get [up and running with Symfony](https://progblog.io/2016/10/02/Symfony-3-1-Up-and-Running-in-60-Seconds/).

This tutorial also assumes that you have [Composer](https://getcomposer.org/) installed globally. [Here](https://getcomposer.org/download/) is a reference for how to do that.

I will also be using a SQL database, but you could use a different type if you wanted.

## Installation


Tell composer to require the bundle:

```php
composer require friendsofsymfony/user-bundle "~2.0@dev"
```

Open up `app/AppKernal.php` and make sure to add the bundle to the `$bundles` array.

```php
<?php
// app/AppKernel.php

public function registerBundles()
{
    $bundles = array(
        // ...
        new FOS\UserBundle\FOSUserBundle(),
        // ...
    );
}
```
### Configure `app/security.yml`

Now lets configure the `app/security.yml` file. Below is a minimal example of the configuration necessary to use the FOSUserBundle in your application:

```php
# app/config/security.yml
security:
    encoders:
        FOS\UserBundle\Model\UserInterface: bcrypt

    role_hierarchy:
        ROLE_ADMIN:       ROLE_USER
        ROLE_SUPER_ADMIN: ROLE_ADMIN

    providers:
        fos_userbundle:
            id: fos_user.user_provider.username

    firewalls:
        main:
            pattern: ^/
            form_login:
                provider: fos_userbundle
                csrf_token_generator: security.csrf.token_manager
                # if you are using Symfony < 2.8, use the following config instead:
                # csrf_provider: form.csrf_provider

            logout:       true
            anonymous:    true

    access_control:
        - { path: ^/login$, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/resetting, role: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/admin/, role: ROLE_ADMIN }
```

Under the providers section, you are making the bundle's packaged user provider service available via the alias fos_userbundle. 

Under the `firewalls` section we have declared a `main` firewall. This tells the app that anytime a user needs to be authenticated to redirect the user to the login page.

The `access_control` section can be customized to restric access to certain pages or parts of the app.

### Create the User Class

Navigate to `src/AppBundle` and create a folder called Entity. In that folder create a file called `User.php` and put this in it. 

```php
<?php
// src/AppBundle/Entity/User.php

namespace AppBundle\Entity;

use FOS\UserBundle\Model\User as BaseUser;
use Doctrine\ORM\Mapping as ORM;

/**
 * @ORM\Entity
 * @ORM\Table(name="fos_user")
 */
class User extends BaseUser
{
    /**
     * @ORM\Id
     * @ORM\Column(type="integer")
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    protected $id;

    public function __construct()
    {
        parent::__construct();
        // your own logic
    }
}
```
### Configure FOSUserBundle

Now we need to configure the bundle to work with the needs of our application.

Add this to your `app/config/config.yml`

```php
# app/config/config.yml
fos_user:
    db_driver: orm # other valid values are 'mongodb', 'couchdb' and 'propel'
    firewall_name: main
    user_class: AppBundle\Entity\User
```

We tell FOSUserBundle that we will be using the Doctrine ORM for our db driver, the 	`main` firewall we just made and the `User` class we just made.

### Import FOSUserBundle routing files

To get all of the pages that FOSUserBundle comes with (login, registration etc.) we must tell our application where those routes are. We do this by editing the `app/config/routing.yml` file and add this to the bottom:

```php
# app/config/routing.yml
fos_user:
    resource: "@FOSUserBundle/Resources/config/routing/all.xml"
```

Symfony offers a lot of functionality through the command line. One of the commands is:

```php
php bin/console debug:router
```
This will list all available routes. You should see all the routes we just imported show up.

![List Routes](debug-route.png)

> Note: To see all available commands type `php bin/console` and it will list the commands.

### Update our Database

I have my project set up on [MAMP](https://www.mamp.info/en/) for developement. When MAMP is running I use [Sequel Pro](https://www.sequelpro.com/) to access MySQL and create my db.

![Sequal Pro](sqlpro.png)

Then you have to update your `app/parameters.yml` with the db credentials.

![Parameters.yml](parameters.png)

Once that is set up we can open the terminal to the root directory of our project and run this command to update the db schema:

```php
php bin/console doctrine:schema:update --force
```

![Schema Successfully Update](update.png)

We can also see the updated schema in Sequal Pro

![View updated Schema](schema.png)


Now you can go to the browser and view your login screen!

![We can now view our login screen but there are some errors!](login-before.png)

We probably do not want it to say `security.login` before Username and password fields. They are there because FOSUserBundle was made to able to be used in different languages, through translations. But first those need to be set up. By editing `app/config.yml` and uncommenting this line `translator:      { fallbacks: ["%locale%"] }` this can be achieved.

![We can now view our login screen!](login-after.png)


## Conclusion

This is a very basic unstyled login form. In my next blog post I will show you how to customize all the pages to look the way that we want.

If you have any questions, please ask us in the comments! :)