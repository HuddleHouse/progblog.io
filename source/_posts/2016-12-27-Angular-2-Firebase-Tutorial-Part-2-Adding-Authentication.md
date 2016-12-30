---
title: 'Angular 2 Firebase Tutorial Part 2: Adding Authentication'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: "This section will cover adding user authentication using Firebase's Authentication. We will see how easy it is to log someone into our app using their Google Account, all without having to worry about any OAuth protocol."
tags:
   - Node.js
   - Angular 2
   - Firebase
   - Angular CLI
   - Realtime Database
   - Authentication
date: 2016-12-27 17:23:52
---
## Introduction
This tutorial is **part 2** of a 4 part series. If you would like to see **part 1** you can go to it [here](https://progblog.io/Angular-2-Firebase-Tutorial-Part-1-Create-a-Firebase-3-CRUD-app-with-Angular-CLI/).

The finished chat app that we are building can be viewed [**here**](https://fir-crud-93710.firebaseapp.com/). The code for all 3 parts is on my [Github](https://github.com/HuddleHouse/firebase-crud), each part of this tutorial is a new branch on that repository.

### Sections

[**Part 1:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-1-Create-a-Firebase-3-CRUD-app-with-Angular-CLI/) Installing necessary components. Generating a blank app through Angular CLI. Add Bootstrap for styling. Create Firebase account. Install and setup Firebase through NPM.

The code for **part 1** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-1). 

[**Part 2:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-2-Adding-Authentication/) Add user login through their Google account. Create the Login Page and Home Page Components. Add asynchronous monitoring of the logged in status. Redirect to the Home Page when logged in and to the Login Page when the user logs out.

The code for **part 2** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-2). 

[**Part 3:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-3-Adding-the-Realtime-Database/) We will add a simple chat app that stores messages into a Firebase Realtime Database. 

The code for **part 3** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-3).

**Part 4:** COMING SOON Adding the ability for a user to edit or delete any messages that they have sent. 


### Getting Started

In this section we will cover adding user authentication using Firebase's Authentication. We will see how easy it is to log someone into our app using their Google Account, all without having to worry about any [OAuth](https://oauth.net/2/) protocol. Firebase handles all of that for you.

