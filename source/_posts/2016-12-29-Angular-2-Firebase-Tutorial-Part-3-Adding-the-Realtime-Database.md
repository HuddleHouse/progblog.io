---
title: 'Angular 2 Firebase Tutorial Part 3: Adding the Realtime Database'
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: "In this section we will be adding a basic chat to the app. It will sync asynchronously with Firebase's Realtime Database.f"
tags:
   - Node.js
   - Angular 2
   - Firebase
   - Angular CLI
   - Realtime Database
   - Authentication
date: 2016-12-29 23:05:04
---
## Introduction
This tutorial is **part 3** of a 5 part series. If you would like to see **part 2** you can go to it [here](https://progblog.io/Angular-2-Firebase-Tutorial-Part-2-Adding-Authentication).

The finished chat app that we are building can be seen live [**here**](https://fir-crud-93710.firebaseapp.com/). The code for all of the parts is on my [Github](https://github.com/HuddleHouse/firebase-crud), each part of this tutorial is a new branch on that repository.

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

![Using Firebase's Realtime Database we sync data in real time](chat.png)

## Firebase

Firebase offers a lot to speed up the development of an app, while allowing for growth. I come from a background of Symfony and SQL Databases, which are great but take a lot of time. The day that I experienced what Firebase and Angular 2 had to offer I have not looked back since. I even removed Sequel Pro from my computer!

There are multiple things that make Firebase's Realtime Database a technology worth learning. 

* It remains responsive regardless of internet connectivity. In a Symfony app an Error like that is usually game over. 
	* For example: while using the chat app we are building if you turn your wifi off and send messages to it. Your messages show up on screen as if they were sent, but really they are queued up and will be sent when you have internet again. When the internet comes back on not only are your messages sent, but any other messages that happened during that time show up, all without the page reloading!
* It handles the complexity of realtime synchronization for you
* It also is in the cloud and ran by Google, so your database will always be up and available. 
* It is a NoSQL database. So adding stuff to the database is amazingly easy! You don't have to update the schema and make sure your types are correct (not to mention updating the queries that don't work bc of that update to the db) you just change it and it works!

## Updating our Service Provider

There are multiple different ways to develop an app, for me I like to build the services and the functions I will need. Then I build the components to use them.

We need to add a few things to the `AF` service provider. Update `src/providers/af.ts` to look like this:

```
// src/providers/af.ts

import {Injectable} from "@angular/core";
import {AngularFire, AuthProviders, AuthMethods, FirebaseListObservable} from 'angularfire2';

@Injectable()
export class AF {
  public messages: FirebaseListObservable<any>;
  public users: FirebaseListObservable<any>;
  public displayName: string;
  public email: string;

  constructor(public af: AngularFire) {
    this.messages = this.af.database.list('messages');
  }

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

  /**
   * Saves a message to the Firebase Realtime Database
   * @param text
   */
  sendMessage(text) {
    var message = {
      message: text,
      displayName: this.displayName,
      email: this.email,
      timestamp: Date.now()
    };
    this.messages.push(message);
  }
}
```

There are a few things worth noting in this chunk of code. In the constructor we added `this.messages = this.af.database.list('messages');` this creates an AngularFire2 reference to our Firebase Database so we can read/write from it. We also added places to hold the `displayName` and `email` of our user.

We also add a function to save new messages to the db, by pushing our message object onto the Firebase Database reference. When we use push to add a message to the Database, it will automatically generate a unique key. The picture below shows what a message looks like when it gets saved to the database:

![Angularfire2 uses push to add a new key while adding it to the db](new-key.png)

> Note: There are other methods we could use to add data to the database. There are set, update and remove that are also available. Set replaces the current value in the db with the new one. Update is less destructive and will update the data. Remove will delete the object from the database.

## Update the Authentication Subscriber

When we save a message to the database we are storing the `message`, `displayName`, `email` and `timestamp`. We need to make sure that the the `displayName` and `email` are not null. 

The best way to do this would be to set them in our authentication subscriber. This is located in the constructor in `src/app/app.component.ts`. It subscribes to any change in the authenticated user. When the user logs out, this subscription sends us to the login screen. Update the constructor to look like this: 

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
          this.afService.displayName = auth.google.displayName;
          this.afService.email = auth.google.email;

          this.isLoggedIn = true;
        }
      }
    );
  }
```
When the user is logged in we set the `displayName` and `email` that are stored in our service provider.

## Update the Home Page Component

Before we add the HTML we are going to use, lets add some stuff to `home-page.component.ts`. We need to add access to the Service Provider. Update the code to look like this: 

```
// src/app/home-page/home-page.component.ts

