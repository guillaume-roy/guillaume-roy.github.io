---
layout: blog
title: Ionic 2 Login/Password Authentication with Firebase
description: 'How to integrate a standard login/password authentication using Firebase system in a Ionic 2 application.'
cover: "/img/posts/2017-04-25-ionic2-login-password-auth-with-firebase/cover.png"
date: 2017-04-25 00:00:00 Z
---

If your [Ionic 2](https://ionicframework.com/getting-started/) application need an authentication system, [Firebase](https://firebase.google.com/) can be a solution. Though it, you could use [several providers](https://firebase.google.com/docs/auth/) like Login/Password, Facebook, Google, Twitter, Github or your own custom system.

In this article, we will see how to integrate a standard login/password authentication using Firebase system in a Ionic 2 application.

## Prerequisites

Before starting this tutorial, you will need to :

* [Install Node.js 6 or greater](https://nodejs.org/)
* Install Ionic CLI
  * `npm install -g ionic cordova`
* [Create an account and a project on Firebase](https://console.firebase.google.com/)
* Configure the login/password authentication in your Firebase project

## Prepare the application

The first step is to initialize our Ionic 2 application using the basic `tabs` template :

```
ionic start ionic2-firebase-auth-sample-app tabs
cd ionic2-firebase-auth-sample-app
```

Then we add [AngularFire2](https://github.com/angular/angularfire2) package to be able to communicate with Firebase :

`npm install firebase angularfire2 --save`

Next step is to configure Firebase credentials in `/src/app/app.module/.ts` like explained in [documentation](https://github.com/angular/angularfire2/blob/master/docs/1-install-and-setup.md#4-setup-ngmodule) :

```
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
    AngularFireModule.initializeApp(firebaseConfig)
  ]
})
export class AppModule {}

```

## Create Authentication service

In order to allow our pages to access to Firebase, we will create an authentication service :

`ionic g provider auth`

This service provides 4 methods : `login`, `register`, `signOut` and `getAuth`.

You can use the code below :

```
import { Injectable } from '@angular/core';
import 'rxjs/add/operator/map';
import { AuthProviders, AuthMethods, FirebaseAuthState, AngularFireAuth, FirebaseApp } from 'angularfire2';

@Injectable()
export class Auth {
  private authState: FirebaseAuthState;
  private auth: any;

  get authenticated(): boolean {
    return this.authState !== null;
  }

  get uid(): string {
    return this.authState.uid;
  }

  constructor(private auth$: AngularFireAuth,
   @Inject(FirebaseApp) fa : any) {
     this.authState = null;
     this.auth = fa.auth();
  }

  getAuth() {
    this.auth$.subscribe((state: FirebaseAuthState) => {
      this.authState = state;
    });
    return this.auth$;
  }

  login(credentials: { email: string, password: string }) {
    if (credentials.email === null || credentials.password === null) {
      return Promise.reject("Please insert credentials");
    } else {
      return this.auth$.login({
        email: credentials.email,
        password: credentials.password
      }, {
        provider: AuthProviders.Password,
        method: AuthMethods.Password
      });
    }
  }

  register(credentials: { email: string, password: string }) {
    if (credentials.email === null || credentials.password === null) {
      return Promise.reject("Please insert credentials");
    } else {
      return this.auth$.createUser({
        email: credentials.email,
        password: credentials.password
      });
    }
  }

  signOut() {
    this.auth$.logout();
  }
}
```

Don't forget to declare it in `providers` section of `/src/app/app.module.ts` :

```
import { Auth } from '../providers/auth';

@NgModule({
  providers: [
    Auth
  ]
})
export class AppModule {}
```

Then we will create 2 new pages :

* Signup/Login page : used by our user to login or create his account
* Waiting page : a "proxy" page displayed during the authentication process. If the user is not authenticated, the application will navigate to the login page and if he is logged-in, the user will see the home page.

## Create Waiting Page

Basically you can display your logo or a waiting spinner in this page.

So we create the page using the [CLI generator function](https://ionicframework.com/docs/cli/generate/) :

`ionic g page waitingPage`

Then you can copy/paste the source below for a basic UI in your `/src/pages/waiting-page/waiting-page.html` :

```
<ion-content padding text-center>
  <h1><ion-spinner></ion-spinner> Loading</h1>
</ion-content>
```

Finally, we set this new page as the root page of the application :

* Import the `WaitingPage` in `/src/app/app.module.ts` and add it in `declarations` and `entryComponents` arrays.
* Set it as `rootPage` in `/src/app/app.component.ts`.

## Create Login/Signup Page

Again, we will use the CLI generator function :

`ionic g page loginPage`

You can also proceed in the same way that previously to declare it in `app.module.ts` & `app.component.ts`.

Here is a basic UI structure for `/src/pages/login-page/login-page.html` :

```
<ion-content padding>
  <ion-card>
    <ion-card-content>
      <form (ngSubmit)="login()" #loginForm="ngForm">
        <ion-row>
          <ion-col>
            <ion-list inset>
              <ion-item>
                <ion-label floating>Email</ion-label>
                <ion-input type="email" name="email" required [(ngModel)]="credentials.email"></ion-input>
              </ion-item>
              <ion-item>
                <ion-label floating>Password</ion-label>
                <ion-input type="password" name="password" required [(ngModel)]="credentials.password"></ion-input>
              </ion-item>
            </ion-list>
          </ion-col>
        </ion-row>
        <ion-row>
          <ion-col>
            <button ion-button full color="primary" type="submit" [disabled]="!loginForm.form.valid">Login</button>
            <button ion-button block clear full color="secondary" (click)="signup()" type="button">Signup</button>
          </ion-col>
        </ion-row>
      </form>
    </ion-card-content>
  </ion-card>
</ion-content>
```

We're using [Angular Form](https://angular.io/docs/ts/latest/guide/forms.html) so we have access to standard validation (required & input email type) and also we can easily disable actions if the form isn't validated.

And here is the code that will interact with our service previously created :

```
import { Component } from '@angular/core';
import { NavController, NavParams, IonicPage } from 'ionic-angular';
import { Auth } from '../../providers/auth';

@IonicPage()
@Component({
  selector: 'page-login-page',
  templateUrl: 'login-page.html',
})
export class LoginPage {
  credentials = {
  	  email: '',
  	  password: ''
  };

  constructor(
    private nav: NavController,
    private navParams: NavParams,
    private authService: Auth) {
  }

  login() {
    this.authService.login(this.credentials);
  }

  signup(){
    this.authService.register(this.credentials);
  }
}
```

## The Glue

The final touch is the below little code block that will redirect user either on the LoginPage or on the HomePage according to its authentication state.

Copy/Paste this code in `/src/app/app.component.ts` :

```
@Component({
  templateUrl: 'app.html'
})
export class MyApp {
  authenticated: boolean;
  rootPage:any = WaitingPage;

  constructor(platform: Platform, statusBar: StatusBar, splashScreen: SplashScreen, public authService: Auth) {
    this.authenticated = false;

    platform.ready().then(() => {
      // Okay, so the platform is ready and our plugins are available.
      // Here you can do any higher level native things you might need.
      statusBar.styleDefault();
      splashScreen.hide();

      this.authService.getAuth().subscribe(state => {
        this.authenticated = this.authService.authenticated;

        if(this.authenticated) {
          this.rootPage = TabsPage;
        } else {
          this.rootPage = LoginPage;
        }
      });
    });
  }
}
```

Due to the fact that `getAuth()` returns an Observable, if the auth state changes, the callback method will be re-executed and the redirection will be fluent.

## Source

You can get sources and run the application here : [https://github.com/guillaume-roy/ionic2-firebase-auth-sample-app](https://github.com/guillaume-roy/ionic2-firebase-auth-sample-app)