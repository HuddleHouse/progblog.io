---
title: How to Globally install Composer on OS X
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'Composer is a PHP dependency management tool. It is not necessarily a package manager in the traditional sense, like NPM or Yum. It does handle packages but it does it on a per-project basis.
'
tags:
  - Composer
date: 2016-10-30 12:49:00
---
## Introduction

[Composer](https://getcomposer.org) is a PHP dependency management tool. It is not necessarily a package manager in the traditional sense, like [NPM](https://www.npmjs.com/) or [Yum](https://fedoraproject.org/wiki/Yum). It does handle packages but it does it on a per-project basis.

This can be very useful when setting up a project or adding dependencies to a project.

Throughout my programming career I have Google'd `How to install composer globally` and always search for the way to do it in one terminal command. Then the next time I need it that site is no longer there. So this post is for me so I no longer have to look for it!

## OS X 10.11
<button class="right copy btn" data-clipboard-target="#linux"><i class="fa fa-clipboard"></i></button>
<div id='linux'>
```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer
```
</div>

Type `composer` into the terminal and you should see this:

![Successfully installed composer](composer.png)

## OS X 10.10

<button class="right copy btn" data-clipboard-target="#linux2"><i class="fa fa-clipboard"></i></button>
<div id='linux2'>
```
curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/bin --filename=composer  
```
</div>

Now you should be able to use composer globally by typing `composer` in the terminal.
