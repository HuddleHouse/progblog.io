---
title: 'Angular 2 Firebase Tutorial Part 4: Adding Email / Password Authentication'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'In this part of the series we will add Email/Password Authentication using Firebase. We will also add a registration page. '
tags:
  - Node.js
  - Angular 2
  - Firebase
  - Authentication
date: 2017-01-07 13:38:43
---
## Introduction
This tutorial is **part 4** of a 5 part series. If you would like to see **part 1** you can go to it [here](https://progblog.io/Angular-2-Firebase-Tutorial-Part-1-Create-a-Firebase-3-CRUD-app-with-Angular-CLI/).


The finished chat app that we are building can be seen live [**here**](https://fir-crud-93710.firebaseapp.com/). The code for all the parts is on my [Github](https://github.com/HuddleHouse/firebase-crud), each part of this tutorial is a new branch on that repository.

### Sections

[**Part 1:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-1-Create-a-Firebase-3-CRUD-app-with-Angular-CLI/) Installing necessary components. Generating a blank app through Angular CLI. Add Bootstrap for styling. Create Firebase account. Install and setup Firebase through NPM.

The code for **part 1** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-1).

[**Part 2:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-2-Adding-Authentication/) Add user login through their Google account. Create the Login Page and Home Page Components. Add asynchronous monitoring of the logged in status. Redirect to the Home Page when logged in and to the Login Page when the user logs out.

The code for **part 2** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-2).

[**Part 3:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-3-Adding-the-Realtime-Database/) We will add a simple chat app that stores messages into a Firebase Realtime Database.

The code for **part 3** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-3).

