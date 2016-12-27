---
title: 'Angular 2 Tutorial Part 1: Create Firebase 3 CRUD app with Angular CLI'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'In this 3 part tutorial series we will build a simple chat app. It will require the user to login and then they can chat with anyone else in the room.'
tags:
  - Node.js
  - Angular 2
  - Firebase
  - Angular CLI
  - Realtime Database
  - Authentication
date: 2016-12-27 17:13:27
---
## Introduction
In this 3 part tutorial series we will build a simple chat app. It will require the user to login and then they can chat with anyone else in the room. We will use [Firebase 3](https://firebase.google.com) as our db and as our authentication. We will use a [NPM](https://www.npmjs.com/) package called [AngularFire 2](https://www.npmjs.com/package/angularfire2), which is the official library for Firebase and Angular 2.

**Part 1:** Installing necessary components. Generating a blank app through Angular CLI. Add Bootstrap for styling. Create Firebase account. Install and setup Firebase through NPM.

If you would like to use the full code for **part 1** it can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-1). 

**Part 2:** Add user login through their Google account. Create the Login Page and Home Page Components. Add asychronous monitoring of the logged in status. Redirect to the Home Page when logged in and to the Login Page when the user logs out.

If you would like to use the full code for **part 2** it can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-2). 

**Part 3:** **COMING SOON** We will build a simple chat app that stores messages in a Firebase Realtime Database. Users will be able to edit or delete messages that they sent to the chat. 

CRUD stands for Create, Read, Update and Delete. Firebase take a lot of the complexity out of a web app, which makes development a lot easier!

### About Firebase

Firebase is a swanky service that is offered by Google that allows the developer to not have to worry about many mundane tasks that slow down development. The Firebase services that we will be using is the Realtime Database, which syncs across all devices in milliseconds. We will also be using their authentication API's and I will also use their hosting to host the final version of the app.

I have been using Firebase for a few months now and I am a huge fan of it. It greatly speeds up development and can grow as big as your app does. All without having to have the knowledge of a System Administrator (or hiring one) to maintain the growth of your app.

> Note: If you clone the repository follow the prerequisites section and then run `npm install` in the project folder.

## Prerequisites

You will need [NodeJS](https://nodejs.org/en/download/) and [npm](https://www.npmjs.com/) installed. NodeJS is an event-driven I/O server-side JavaScript environment. Node.js' package ecosystem, npm, is the largest ecosystem of open source libraries in the world.

Installation should be as simple as installing NodeJS and npm comes with with.

> IMPORTANT: You must have NodeJS 4 or higher AND npm 3 or higher. When downloading NodeJS note that the LTS version of Node comes with npm 2.15.9. If you are about to install Node, get the Current/Latest Features download. You can also update npm from the command line `npm install -g npm`. 

Once you have npm installed from the command line you can install Angular CLI by typing:

```
#Note: the -g flag installs this globally on your computer
npm install -g angular-cli
```

For AngularFire2 to work we will also need to install `typings` and `typescript`.

```
npm install -g typings
npm install -g typescript
```
> Note: You might have to run the above commands as an administrator ex: `sudo npm install -g angular-cli`.

## Using Angular CLI to create a new project

Creating a blank app with the Angular CLI is easy. In a terminal type in this command:

```
ng new firebase-crud
```

> Note: `firebase-crud` is the name of the folder the project will be created in. Name this whatever makes sense to you. 

![The output after creating a new app using Angular CLI](new-app.png)

To view that our app is working lets display it.

```
cd firebase-crud
ng serve
```
Our app should now be visible from a browser on the default url `http://localhost:4200`

> Note: To change the port that is used you can type `ng serve -p 4000`

![Now we can see that our blank Angular 2 app is working in the browser](FirebaseCrud.png)

## Add Bootstrap to the app for styling

If we open our app in a text editor, I use [Webstorm](https://www.jetbrains.com/webstorm/) which is an excellent editor IMO. The first thing we need to do is add the Bootstrap cdn to our app. We can do this by opening up `app/index.html` and edit it so that it looks like this:

```
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>FirebaseCrud</title>
  <base href="/">
  <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
  <link href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" rel="stylesheet">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="icon" type="image/x-icon" href="favicon.ico">
</head>
<body>
  <app-root>Loading...</app-root>
</body>
</html>
```
> Note: Bootstrap has an Angular 2 extension called [ng-bootstrap](https://ng-bootstrap.github.io/#/home). It adds Angular 2 components, but for our purposes we do not need them. 

Now lets add a navbar in `src/app/app.component.html` to make our app feel a little more polished.

```
<nav class="navbar navbar-default">
  <div class="container-fluid">
    <!-- Brand and toggle get grouped for better mobile display -->
    <div class="navbar-header">
      <a class="navbar-brand" href="#">Firebase Crud</a>
    </div>
  </div>
</nav>
```

## Install AngularFire2

### Create Project in the Firebase Console

The next thing that we need to do is add AngularFire2 to our project. Before we are able to do this we need to login to the [Firebase Console](https://console.firebase.google.com/). You can log in using any Google account that you already have. Once you are logged in we need to create a new Project. 

![Creating a new project in the Firebase Console](Firebase_Console.png)

### Install AngularFire2 through npm

Now that we have a Firebase Project created run the following command from inside of your project:

```
npm install firebase angularfire2 --save
```

### Setup @NgModule

Open `src/app/app.module.ts`, inject the Firebase providers and add your app specific Firebase configuration.

```
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { AngularFireModule } from 'angularfire2';

// Must export the config
export const firebaseConfig = {
  apiKey: '<your-key>',
  authDomain: '<your-project-authdomain>',
  databaseURL: '<your-database-URL>',
  storageBucket: '<your-storage-bucket>',
  messagingSenderId: '<your-messaging-sender-id>'
};

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig)
  ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ]
})
export class AppModule {}
```
Your Firebase credentials can be found in the Firebase console. When you log in, click `Add Firebase to your Web App` and your credentials will be there. 

![In the Firebase console we can get our Credentials by clicking add Firebase to your web app](addToWebApp.png)

## Conclusion

Now we are set up to use Firebase in our project. In the next tutorial we will add user authentication. If you have any questions please don't hesitate to ask me! :)
