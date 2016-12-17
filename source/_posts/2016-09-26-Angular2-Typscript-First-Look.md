---
title: 'Angular2 & Typscript: First Look'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: https://huddlehouse.io
tags:
- Angular2
- Typescript
date: 2016-09-26 18:11:10
---

<a href="http://angular.io/" target="_blank">AngularJS</a> is used by developers around the globe. Brad Green, Google's engineering director, said that

>1.3 million developers use AngularJS and 300 thousand are already using the recently released Angular 2.

When I first began using Angular it changed the way I programmed<!-- more --> on the front end. Angular extends HTML for your applications giving you clean ways to manipulate data in the browser.

One way that Angular revolutionized my programming was with ngModel. As a programmer that grew up learning with the Model View Controller (<a href="https://en.wikipedia.org/wiki/Model-view-controller" target="_blank">MVC</a>) methodologies I felt at home. You were now able to bind the Model from JS to HTML elements and update the screen without reloading the page, all with minimal effort.

I used to cry (not really, but almost!) when I had to repeatedly make AJAX requests when building a web app. With Angular the process is extremely simple (as long as your Model structure and JSON data are the same)!


## Features

#### Component-based Framework

![Component Based Framework](webcomponents.jpg)

Angular2 was built to be a component-based framework, instead of being a MVC framework. An Angular2 application is the scaffolding that holds multiple components. The boring college definition (from [Wikipedia](https://en.wikipedia.org/wiki/Component-based_software_engineering#Software_component)) for this is  is:

>An individual software component is a software package, a web service, a web resource, or a module that encapsulates a set of related functions (or data).


The real world explanation:

>WE WANT SINGLE PAGE APPLICATIONS!

I want to load a single HTML page that dynamically reacts to the users input, without wanting to pull my hair out developing it. Components are crucial to achieving this. At their core components are simply reusable bits of logic. They can be combined and reused independently.

#### Typsescript


As a Full Stack developer who has been doing mostly Symfony apps for the last few years, this is what I am most excited about! It gives you the benefit of Object Oriented Programming ([OOP](http://searchsoa.techtarget.com/definition/object-oriented-programming)). I never thought I would see that!


Now you can fully leaverage the debugging ability that a good editor like [WebStorm](https://www.jetbrains.com/webstorm/) offers. As a Symfony developer I use PhpStorm ([PS](https://www.jetbrains.com/phpstorm/)) as my main editor, because PHP is an OOP language (or it has 'types') it does most of the typing for me. It will immediately tell me that the variable is supposed to be and a lot more. That is a huge feature to be excited about!

## Conclusion

Angular2 offers a more simple way to acheive things that we have come to expect in modern single page apps. It leaverages the benefits of Typescript, making it a more powerful complete framework. I have not yet got to do the tutorials on Angular2's website, but I can't wait to dive in!

If you have questions or anything that you would like to add, please do so in the comments!