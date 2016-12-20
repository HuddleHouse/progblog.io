---
title: 'Angular 2 Tutorial: Create an App with Angular CLI and Angular 2 Material Design'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'In this tutorial we will be setting up a new Angular 2 project using the Angular 2 CLI (Command Line Interface). Then we will add Material Design for Angular 2 to it and customize.'
tags:
  - Angular 2
  - Node.js
  - NPM
  - Angular CLI
date: 2016-10-17 15:51:16
---
## Introduction

In this tutorial we will be setting up a new [Angular 2](https://angular.io/) project using the [Angular 2 CLI (Command Line Interface)](https://github.com/angular/angular-cli). Then we will add [Material Design for Angular 2](https://github.com/angular/material2) to our project and customize.

First, we must install the Angular 2 CLI so that we can use it to generate a new Angular 2 project. Using npm we will install and configure Material Design for Angular 2 into our project. Lastly, we will add a custom SCSS style sheet and get Angular CLI to load it in for us.

The code for this project is hosted on my [Github](https://github.com/HuddleHouse/angular-2-cli-material). Feel free to fork and use freely!

The End Result:

![In this Angular 2 Tutorial we build a demo app that will look like this in the end!](final.png)

## Overview

The Angular CLI makes it easy to get up and running with a project that is set up using the best practices. 

As a developer who has spent a lot of time using [Symfony](https://symfony.com/) and [Composer](https://getcomposer.org/), I feel right at home using the Angular 2 CLI. The CLI also has commands to generate components, routes, services, pipes and prepare your application for production.

Material Design for Angular 2 is built using the Material Design specs built on top of Angular 2.

> Note: Material Design for Angular 2 is currently in Alpha, but most of the desired functionality is there and it is currently in developement. That being said you will have to upgrade through npm to use new features as they become available.

## Prerequisites

You will need [NodeJS](https://nodejs.org/en/download/) and [npm](https://www.npmjs.com/) installed. NodeJS is an event-driven I/O server-side JavaScript environment. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.

Installation should be as simple as installing NodeJS and npm comes with with.

> IMPORTANT: You must have NodeJS 4 or higher AND npm 3 or higher. When downloading NodeJS note that the LTS version of Node comes with npm 2.15.9. If you are about to install Node, get the Current/Latest Features download. You can also update npm from the command line `npm install -g npm`. 

## Install Angular 2 CLI & Create Project

To install the Angular 2 CLI run this command: 

```php
npm install -g angular-cli
```

> The first time running this command I had trouble getting it to work. I had a package installed globally that conflicted. If you have issues, post your error message in the comments and I'll try to help!

We now can use the `ng` command in a terminal window. To create a new project we simply type:

> Note: Creating the new project can take a few minutes on a slower computer.

```php
ng new my-project-name
```

![Terminal window after creating our new Angular 2 Material Design Demo project.](created-new.png)

A new project will be made inside of the folder `my-project-name`.

> Note: In Angular 2 it is convention to spell our file names with [dash case](https://angular.io/docs/ts/latest/guide/glossary.html#!#dash-case) so there are no worries about case sensitity on the server, across dev environments and in our Version Control.

To start the dev server and see the live product run these commands:

```php
cd my-project-name
ng serve
```
> My favorite feature of using `ng serve` is that it watches your files for changes. Upon noticing them it automatically reloads the browser for you!

Open your browser and point it to `http://localhost:4200`. You should see:

![Our new Angular 2 Material Design Demo app.](it-works.png)

## Install Material Design for Angular 2 through npm

To add Material Design for Angular 2 to our project run the command:

```php
npm install --save @angular/material
```

> Note: `--save` adds this to our `package.json` file for us. 

If you run into errors running the above command make sure you have the correct version of NodeJS and npm installed.

Next in our `src/app/app.module.ts` file we need to import the `MaterialModule` at the top. Then we need to register it for root in the imports section. After doing this my file looks like this:

```php
// src/app/app.module.ts

import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';
import { MaterialModule } from '@angular/material';

import { AppComponent } from './app.component';

@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    MaterialModule.forRoot()
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Check that everything is still working in the broswer with no errors.

## Add a Custom Stylesheet

Add this to the header of your `index.html` file to make Material Icons available.

```php
// app/index.html

<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

Create the file `src/assets/custom-theme.scss`. In this file we can write Angular 2 Material that will override the default styles. 

All available colors are found in `node_modules/@angular/material/core/theming/_palette.scss`. Use these ase a reference to choose the colors for your app.

```php
// src/assets/custom-theme.scss

@import '~@angular/material/core/theming/all-theme';
@include md-core();

$primary: md-palette($md-light-blue);
$accent:  md-palette($md-amber, A200, A100, A400);

$theme: md-light-theme($primary, $accent);

@include angular-material-theme($theme);

.m2app-dark {
  $dark-primary: md-palette($md-pink, 700, 500, 900);
  $dark-accent:  md-palette($md-blue-grey, A200, A100, A400);
  $dark-warn:    md-palette($md-deep-orange);

  $dark-theme: md-dark-theme($dark-primary, $dark-accent, $dark-warn);

  @include angular-material-theme($dark-theme);
}
```

> Note: Do not change the contents of any file in `node_modules` they will get over written on the next npm install.

Angular CLI has a built in SCSS compiler. All we have to do is let it know that the new SCSS file is there. We do this by editing our `angular-cli.json` file. We want to edit the `styles` section to look like this:

```php
// package.json

"styles": [
        "styles.css",
     	 "./assets/custom-theme.scss"
      ],
```

## Add HTML and add CSS

If we look at the Github for [Material Design for Angular 2](https://github.com/angular/material2) there is a section in the README that shows all the features and thier current developement status. 

To add a sample of available options add this to `app.component.html`:

```php
// src/app/app.component.html

<md-sidenav-layout [class.m2app-dark]="isDarkTheme">
  <md-sidenav #sidenav mode="side" class="app-sidenav">
    Sidenav
  </md-sidenav>
  
  <md-toolbar color="primary">
    <button class="app-icon-button" (click)="sidenav.toggle()">
      <i class="material-icons app-toolbar-menu">menu</i>
    </button>
    Angular 2 Material2 Example App
    <span class="app-toolbar-filler"></span>
    <button md-button (click)="isDarkTheme = !isDarkTheme">TOGGLE DARK THEME</button>
  </md-toolbar>
  
  <div class="app-content">
    <md-card>
      <button md-button>FLAT</button>
      <button md-raised-button md-tooltip="This is a tooltip!">RAISED</button>
      <button md-raised-button color="primary">PRIMARY RAISED</button>
      <button md-raised-button color="accent">ACCENT RAISED</button>
    </md-card>
  
    <md-card>
      <md-checkbox>Unchecked</md-checkbox>
      <md-checkbox [checked]="true">Checked</md-checkbox>
      <md-checkbox [indeterminate]="true">Indeterminate</md-checkbox>
      <md-checkbox [disabled]="true">Disabled</md-checkbox>
    </md-card>
  
    <md-card>
      <md-radio-button name="symbol">Alpha</md-radio-button>
      <md-radio-button name="symbol">Beta</md-radio-button>
      <md-radio-button name="symbol" disabled>Gamma</md-radio-button>
    </md-card>
  
    <md-card class="app-input-section">
      <md-input placeholder="First name"></md-input>
      <md-input #nickname placeholder="Nickname" maxlength="50">
        <md-hint align="end">
          {{nickname.characterCount}} / 50
        </md-hint>
      </md-input>
      <md-input>
        <md-placeholder>
          <i class="material-icons app-input-icon">android</i> Favorite phone
        </md-placeholder>
      </md-input>
      <md-input placeholder="Motorcycle model">
        <span md-prefix>
          <i class="material-icons app-input-icon">motorcycle</i>
          &nbsp;
        </span>
      </md-input>
    </md-card>
  
    <md-card>
      <md-spinner class="app-spinner"></md-spinner>
      <md-spinner color="accent" class="app-spinner"></md-spinner>
    </md-card>
  
    <md-card>
      <label>
        Indeterminate progress-bar
        <md-progress-bar
          class="app-progress"
          mode="indeterminate"
          aria-label="Indeterminate progress-bar example"></md-progress-bar>
      </label>
      <label>
        Determinate progress bar - {{progress}}%
        <md-progress-bar
          class="app-progress"
          color="accent"
          mode="determinate"
          [value]="progress"
          aria-label="Determinate progress-bar example"></md-progress-bar>
      </label>
    </md-card>
  
    <md-card>
      <md-tab-group>
        <md-tab>
          <template md-tab-label>
            Earth
          </template>
          <template md-tab-content>
            <p>EARTH</p>
          </template>
        </md-tab>
        <md-tab>
          <template md-tab-label>
            Fire
          </template>
          <template md-tab-content>
            <p>FIRE</p>
          </template>
        </md-tab>
      </md-tab-group>
    </md-card>
  </div>
</md-sidenav-layout>

<span class="app-action" [class.m2app-dark]="isDarkTheme">
  <button md-fab><md-icon>check circle</md-icon></button>
</span>
```

Lastly, update `app.component.css` and `app/styles.css`.

> Note: The styles that are in `app/styles.css` will affect the whole app. The styles in `app.component.css` will only affect that one component.

Here is `app.component.css`:

```php
// src/app/app.component.css

.app-toolbar-filler {

  flex: 1 1 auto;
}

md-sidenav-layout.m2app-dark {
  background: black;
}

.app-content {
  padding: 20px;
}

.app-content md-card {
  margin: 20px;
}

.app-sidenav {
  padding: 10px;
  min-width: 100px;
}

.app-content md-checkbox {
  margin: 10px;
}

body {
  margin: 0 !important;
}
.app-toolbar-menu {
  padding: 0 14px 0 14px;
  color: white;
}

.app-icon-button {
  box-shadow: none;
  user-select: none;
  background: none;
  border: none;
  cursor: pointer;
  filter: none;
  font-weight: normal;
  height: auto;
  line-height: inherit;
  margin: 0;
  min-width: 0;
  padding: 0;
  text-align: left;
  text-decoration: none;
}

.app-action {
  display: inline-block;
  position: fixed;
  bottom: 20px;
  right: 20px;
}

.app-spinner {
  height: 30px;
  width: 30px;
  display: inline-block;
}

.app-input-icon {
  font-size: 16px;
}

.app-list {
  border: 1px solid rgba(0,0,0,0.12);
  width: 350px;
  margin: 20px;
}

.app-progress {
  margin: 20px;
}
```

Here is `app/styles.css`:

```php
// app/styles.css

body {
  margin: 0;
}

```

Once you have added all of that you should be able to run `ng serve` and see the final product! You can also press the `TOGGLE DARK THEME` button and see the dark theme.

![This is the final result of our Angular 2 Material Design demo app!](final-dark.png)

## Conclusion

Now we have a solid start to build an Angular 2 application, using Angular 2 Material Design. Angular 2 Material Design may still be in Alpha, but it will grow quick. Soon it will be the same standard just like there was Angular 2 Material

Personally I would rearrange the filesystem so the project won't get out of control as it grows.

If you ran into any hiccups or have any questions, please let me know! I'm happy to help :)