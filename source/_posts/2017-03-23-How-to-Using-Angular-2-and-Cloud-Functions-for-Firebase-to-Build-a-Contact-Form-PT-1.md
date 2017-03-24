---
title: 'How-to: Using Angular 2 and Cloud Functions for Firebase to Build a Contact Form, Part 1'
toc: true
author: Richard Jeffords
author_img: /img/pbdickpic.png
author_link: 'https://richardjeffords.com'
excerpt: 'In part 1 of this tutorial, we will build the frontend of an over-engineered contact form.'
tags:
  - Angular 2
  - Cloud Functions
  - Firebase
  - reCaptcha
  - forms
date: 2017-03-23 15:43:04
---
## Introduction
It's no mystery that we at progblog love Angular 2 and Firebase. When Google announced Cloud Functions for Firebase, we just had to try it out! In part 1 of this tutorial, we will build the frontend of an over-engineered contact form to learn how use Angular 2's template-driven forms with a slightly reactive approach (more on that later).

[Here](https://pb-contact-form.firebaseapp.com/) is a working preview of the form and [here](https://github.com/dockleryxk/pb-contact-form) is the github repo should you want to clone it.

In part two we will build the backend, save the submissions in the database, and use Cloud Functions to send an email of the form contents. Keep in mind that this is a simple use case for these tools, and I am using it to show off the potential. Check [here](https://firebase.google.com/docs/functions/use-cases) for more in-depth info on use cases for functions.

## Prerequisites
The set up for this app is quite similar to the [Firebase CRUD App Tutorial](https://progblog.io/Angular-2-Firebase-Tutorial-Part-1-Create-a-Firebase-3-CRUD-app-with-Angular-CLI/#Prerequisites), so head over there and follow from the "Prerequisites" section down to and including the "Add Bootstrap to the app for styling" section.
>**Note:** To stick to conventions, the angular-cli package is now called @angular/cli. Install it is like so:

<button class="right copy btn" data-clipboard-target="#cli"><i class="fa fa-clipboard"></i></button><div id='cli'>

```
npm install -g @angular/cli@latest
```
</div>

>If you already have the Angular CLI installed, but your version is out of date, check [here](https://github.com/angular/angular-cli/wiki/stories-1.0-update) for more information about upgrading.

### A Dependency Approaches..
I decided to use the [ng-recaptcha](https://www.npmjs.com/package/ng-recaptcha) package for handling the reCaptcha. I tried to do it myself at first, so I can confidently tell you now it's not worth the time. Go ahead and run the following command in the root of your project:
<button class="right copy btn" data-clipboard-target="#install-recaptcha"><i class="fa fa-clipboard"></i></button><div id='install-recaptcha'>

```
npm install ng-recaptcha --save
```
</div>

## Scaffolding
We are almost ready to start coding, but first we need to get the structure set up. To start, I added the following line to `src/styles.css` because I think Bootstrap looks much nicer this way.
<button class="right copy btn" data-clipboard-target="#styles"><i class="fa fa-clipboard"></i></button><div id='styles'>

```
* { border-radius: 0 !important; }
```
</div>

### The Model
Let's get this out of the way, because we'll never need to look at this again. The `model` in this project is the JSON used to encapsulate forms in Angular.

In the `src/app` directory make a new folder called `models`. Inside of that folder create a file called `contact-submission.ts`. Here are the contents:
<button class="right copy btn" data-clipboard-target="#model"><i class="fa fa-clipboard"></i></button><div id='model'>

```
// src/models/contact-submission.ts
export class ContactSubmission {
  public date: Date;
  public contactName: string;
  public contactEmail: string;
  public contactMessage: string;
  public contactWebsite: string;
  public captcha: string;

  constructor() {}
}
```
</div>

### The Contact Component
It's time to generate the main piece we will be working on with the cli. Use the following command:
<button class="right copy btn" data-clipboard-target="#gen-comp"><i class="fa fa-clipboard"></i></button><div id='gen-comp'>

```
ng g component contact
```
</div>

The cli should have just created a folder in `src/app` called contact containing all of the component files. Additionally, it should have automatically handling the import and declaration in `src/app/app.module.ts`, high tech!

### The Shared Module
Next we will use the cli to generate our shared module. We will use it to contain our custom email validation directive. In a real app, it is likely that there would be multiple forms. As such, it is best to keep validation directives, among other reusable code, in a shared module. For more info, [here](https://angular.io/styleguide#!#04-10) is the section about it in the style guide.

Now, let's generate it:
<button class="right copy btn" data-clipboard-target="#gen-shared"><i class="fa fa-clipboard"></i></button><div id='gen-shared'>

```
ng g module shared
```
</div>

Now we need to import this module into our app module. While we're on it, now is a good time to import ng-recaptcha package as well. To do so, add the import lines `import { SharedModule } from './shared/shared.module';` and `import { RecaptchaModule } from 'ng-recaptcha';` to the top of `app.module.ts` and then add `SharedModule` and `RecaptchaModule.forRoot()` to your imports. You *must* use forRoot() on the reCaptcha module. Your `app.module.ts` file should now look like this:
<button class="right copy btn" data-clipboard-target="#app-mod"><i class="fa fa-clipboard"></i></button><div id='app-mod'>

```
// src/app.module.ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { ContactComponent } from './contact/contact.component';
import { SharedModule } from './shared/shared.module';
import { RecaptchaModule } from 'ng-recaptcha';

@NgModule({
  declarations: [
    AppComponent,
    ContactComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
    SharedModule,
    RecaptchaModule.forRoot()
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```
</div>

### The Custom Email Validation Directive
Lastly, run `cd src/app/shared` to move locations to your shared module, and then generate the directive like so:
<button class="right copy btn" data-clipboard-target="#gen-dir"><i class="fa fa-clipboard"></i></button><div id='gen-dir'>

```
ng g directive email-validator
```
</div>

While we're here, let's build the directive so we don't have to come back. It's rather straightforward:
<button class="right copy btn" data-clipboard-target="#dir-logic"><i class="fa fa-clipboard"></i></button><div id='dir-logic'>

```
import { Directive } from '@angular/core';
import { AbstractControl, NG_VALIDATORS, Validator } from '@angular/forms';

@Directive({
  selector: '[appEmailValidator]',
  providers: [{provide: NG_VALIDATORS, useExisting: EmailValidatorDirective, multi: true}]
})
export class EmailValidatorDirective implements Validator {
  validate(control: AbstractControl) {
    const emailRegex = /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i;

    return emailRegex.test(control.value) ? null : { validEmail: { valid: false } };
  }
}
```
</div>

The important part is the validate function which tests a value against the regex and returns null if it is valid, or that atrocious JSON if invalid.

It is not a coincidence that we changed to the shared model directory before generating our new directive. The Angular cli is smart enough to recognize this and automagically import the directive into the shared module and not the app module. Unfortunately however it does not know that we are building a shared module, so we must export or directive from our module manually. Your shared module should now reflect this:
<button class="right copy btn" data-clipboard-target="#imp-dir"><i class="fa fa-clipboard"></i></button><div id='imp-dir'>

```
// src/app/shared/shared.module.ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { EmailValidatorDirective } from './email-validator.directive';

@NgModule({
  imports:      [ CommonModule ],
  declarations: [ EmailValidatorDirective ],
  exports:      [ EmailValidatorDirective, CommonModule ]
})
export class SharedModule { }

```
</div>

### Check Yo Self
That was a lot! Double check that your file structure matches mine, and then we can get started building the actual form.

![File Structure](file-structure.png)

## Form Building
Now that everything is set up, we can start building the form. In Angular 2, the are three choices in regards to the type of form building approach we take. They are template-driven, reactive, and then the middle of the road approach which incorporates elements of the two former. You may read more about that [here](https://angular.io/docs/ts/latest/cookbook/form-validation.html). The approach we are taking here is the middle of the road.

First, we need to write our main template file. Since all we're doing is displaying a form, it's very simple:
<button class="right copy btn" data-clipboard-target="#cont-html"><i class="fa fa-clipboard"></i></button><div id='cont-html'>

```
// src/app/app.component.html
<div class="container">
  <div class="row">
    <div class="col-md-4 col-md-offset-4 col-xs-12 col-xs-offset-0">
      <h1>PbContactForm</h1>
    </div>
  </div>
  <div class="row">
    <pb-contact></pb-contact>
  </div>
</div>
```
</div>

You may notice that I changed app-contact to pb-contact. TSLint does not like it, but I like naming my components to my liking. As far as I know, it does not matter.

### The CSS
This is totally optional, but the form might as well look great, right?
<button class="right copy btn" data-clipboard-target="#the-css"><i class="fa fa-clipboard"></i></button><div id='the-css'>

```
/* src/app/contact/contact.component.css */
.char-count {
  color: grey;
  font-size: 0.8em;
}

.panel {
  margin-bottom: 0;
}

.panel-body {
  max-height: 500px;
  overflow-y: auto;
}

```
</div>

### The HTML
It's time to put the face on! Let's go through it one piece at a time, starting the with outermost tags:
<button class="right copy btn" data-clipboard-target="#outer"><i class="fa fa-clipboard"></i></button><div id='outer'>

```
// src/app/contact/contact.component.html
<div class="col-md-4 col-md-offset-4 col-sm-6 col-sm-offset-3 col-xs-12 col-xs-offset-0 well">
  <form #contactForm="ngForm" [hidden]="submitted" (ngSubmit)="captchaRef.execute()">
  
  <!-- omitted -->
  
  </form>
  <div [hidden]="!submitted">
    <div class="panel panel-primary">
      <div class="panel-heading">
        <h3 class="panel-title">Contact form submitted with values:</h3>
      </div>
      <div class="panel-body">
        {{ submittedForm }}
      </div>
    </div>
  </div>
</div>
```
</div>

There's two main points to talk about here. First is the form tag.
`<form #contactForm="ngForm" [hidden]="submitted" (ngSubmit)="captchaRef.execute()">`
First, we set a template reference variable, contactForm, to keep track of the form as a whole. Next, we bind the hidden directive to trigger when the submitted flag is set. Finally, we bind the recaptcha.execute function to be triggered when the form gets submitted.

If you have used reCaptchas before, you might be perplexed by the execute function. It's because it's going to be an *invisible* reCaptcha, which isn't called until just before the form is submitted. 

The second point here is the div following the form. It simply appears after submission to show the submitted values. This will be removed from the final version. Now, let's go through each group in the form:
<button class="right copy btn" data-clipboard-target="#group1"><i class="fa fa-clipboard"></i></button><div id='group1'>

```
<div class="form-group">
      <input type="text"
             [(ngModel)]="model.contactName"
             #contactName="ngModel"
             name="contactName"
             required
             placeholder="Full Name*"
             class="form-control">
      <div *ngIf="formErrors.contactName" class="alert alert-danger">
        {{ formErrors.contactName }}
      </div>
    </div>
```
</div>
Our name field is pretty standard fare. We have our two-way binding to the model, the template reference variable, the form name, and so on. Following that, we have an error field that will show up if there are errors. More on that later.

Next is the email field:
<button class="right copy btn" data-clipboard-target="#group2"><i class="fa fa-clipboard"></i></button><div id='group2'>

```
<div class="form-group">
      <input type="email"
            [(ngModel)]="model.contactEmail"
            #contactEmail="ngModel"
            name="contactEmail"
            required
            appEmailValidator
            placeholder="Email*"
            class="form-control">
      <div *ngIf="formErrors.contactEmail" class="alert alert-danger">
        {{ formErrors.contactEmail }}
      </div>
    </div>
```
</div>

As you can see, this is largely the same, except we have referenced a validator that we have yet to define -- appEmailValidator.

Next, the simplest input, website, does not have much to it at all because it really isn't important.
<button class="right copy btn" data-clipboard-target="#group3"><i class="fa fa-clipboard"></i></button><div id='group3'>

```
<div class="form-group">
     <input type="text"
          [(ngModel)]="model.contactWebsite"
          id="contact_website"
          name="contactWebsite"
          placeholder="Website"
          class="form-control">
     </div>
</div>
```
</div>

Here is the message field, a textarea, with a small feature I added for fun:
<button class="right copy btn" data-clipboard-target="#group4"><i class="fa fa-clipboard"></i></button><div id='group4'>

```
<div class="form-group">
      <textarea [(ngModel)]="model.contactMessage"
                (keyup)="countChars($event)"
                #contactMessage="ngModel"
                name="contactMessage"
                required
                placeholder="Message*"
                class="form-control"
                maxlength="250"
                rows="6"></textarea>
      <div *ngIf="formErrors.contactMessage" class="alert alert-danger" style="margin-bottom:0">
        {{ formErrors.contactMessage }}
      </div>
      <span class="char-count">{{ charsLeft }}</span>
    </div>
```
</div>

In addition to validation, I added a binding on keyup to display the amount of characters are left before the message limit is reached. Lastly, here's the reCaptcha and submit button:
<button class="right copy btn" data-clipboard-target="#group5"><i class="fa fa-clipboard"></i></button><div id='group5'>

```
<re-captcha #captchaRef="reCaptcha"
            name="captcha"
            siteKey="your site key here"
            size="invisible"
            (resolved)="$event && onSubmit($event)"></re-captcha>
<button type="submit"
        class="btn btn-primary pull-right">Send</button>
```
</div>

The invisible reCaptcha requires the resolved binding so it can know what function to call once it is done verifying. For more information on how it works, go [here](https://www.google.com/recaptcha/intro/invisible.html). For more information on how it works in Angular 2, go [here](https://www.npmjs.com/package/ng-recaptcha).

That's it for the HTML! You could check now, but you're going to get errors since there is nothing done in the typescript. At any rate, here is the full file:
<button class="right copy btn" data-clipboard-target="#group6"><i class="fa fa-clipboard"></i></button><div id='group6'>

```
<!-- src/app/contact/contact.component.html -->
<div class="col-md-4 col-md-offset-4 col-sm-6 col-sm-offset-3 col-xs-12 col-xs-offset-0 well">
  <form #contactForm="ngForm" [hidden]="submitted" (ngSubmit)="captchaRef.execute()">
    <div class="form-group">
      <input type="text"
             [(ngModel)]="model.contactName"
             #contactName="ngModel"
             name="contactName"
             required
             placeholder="Full Name*"
             class="form-control">
      <div *ngIf="formErrors.contactName" class="alert alert-danger">
        {{ formErrors.contactName }}
      </div>
    </div>
    <div class="form-group">
      <input type="email"
             [(ngModel)]="model.contactEmail"
             #contactEmail="ngModel"
             name="contactEmail"
             required
             appEmailValidator
             placeholder="Email*"
             class="form-control">
      <div *ngIf="formErrors.contactEmail" class="alert alert-danger">
        {{ formErrors.contactEmail }}
      </div>
    </div>
    <div class="form-group">
      <input type="text"
             [(ngModel)]="model.contactWebsite"
             id="contact_website"
             name="contactWebsite"
             placeholder="Website"
             class="form-control">
    </div>
    <div class="form-group">
      <textarea [(ngModel)]="model.contactMessage"
                (keyup)="countChars($event)"
                #contactMessage="ngModel"
                name="contactMessage"
                required
                placeholder="Message*"
                class="form-control"
                maxlength="250"
                rows="6"></textarea>
      <div *ngIf="formErrors.contactMessage" class="alert alert-danger" style="margin-bottom:0">
        {{ formErrors.contactMessage }}
      </div>
      <span class="char-count">{{ charsLeft }}</span>
    </div>
    <re-captcha #captchaRef="reCaptcha"
            name="captcha"
            siteKey="6LdO5BkUAAAAAMeBwPHvMl6kZJxD5ODLO-AcyvnM"
            size="invisible"
            (resolved)="$event && onSubmit($event)"></re-captcha>
    <button type="submit"
            class="btn btn-primary pull-right">Send</button>
  </form>
  <div [hidden]="!submitted">
    <div class="panel panel-primary">
      <div class="panel-heading">
        <h3 class="panel-title">Contact form submitted with values:</h3>
      </div>
      <div class="panel-body">
        {{ submittedForm }}
      </div>
    </div>
  </div>
</div>
```
</div>

### The Typescript
Almost done! In the spirit of consistency, Let's start with the bits outside of the class:
<button class="right copy btn" data-clipboard-target="#outside-class"><i class="fa fa-clipboard"></i></button><div id='outside-class'>

```
import { Component, AfterViewChecked, ViewChild } from '@angular/core';
import { NgForm } from '@angular/forms';

import { ContactSubmission } from '../models/contact-submission';

@Component({
  selector: 'pb-contact',
  templateUrl: './contact.component.html',
  styleUrls: ['./contact.component.css']
})
export class ContactComponent implements AfterViewChecked {

// omitted

}
```
</div>

We'll be implementing AfterViewChecked to check for validation changes, and using ViewChild to access template variables. The rest is pretty standard Angular 2 stuff. Don't forget to import the model!

Now let's talk about the instance variables:
<button class="right copy btn" data-clipboard-target="#instance-vars"><i class="fa fa-clipboard"></i></button><div id='instance-vars'>

```
  model = new ContactSubmission();
  submitted = false;
  charsLeft = 250;
  contactForm: NgForm;
  @ViewChild('contactForm') currentForm: NgForm;
  submittedForm: any; // we will get rid of this later

  formErrors = {
    'contactName': '',
    'contactEmail': '',
    'contactMessage': ''
  };

  validationMessages = {
    'contactName': {
      'required': 'Name is required.'
    },
    'contactEmail': {
      'required': 'Email is required.',
      'validEmail': 'Email must be in a valid format.'
    },
    'contactMessage': {
      'required': 'A message is required.'
    }
  };
```
</div>

Wow, that is quite a list! The first three are self-evident, so I'll skip those:
* contactForm is the currentForm as the javascript last saw it. It's used to compare changes.
* @ViewChild('contactForm') currentForm is a reference to the template reference variable contactForm. Whew.
* submittedForm is serving as our diagnostic, so we will get rid of it later.
* formErrors is the object where error messages that currently apply to the form are stored.
* validationMessages is the object where we store an error message for each error type, for each field we want to provide feedback for.

Now, let's go through the first half of the functions.
<button class="right copy btn" data-clipboard-target="#function1"><i class="fa fa-clipboard"></i></button><div id='function1'>

```
 ngAfterViewChecked() {
    this.formChanged();
  }

  formChanged() {
    if (this.currentForm === this.contactForm) { return; }
    this.contactForm = this.currentForm;
    if (this.contactForm) {
      this.contactForm.valueChanges
        .subscribe((data) => this.onValueChanged(data));
    }
  }

  onValueChanged(data?: any) {
    if (!this.contactForm) { return; }
    const form = this.contactForm.form;

    for (const field in this.formErrors) {
      // clear previous error message (if any)
      this.formErrors[field] = '';
      const control = form.get(field);

      if (control && control.dirty && !control.valid) {
        const messages = this.validationMessages[field];
        for (const key in control.errors) {
          this.formErrors[field] += messages[key] + ' ';
        }
      }
    }
  }
```
</div>

ngAfterViewChecked calls what is essentially a change listener, formChanged. If there is not change, we do nothing, but if there is a change since the last check we subscribe to the form's observable.

The listener called onValueChanged in order to populate/clear the formError fields, should there be any need.

As far as the last two functions go, onSubmit doesn't do much yet, and countChars is obvious, so I'm going to paste up the whole file now.
<button class="right copy btn" data-clipboard-target="#function2"><i class="fa fa-clipboard"></i></button><div id='function2'>

```
 src/app/contact/contact.component.ts
 import { Component, AfterViewChecked, ViewChild } from '@angular/core';
 import { NgForm } from '@angular/forms';
 
 import { ContactSubmission } from '../models/contact-submission';
 
 @Component({
   selector: 'pb-contact',
   templateUrl: './contact.component.html',
   styleUrls: ['./contact.component.css']
 })
 export class ContactComponent implements AfterViewChecked {
 
   model = new ContactSubmission();
   submitted = false;
   charsLeft = 250;
   contactForm: NgForm;
   @ViewChild('contactForm') currentForm: NgForm;
   submittedForm: any; // we will get rid of this later
 
   formErrors = {
     'contactName': '',
     'contactEmail': '',
     'contactMessage': ''
   };
 
   validationMessages = {
     'contactName': {
       'required': 'Name is required.'
     },
     'contactEmail': {
       'required': 'Email is required.',
       'validEmail': 'Email must be in a valid format.'
     },
     'contactMessage': {
       'required': 'A message is required.'
     }
   };
 
   constructor() {}
 
   ngAfterViewChecked() {
     this.formChanged();
   }
 
   formChanged() {
     if (this.currentForm === this.contactForm) { return; }
     this.contactForm = this.currentForm;
     if (this.contactForm) {
       this.contactForm.valueChanges
         .subscribe((data) => this.onValueChanged(data));
     }
   }
 
   onValueChanged(data?: any) {
     if (!this.contactForm) { return; }
     const form = this.contactForm.form;
 
     for (const field in this.formErrors) {
       // clear previous error message (if any)
       this.formErrors[field] = '';
       const control = form.get(field);
 
       if (control && control.dirty && !control.valid) {
         const messages = this.validationMessages[field];
         for (const key in control.errors) {
           this.formErrors[field] += messages[key] + ' ';
         }
       }
     }
   }
 
   onSubmit(captchaResponse: string) {
     this.model.captcha = captchaResponse;
     this.submittedForm = JSON.stringify(this.model, null, 4);
     this.submitted = true;
   }
 
   countChars(event: any) {
     this.charsLeft = 250 - event.target.value.length;
   }
 }

```
</div>

## Conclusion

We now have a swanky contact form that has solid client side validation and reCaptcha functionality. In Part 2 we will write the backend which will store each submitted contact-submission model into the database, as well as send a notification email containing the fields.

Thanks for reading! Feel free to contact me with questions, concerns, or corrections.