[**Part 4:**](https://progblog.io/Angular-2-Firebase-Tutorial-Part-4-Adding-Email-Password-Authentication/) We add Email/Password user authentication using Firebase.

The code for **part 4** can be found [here](https://github.com/HuddleHouse/firebase-crud/tree/firebase-crud-part-4).

**Part 5:** COMING SOON Adding the ability for a user to edit or delete any messages that they have sent.

## Getting Started

In this part of the series we will add Email/Password Authentication using Firebase. We will also add a registration page.

### Login Page
![The final Angular 2 login page](login.png)

### Registration Page
![the final Angular 2 register page](register.png)

## Enable Email/Password Auth in Firebase
We need to log into [Firebase](https://firebase.google.com) and go to Authentication -> Sign-In Method and enable Email/Password Authentication.

![Succefully enabling Email/Password Authentication in Firebase](email/password.png)

## Generate the Registration Page

Using the Angular CLI interface generate the RegistrationPageComponent.

![Using Angular CLI to genereate a new component](new-component.png)

Now that the page is generated that users will register on we need to add a route in `app.module.ts` to point to it.

```
// src/app/app.module.ts

const routes: Routes = [
  { path: '', component: HomePageComponent },
  { path: 'login', component: LoginPageComponent },
  { path: 'register', component: RegistrationPageComponent}
];
```

## Update the Service
Now that we have a Registration page generated we need to add three functions to `src/providers/af.ts`:

```
/**
   * Calls the AngularFire2 service to register a new user
   * @param model
   * @returns {firebase.Promise<void>}
   */
  registerUser(email, password) {
    console.log(email)
    return this.af.auth.createUser({
      email: email,
      password: password
    });


  }

  /**
   * Saves information to display to screen when user is logged in
   * @param uid
   * @param model
   * @returns {firebase.Promise<void>}
   */
  saveUserInfoFromForm(uid, name, email) {
    return this.af.database.object('registeredUsers/' + uid).set({
      name: name,
      email: email,
    });

   /**
   * Logs the user in using their Email/Password combo
   * @param email
   * @param password
   * @returns {firebase.Promise<FirebaseAuthState>}
   */
  loginWithEmail(email, password) {
    return this.af.auth.login({
        email: email,
        password: password,
      },
      {
        provider: AuthProviders.Password,
        method: AuthMethods.Password,
      });
  }
```

## Update the Registration Page
### registration-page.component.ts
Lets first update `registration-page.component.ts`. We need to inject `Router` and our `AF` provider into the component. We also build a `register` function that will register a user using our new created service functions. Upon success we use the `Router` to send them to the home page.

```
// src/app/registration-page/registration-page.component.ts

import { Component } from '@angular/core';
import {AF} from "../../providers/af";
import {Router} from "@angular/router";

@Component({
  selector: 'app-registration-page',
  templateUrl: './registration-page.component.html',
  styleUrls: ['./registration-page.component.css']
})
export class RegistrationPageComponent {
  public error: any;

  constructor(private afService: AF, private router: Router) { }

	//registers the user and logs them in
  register(event, name, email, password) {
    event.preventDefault();
    this.afService.registerUser(email, password).then((user) => {
      this.afService.saveUserInfoFromForm(user.uid, name, email).then(() => {
        this.router.navigate(['']);
      })
        .catch((error) => {
          this.error = error;
        });
    })
      .catch((error) => {
        this.error = error;
        console.log(this.error);
      });
  }
}

```

> Note: If we catch an error we set a variable named error to the value of the erro

### registration-page.component.html

Now lets add the registration form to the screen. Add this code to `registration-page.component.ts`:

```
// src/app/registration-page/registration-page.component.html

<div *ngIf="error" class="alert alert-warning" role="alert">
  <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
  <span class="sr-only">Error:</span>
  {{error}}
</div>

<div class="modal-dialog">
  <div class="registermodal-container">
    <h1>Register</h1><br>

    <form class="form-signin" (submit)="register($event, name.value, email.value, password.value)">
      <label for="name" class="sr-only">Name</label>
      <input #name type="text" id="name" class="form-control" placeholder="Name" required="">
      <label for="email" class="sr-only">Email address</label>
      <input #email type="email" id="email" class="form-control" placeholder="Email address" required="" autofocus="">
      <label for="inputPassword" class="sr-only">Password</label>
      <input #password type="password" id="inputPassword" class="form-control" placeholder="Password" required="">
      <br>
      <button  class="btn btn-md btn-primary btn-block" type="submit">Register</button>
    </form>
  </div>
</div>

```

> Note: The top portion of this code will not show unless we run into and error.

We create local variables on our form by using `#name`, `#email` and `#password`. This is so that on form submission we can pass the values to Firebase.

### registration-page.component.css

This is easy part but lets add some css to make this look better.

```
// src/app/registration-page/registration-page.component.css

.registermodal-container {
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

.registermodal-container h1 {
  text-align: center;
  font-size: 1.8em;
  font-family: roboto;
}
```

Check toverify that the registered user is showing up in your Firebase Console:

![We now see the user data is showing up in the console](database.png)

Now we have a registration form that registers a new user using Firebase and logs that user in! We are almost finished.


## Update Login Page

Lastly, we need to add a way for a registered user to log in using thier email and password.

### login-page.component.ts
Lets first add a `loginWithEmail` function to `login-page.component.ts`. For clarity I also change my `login` function to `loginWithGoogle`:

```
// src/app/component/login-page/login-page.component.ts

import { Component } from '@angular/core';
import {AF} from "../../providers/af";
import {Router} from "@angular/router";

@Component({
  selector: 'app-login-page',
  templateUrl: './login-page.component.html',
  styleUrls: ['./login-page.component.css']
})
export class LoginPageComponent {
  public error: any;

  constructor(public afService: AF, private router: Router) {}

  loginWithGoogle() {
    this.afService.loginWithGoogle().then((data) => {
      // Send them to the homepage if they are logged in
      this.afService.addUserInfo();
      this.router.navigate(['']);
    })
  }

  loginWithEmail(event, email, password){
    event.preventDefault();
    this.afService.loginWithEmail(email, password).then(() => {
      this.router.navigate(['']);
    })
      .catch((error: any) => {
        if (error) {
          this.error = error;
          console.log(this.error);
        }
      });
  }
}
```

### login-page.component.html
We need to add a place to display an error message if there is one. We also need to add the form to take in the Email/Password of the user.

```
// src/app/component/login-page/login-page.component.html

<div *ngIf="error" class="alert alert-warning" role="alert">
  <span class="glyphicon glyphicon-exclamation-sign" aria-hidden="true"></span>
  <span class="sr-only">Error:</span>
  {{error}}
</div>

<div class="modal-dialog">
  <div class="loginmodal-container">
    <h1>Login to Your Account</h1><br>

    <form class="form-signin" (submit)="loginWithEmail($event, email.value, password.value)">
      <label for="email" class="sr-only">Email address</label>
      <input #email type="email" id="email" class="form-control" placeholder="Email address">
      <label for="password" class="sr-only">Password</label>
      <input #password type="password" id="password" class="form-control" placeholder="Password" required="">
      <br>
      <button class="btn btn-md btn-primary btn-block" type="submit">Sign in</button>
    </form>
    <hr>

    <button class="login loginmodal-submit" (click)="loginWithGoogle()">Login With Google</button>
    <hr>
    <a (click)="router.navigate(['register'])" style="cursor: pointer;">Register Now</a>
  </div>
</div>
```
We once again use local variables to transport the Email and Password to Firebase.

The final thing we need to do is update the constructor in `app.component.ts`.

```
// src/app/app.component.ts

constructor(public afService: AF, private router: Router) {
    // This asynchronously checks if our user is logged it and will automatically
    // redirect them to the Login page when the status changes.
    // This is just a small thing that Firebase does that makes it easy to use.
    this.afService.af.auth.subscribe(
      (auth) => {
        if(auth == null) {
          console.log("Not Logged in.");

          this.isLoggedIn = false;
          this.router.navigate(['login']);
        }
        else {
          console.log("Successfully Logged in.");
          // Set the Display Name and Email so we can attribute messages to them
          if(auth.google) {
            this.afService.displayName = auth.google.displayName;
            this.afService.email = auth.google.email;
          }
          else {
            this.afService.displayName = auth.auth.email;
            this.afService.email = auth.auth.email;
          }

          this.isLoggedIn = true;
          this.router.navigate(['']);
        }
      }
    );
  }
```

We now have two ways for a user to be able to login. If they are are using Google we need to get thier name from `auth.gooogle` and `auth.auth` otherwise.

## Conclusion

Now we have a working Email/Password login and registration page using Firebase. If you have any questions please ask in the comments :)