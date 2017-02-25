---
title: 'Symfony 3.1: Up and Running in 60 seconds'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
excerpt: "As a full stack developer I love setting up projects from the command line. Doing this can be cumbersome and adds time before development can start. At the time of writing this the latest version of Symfony is 3.1.4"
tags:
  - Symfony
  - PHPStorm
date: 2016-10-02 12:58:41
author_link: https://huddlehouse.io
---

## Introduction

As a full stack developer I love setting up projects from the command line. Doing this can be cumbersome and adds time before development can start. 

> At the time of writing this the latest version of Symfony is 3.1.4

With a good text editor like [PHPStorm](https://www.jetbrains.com/phpstorm/) all of the set up is done for you! I know PHPStorm costs money, but it really is worth it to have a good editor. 

> If you are a student you can use PHPStorm (and all JetBrains products) for free [here](https://blog.jetbrains.com/blog/2015/12/11/jetbrains-student-program-how-to-renew-graduation-discount/) 

Open PHPStorm and choose to Create a New Project. On the left select a Symfony project and rename the location to the name you want to call the project and click create. PHPStorm downloads Symfony and sets everything up in that folder.

![Select a SYmfony project.](pic.png)

Once that has finished open the the Terminal in PHPStorm and run this command to start the development servers:

<button class="right copy btn" data-clipboard-target="#phpstorm"><i class="fa fa-clipboard"></i></button>
<div id='phpstorm'>
```php
php bin/console server:run
```
</div>

![Start the servers](command.png)
<br>
Now open up your browser and go to `http://127.0.0.1:8000/`
<br>

![Success.](Welcome_.png)

<br>

Now you are ready to start building your Symfony 3.1 application! 

> Note: At this point you don't have a db set up. The next step would be to set that up, by creating a db and then adding the credentials to `app/config/parameters.yml`

![Success.](parameters.png)

<br>
If you have any questions please ask us in the comments :)