import {Component, OnInit} from '@angular/core';
import {AF} from "../../providers/af";
import {FirebaseListObservable} from "angularfire2";

@Component({
  selector: 'app-home-page',
  templateUrl: './home-page.component.html',
  styleUrls: ['./home-page.component.css']
})
export class HomePageComponent implements OnInit {
  public newMessage: string;
  public messages: FirebaseListObservable<any>;

  constructor(public afService: AF) {
    this.messages = this.afService.messages;
  }

  ngOnInit() {}
}
```

AngularFire2 synchronizes data as lists using the `FirebaseListObservable` object. We point that to the `messages` reference we made in our service provider. We also add a variable called `newMessage`. We will use a two way binding to the input field so it will have the value of the new message that has been typed.

> Note: Since we are retrieving all the messages we need to use FirebaseListObservable, but we also have the option of FirebaseObjectObservable. We would use that if we were retrieving a single message.

In order for that two way binding to work we need to add `FormsModule` to the imports section of `app.module.ts` or we will get a gnarly error later:

```
// src/app/app.module.ts

import {FormsModule} from "@angular/forms";

@NgModule({
  imports: [
    BrowserModule,
    AngularFireModule.initializeApp(firebaseConfig),
    RouterModule.forRoot(routes),
    FormsModule
  ],
  ...
})
```

## Adding the HTML and CSS

Lets first add the CSS that will style our Message system to `home-page.component.css`:

```
// src/app/home-page/home-page.component.css

.bs-example {
  position: relative;
  padding: 15px 15px 15px;
  margin: 0 -15px 0px;
  border-color: #e5e5e5 #eee #eee;
  border-style: solid;
  border-width: 1px 0;
  background-color: #E1E2E3;
  box-shadow: inset 0 3px 6px rgba(0,0,0,.05);
  height: 60vh;
  overflow-y: scroll;
}
#messages {
  background-color: #2d384a !important;
}

.author {
  font-size: 12px;
}
.send-message {
  color: #ffffff;
  background-color: #4184f3;
  text-decoration: none;
  padding: 10px 20px 10px 20px;
  display: inline-flex;
  cursor: pointer;
}

.message-text {
  display: block;
  padding: 9.5px;
  margin: 0 0 10px;
  font-size: 13px;
  line-height: 1.42857143;
  color: #333;
  word-break: break-all;
  word-wrap: break-word;
  background-color: #f5f5f5;
  border: 1px solid #ccc;
  border-radius: 4px;
  width: 100%;
}

p {
  font-size: 16px;
}

.bubble{
  background-color: #F2F2F2;
  border-radius: 5px;
  box-shadow: 0 0 6px #B2B2B2;
  display: inline-block;
  padding: 10px 18px;
  position: relative;
  vertical-align: top;
}

.bubble::before {
  background-color: #F2F2F2;
  content: "\00a0";
  display: block;
  height: 16px;
  position: absolute;
  top: 11px;
  transform:             rotate( 29deg ) skew( -35deg );
  -moz-transform:    rotate( 29deg ) skew( -35deg );
  -ms-transform:     rotate( 29deg ) skew( -35deg );
  -o-transform:      rotate( 29deg ) skew( -35deg );
  -webkit-transform: rotate( 29deg ) skew( -35deg );
  width:  20px;
}

.me {
  display: inherit;
  margin: 5px 45px 5px 20px;
}

.me::before {
  box-shadow: -2px 2px 2px 0 rgba( 178, 178, 178, .4 );
  left: -9px;
}

.you {
  display: inherit;
  margin: 5px 20px 5px 45px;
}

.you::before {
  box-shadow: 2px -2px 2px 0 rgba( 178, 178, 178, .4 );
  right: -9px;
}

.bs-example+.highlight, .bs-example+.zero-clipboard+.highlight {
  margin: 0px -15px 15px;
  border-width: 0 0 1px;
  border-radius: 0;
}
.highlight {
  padding: 9px 14px;
  margin-bottom: 14px;
  background-color: #f7f7f9;
  border: 1px solid #e1e1e8;
  border-radius: 4px;
}
```

Now we will add the HTML to `home-page.component.html`. A lot is going on in this code. I will attempt to break it down bit by bit by labeling the key points.

```
// src/app/home-page/home-page.component.html

