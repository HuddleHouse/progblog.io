---
title: How to upgrade to Angular 4
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'How to upgrade your Angular 2 application to Angular 4. Upgrading to Angular 4 is an easy task and will greatly speed up your Angular 2 app!'
tags:
  - Node.js
date: 2017-03-26 16:25:39
---
## Introduction
This tutorial will show you how to upgrade your Angular 2 web application to use Angular 4.

Upgrading to Angular 4 is an easy task and will greatly speed up your Angular 2 app! I have had no issues or complications when upgrading all our current projects.

It is as easy as updating your projects `composer.json` file. We can do this through the command line using NPM.

Angular is in the process of building an "interactive [Angular Update Guide](https://angular-update-guide.firebaseapp.com/)". After messing around with it some, if you check all the options, the install process is super easy!

## Commands

### On Linux/Mac
<button class="right copy btn" data-clipboard-target="#mac"><i class="fa fa-clipboard"></i></button>
<div id='mac'>
```
npm install @angular/{common,compiler,compiler-cli,core,forms,http,platform-browser,platform-browser-dynamic,platform-server,router,animations}@latest typescript@latest --save
```
</div>

### On Windows
<button class="right copy btn" data-clipboard-target="#windows"><i class="fa fa-clipboard"></i></button>
<div id='windows'>
```
npm install @angular/common@latest @angular/compiler@latest @angular/compiler-cli@latest @angular/core@latest @angular/forms@latest @angular/http@latest @angular/platform-browser@latest @angular/platform-browser-dynamic@latest @angular/platform-server@latest @angular/router@latest @angular/animations@latest typescript@latest --save
```
</div

Once you have done that run `ng serve` as you normally would and BOOM it should work. I know it never is that easy, but it really is this time!

> Note: When i ran the command there were multiple `errors` and `warnings`. A lot of those can be ignored as Angular 4 was designed so that Angular 2 apps will work (of course there are rare exceptions!).

If you rely on Animations refer to [this](http://angularjs.blogspot.com/2017/03/angular-400-now-available.html) article that will show you how to get them working.