We will also set up Routing and Navigation using [Angular Router](https://angular.io/docs/ts/latest/guide/router.html).

CRUD stands for Create, Read, Update and Delete. Firebase take a lot of the complexity out of a web app, which makes development a lot easier!

Here is what we will be creating:

![Our Angular 2 login page using Firebase Authentication](login-pic.png)

Once the user is logged in, we display a `Logout` button and are set up to add the chat features!

![When the user is logged in using Firebase Authentication we show the homePage](homepage.png)

### About Firebase
Firebase is a swanky service that is offered by Google that allows the developer to not have to worry about many mundane tasks that slow down development. The Firebase services that we will be using is the Realtime Database, which syncs across all devices in milliseconds. We will also be using their authentication API's and I will also use theirr hosting to host the final version of the app.

I have been using Firebase for a few months now and I am a huge fan of it. It greatly speeds up development and can grow as big as your app does. All without having to have the knowledge of a System Administrator (or hiring one) to maintain the growth of your app.

## Create AngularFire2 Service Provider
### Create and Register AngularFire2 Service Provider
We can take advantage of the awesome Dependency Injection that Angular 2 has to offer by putting our service calls in a provider. This makes our app scalable and allows the functions in the provider to be used in any Component in our app!

In the `src/app` directory make a new folder called `providers`. Inside of that folder create a file called `af.ts`. Add this to `af.ts`:

```
// src/app/providers/af.ts
import {Injectable} from "@angular/core";
import {AngularFire, AuthProviders, AuthMethods} from 'angularfire2';

@Injectable()
export class AF {

  constructor(public af: AngularFire) {}

  /**
   * Logs in the user
   * @returns {firebase.Promise<FirebaseAuthState>}
   */
  loginWithGoogle() {
    return this.af.auth.login({
      provider: AuthProviders.Google,
      method: AuthMethods.Popup,
    });
  }

  /**
   * Logs out the current user
   */
  logout() {
    return this.af.auth.logout();
  }
}
```

Here we declared two functions `loginWithGoogle()` and `logout()`, that do precisely what they say. We now need to add our provider to `app.module.ts`:

```
// src/app/app.module.ts

import {AF} from "../providers/af";

@NgModule({
	....,
  	providers: [AF]
})
```

There is one more thing that we must do inside of the Firebase Console for our login to work. When logged into the Firebase Console click `Authentication` on the side menu. Then at the top click `SIGN-IN METHOD`. In this view we see all the possible ways Firebase allows Authentication to be done. Click on the Google tab and enable it.

![Here we enable Google Authentication from the Firebase Console](enable-google.png)

## Creating the Login Page Component
### Generate the Component using the Angular CLI
In Angular 2 everything is a Component. Components can be reused throughout the app which is extremely useful. The first thing that we need to have is our basic login page. The Angular CLI has a component generator that comes with it. We can run the following command to generate a `login-page` component:

```
ng generate component loginPage
```

Inside `src/app` we now have a folder called `login-page`. This component is already set up with a `.html` file for our HTML, `.ts` file for our "controller", a `.css` file that can be used to style our component and then a `.spec.ts` file that is used for unit testing. 

![After using the Angular CLI to generate a component we can see that it added a folder in our project](login-page.png)

> Note: The `.spec.ts` file is out of the scope of this tutorial. But your can read more about it [here](https://angular.io/docs/ts/latest/guide/testing.html).

When using the Angular CLI it automatically adds our new component to `src/app/app.module.ts`. I always double check to make sure it was added along with the Import statement. `app.module.ts` should now have these added to it:

```
// src/app/app.module.ts

import { LoginPageComponent } from './login-page/login-page.component';

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig)
  ],
  declarations: [ AppComponent, LoginPageComponent ],
  bootstrap: [ AppComponent ]
})
```

### Update login-page.component.ts
Now we need to use dependency injection to inject the service provider we just created (AF) and Angular Router in our constructer so that we can use them. We will create a function called `login()` that will be triggered when they click the login button. Inside of that function we will call the login function we just made. On success we will send them to our homepage (which we will create next).

```
// src/app/login-page/login-page.component.ts

import { Component } from '@angular/core';
import {AF} from "../../providers/af";
import {Router} from "@angular/router";

@Component({
  selector: 'app-login-page',
  templateUrl: './login-page.component.html',
  styleUrls: ['./login-page.component.css']
})
export class LoginPageComponent {

  constructor(public afService: AF, private router: Router) {}

  login() {
    this.afService.loginWithGoogle().then((data) => {
      // Send them to the homepage if they are logged in
      this.router.navigate(['']);
    })
  }
}
```

### Adding the Login HTML and CSS
Add the following HTML and CSS to the respective files labeled in the comments below:

```
// src/app/login-page/login-page.component.html

<div class="modal-dialog">
  <div class="loginmodal-container">
    <h1>Login to Your Account</h1><br>
    <button class="login loginmodal-submit" (click)="login()">Login With Google</button>
  </div>
</div>
```
Now lets add some CSS to make our login page look better:

```
// src/app/login-page/login-page.component.css

.loginmodal-container {
  padding: 30px;
  max-width: 350px;
  width: 100% !important;
  background-color: #F7F7F7;
  margin: 0 auto;
  border-radius: 2px;
  box-shadow: 0px 2px 2px rgba(0, 0, 0, 0.3);
  overflow: hidden;
  font-family: roboto;
}

.loginmodal-container h1 {
  text-align: center;
  font-size: 1.8em;
  font-family: roboto;
}

.loginmodal-submit {
  /* border: 1px solid #3079ed; */
  width: 100%;
  border: 0px;
  color: #fff;
  text-shadow: 0 1px rgba(0,0,0,0.1);
  background-color: #4d90fe;
  padding: 17px 0px;
  font-family: roboto;
  font-size: 21px;
   background-image: -webkit-gradient(linear, 0 0, 0 100%,   from(#4d90fe), to(#4787ed));
}

.loginmodal-submit:hover {
  /* border: 1px solid #2f5bb7; */
  border: 0px;
  text-shadow: 0 1px rgba(0,0,0,0.3);
  background-color: #357ae8;
  /* background-image: -webkit-gradient(linear, 0 0, 0 100%,   from(#4d90fe), to(#357ae8)); */
}

.loginmodal-container a {
  text-decoration: none;
  color: #666;
  font-weight: 400;
  text-align: center;
  display: inline-block;
  opacity: 0.6;
  transition: opacity ease 0.5s;
}
```

## Create the Homepage Component

We will do this using the Angular CLI again, the same way we created our Login Component.

```
ng generate component homePage
```
We also need to verify that our new component was added to `app.module.ts`:

```
// src/app/app.module.ts

import { HomePageComponent } from './home-page/home-page.component';

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig),
    RouterModule.forRoot(routes)
  ],
  declarations: [ AppComponent, LoginPageComponent, HomePageComponent ],
  bootstrap: [ AppComponent ],
  providers: [AF]
})
```

We don't need to make any changes (yet) to `home-page.component.ts`, but I am slightly OCD and did add some slight styling to `home-page.component.html`:

```
// src/app/home-page/home-page.component.html

<div class="container">
  <div class="row">
    <div class="col-md-12">
      <h1>
        home-page works!
      </h1>
    </div>
  </div>
</div>
```

## Routing and Navigation
Angular Router give us a way to make our app a [Single Page Application](https://en.wikipedia.org/wiki/Single-page_application) (SPA). To add Angular Router to our project add an import statement in `src/app/app.module.ts`. We will also need to add an array that will hold all of our routes.

This is what our `app.module.ts` looks like now:

```
// src/app/app.module.ts

import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppComponent } from './app.component';
import { AngularFireModule } from 'angularfire2';
import { LoginPageComponent } from './login-page/login-page.component';
import {RouterModule, Routes} from "@angular/router";
import {AF} from "../providers/af";
import { HomePageComponent } from './home-page/home-page.component';

export const firebaseConfig = {
  apiKey: '<your-key>',
  authDomain: '<your-project-authdomain>',
  databaseURL: '<your-database-URL>',
  storageBucket: '<your-storage-bucket>',
  messagingSenderId: '<your-messaging-sender-id>'
};

const routes: Routes = [
  { path: '', component: HomePageComponent },
  { path: 'login', component: LoginPageComponent }
];

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig),
    RouterModule.forRoot(routes)
  ],
  declarations: [ AppComponent, LoginPageComponent, HomePageComponent ],
  bootstrap: [ AppComponent ],
  providers: [AF]
})
export class AppModule {}
```

> Note: Leaving `path: ''` will redirect us to our HomePageComponent from our root url.

To make our app display the contents of the components that we are routing to. We need to add this to our host view. For us that is `src/app/app.component.html'. After our navbar add the following html:

```
// src/app/app.component.html

<div class="app-content">
  <router-outlet></router-outlet>
</div>
```

## Fine Tuning our apps Routing
We need to make our app function more intelligently. Notice that our base url take you to the `HomePageComponent`, but what if the user is not logged yet? In that case we need to redirect them to the login page. We need to make some changes to `app.component.ts`:

```
// src/app/app.component.ts

import { Component } from '@angular/core';
import { AF } from "../providers/af";
import { Router } from "@angular/router";

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  public isLoggedIn: boolean;

  constructor(public afService: AF, private router: Router) {
    // This asynchronously checks if our user is logged it and will automatically
    // redirect them to the Login page when the status changes.
    // This is just a small thing that Firebase does that makes it easy to use.
    this.afService.af.auth.subscribe(
      (auth) => {
        if(auth == null) {
          console.log("Not Logged in.");
          this.router.navigate(['login']);
          this.isLoggedIn = false;
        }
        else {
          console.log("Successfully Logged in.");
          this.isLoggedIn = true;
        }
      }
    );
  }

  logout() {
    this.afService.logout();
  }
}
```

AngularFire2 has two services built in that allow us to asynchronously sync data to Firebase. They are `af.auth` and `af.database`. The one that we use here is `af.auth` and will allow us to use the Authorization features. `af.database` allows us access to the db, which we will use in part 3 of this tutorial.

When we subscribe to `af.auth` we can monitor in real time if a user is logged in or not. If they are not logged in we redirect them to the `LoginPageComponent`. 

We also made a logout function that would log the user out. Notice that we have a boolean value named `isLoggedIn`, that gets set according to the logged in state of the user. We will use this to add a `Logout` button in our navbar, only if the user is logged in. To implement this lets change `app.component.html`:

```
// src/app/app.component.html

<nav class="navbar navbar-default">
  <div class="container-fluid">
    <div class="navbar-header">
      <a class="navbar-brand" href="#">Firebase Crud</a>
    </div>

    <div *ngIf="isLoggedIn" class="collapse navbar-collapse" id="bs-example-navbar-collapse-1">
      <ul class="nav navbar-nav navbar-right">
        <li><a (click)="logout()">Logout</a></li>
      </ul>
    </div>
  </div>
</nav>

<div class="app-content">
  <router-outlet></router-outlet>
</div>
```

## Make Sure our App Works

From the project folder run `ng serve` and then view our app in the browser. When you login to the app Firebase automatically creates a new user.

![This view shows that our new user was created in our Firebase Console](new-user.png)

Now once we are logged in this is the view that we will see:

![When the user is logged in using Firebase's Authentication we display the HomePageComponent](homepage.png)

## Conclusion

Now we are using Firebase to Authenticate the user and direct them to the homepage. In the next tutorial we will add the final part of our messaging app. If you have any questions please don't hesitate to ask me! :)
