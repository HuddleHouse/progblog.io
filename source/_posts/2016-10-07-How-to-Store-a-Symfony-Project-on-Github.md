---
title: How to Store a Symfony Project on Github
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: "Git is a free version control system for any project, small or large. Git allows multiple people to work on the same code base at the same time using branches. While also saving a versioned copy of your work, in case something were to go wrong."
tags:
  - Symfony
  - Git
  - Github
date: 2016-10-07 15:57:21
---
## Introduction

Git is a free version control system for any project, small or large. Git allows multiple people to work on the same code base at the same time using branches. While also saving a versioned copy of your work, in case something were to go wrong. 

## Prerequisites

You must have Git installed. For a reference on how to do that go [here](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git).

We will also need a Symfony project ready. We have a tutorial that will get you set up quickly, you can view it [here](https://progblog.io/Symfony-3-1-Up-and-Running-in-60-Seconds/).

## Create Repository on Github

Head over to Github.com and make an account if needed. Once you are logged in, click on Repositories on your profile. Then click to add a new one and name it:

![Name Github Repository.](New_Repository.png)

Open a terminal and `cd` to the root directory of your Symfony project.

```php
echo "# Project Name" >> README.md
git init
git add .
git commit -m "initial commit"
git remote add origin git@github.com:HuddleHouse/symfony-demo.git //this will be the URL for the repository we just made
git push -u origin master
```

On success the terminal will output something like this:

![Successful terminal output.](terminal.png)

At that point you should be able to refresh Github and see your code!

My project is [here](https://github.com/HuddleHouse/symfony-demo)


If you have any questions, or suggestions for more tutorials please let me know in the comments! :)