<div class="container">
  <div class="row">
    <div class="col-md-8 col-md-offset-2">
      <div class="bs-example" id="messages">
        
        <!-- Point: 1 -->
        <div *ngFor="let message of messages | async">
          
          <!-- Point: 2 -->
          <div class="bubble" [class.you]="isYou(message.email)" [class.me]="isMe(message.email)">
            <p>{{ message.message }}</p>
            <div class="author">
              {{ message.displayName }} | {{ message.timestamp | date:"MM/dd/yy hh:mm a" }}
            </div>
          </div>
        </div>

      </div>

      <figure class="highlight">
        
        <!-- Point: 3 -->
        <input type="textarea" class="message-text" [(ngModel)]="newMessage" (keyup.enter)="sendMessage()">
        <a class="send-message" (click)="sendMessage()">SEND</a>
        
      </figure>
    </div>
  </div>
</div>
```

*Point 1:* This is where we bind a loop to our messages. Remember that the messages object is a `FirebaseListObservable`. This means it will asynchronously stay current with the db. Any changes to the messages in the database will immediately be reflected here. We have to have to specify this relationship in the loop by adding `async`.

*Point 2:* In the CSS I have classes called `you` and `me`. Which puts the correct css for the chat bubble depending on if it was the active users or not. Using the syntax `[class.you]="isYou(message.email)"` binds the presence of the class `you` to the truthfulness of the function `isYou`. To make both `isYou` and `isMe` work we need to add those functions to `home-page.component.ts`. They return true or false if the email on the message posted matches the email of the current user.

```
// src/app/home-page/home-page.component.ts

isYou(email) {
    if(email == this.afService.email)
      return true;
    else
      return false;
  }

  isMe(email) {
    if(email == this.afService.email)
      return false;
    else
      return true;
  }
```

*Point 3:* We two way bind the input to the textarea to the variable `newMessage` by doing this `[(ngModel)]="newMessage"`. We also want the user to be able to submit their message by hitting enter, so we add the event `keyup.enter` to trigger our `sendMessage` function. We also bind the `click` event on the SEND button to trigger the `sendMessage` function also.

Now when a user sends a message they get stored in the Firebase Realtime Database!

![The structure of our Firebase Realtime Database](Firebase_Console.png)

Run `ng serve` from the terminal to very your app is working and the data is being persisted to the database.

## Final Enhancements
Now that we have a working messaging system there is only one thing left that does not work the way that I would like it. When our app loads up the oldest message appears first. I would rather it scroll to the bottom to show the most recent messages. 

To do this we need to add import `AfterViewChecked`, `ElementRef` and `ViewChild` from the core Angular 2 library. We will also need to make a function that will achieve this. Add this code to 	`home-page.component.ts`:

```
// src/app/home-page/home-page.component.ts

import {Component, OnInit, AfterViewChecked, ElementRef, ViewChild} from '@angular/core';

export class HomePageComponent implements OnInit, AfterViewChecked {
  @ViewChild('scrollMe') private myScrollContainer: ElementRef;

  ngAfterViewChecked() {
    this.scrollToBottom();
  }

  scrollToBottom(): void {
    try {
      this.myScrollContainer.nativeElement.scrollTop = this.myScrollContainer.nativeElement.scrollHeight;
    } catch(err) { console.log('Scroll to bottom failed yo!') }
  }
}
```

We need to add the `ngAfterViewChecked` function bc it gets called after every check of the component's view, which will trigger the `scrollToBottom` function.

We bind an `ElementRef` variable to `myScrollContainer` to the local variable `scrollMe`. We need to add that variable to `home-page.component.html` so that our app knows which div needs to be scrolled.

```
// src/app/home-page/home-page.component.html

<div #scrollMe class="bs-example" id="messages">
   <div *ngFor="let message of messages | async">
       <div class="bubble" [class.you]="isYou(message.email)" [class.me]="isMe(message.email)">
          <p>{{ message.message }}</p>
          <div class="author">
              {{ message.displayName }} | {{ message.timestamp | date:"MM/dd/yy hh:mm a" }}
          </div>
       </div>
   </div>
</div>
```
Now our messaging system functions how it should!

## Conclusion

At this point we have a working messaging app that uses Firebase to Authenticate users and store the chat messages in the Realtime Database. I hope this helps you learn how to harness the power that Angular 2 and Firebase have.

If you have any questions please don't hesitate to ask me! :)

Oh yeah I almost forgot. I do realize the title says it is a CRUD and we are not updating or deleting anything form our Database. I will be adding a 5th part to this series so that users can edit their own messages or delete them.