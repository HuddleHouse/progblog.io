---
title: 4 Reasons to update to Angular 4 from Angular 2
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'Things are constantly changing with programming. Sometimes it can seem daunting to try ot keep up with it all. The Angular dev team has done us a favor with what they are calling an "invisible-makeover".'
tags:
  - Angular
date: 2017-03-26 18:08:04
---
## Introduction

Here are the super easy instructions for [how to upgrade to Angular 4](https://progblog.io/How-to-upgrade-to-Angular-4).

Things are constantly changing with programming. Sometimes it can seem daunting to try ot keep up with it all. The Angular dev team has done us a favor with what they are calling an "invisible-makeover".

It was actually such an easy upgrade that I was inspired to write this blog post about it! I am currently working on three projects that are using Angular 2 and all three upgraded to Angular 4 without any complications. So lets get to it.

## 1. Angular 4 is smaller than Angular 2
The Angular dev team has done a great job minimizing the AOT (ahead-of-time) generated good. This is how they describe it:

> These changes reduce the size of the generated code for your components by around 60%  in most cases. The more complex your templates are, the higher the savings.

## 2. Angular 4 is faster than Angular 2

The Angular dev team did a great job enhancing the speed. It has been noticable when doing development, but I have not been able to test the difference on a live server.

## 3. Angular 4 has enhanced \*ngFor and \*ngIf

They add an if/else style sytax to \*ngIf. When you are making asynchronous calls to say, a Firebase database, you can add a loading screen within the page while that data loads. The example that the Angular Dev team gives is:

<button class="right copy btn" data-clipboard-target="#mac"><i class="fa fa-clipboard"></i></button>
<div id='mac'>
```
<div *ngIf="userList | async as users; else loading">
  <user-profile *ngFor="let user of users; count as count" [user]="user">
  </user-profile>
 <div>{{count}} total users</div>
</div>
<ng-template #loading>Loading...</ng-template>
```
</div>

There is a lot of potential with the ability of an if/else.

## 4. Angular 4 has more meaningful errors

Many times in programming there are not many helpful error messages, your app just won't work. Angular 4 continues to try to change this by generating source maps.

When an error is caused in one of your templates, Angular 4 "generate source maps that give a meaningful context in terms of the original template."



## Conclusion

Most Angular 2 apps will work using Angular 4 without changing anything. It might not be a momumental upgrade, in terms of visible changes, but it offers huge enhancements to Angular 2. It really shows the Angular Teams' commitment to making Angular applicatons smaller and faster, which makes me super happy!