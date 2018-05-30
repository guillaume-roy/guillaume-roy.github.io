---
layout: blog
title: Continuous Deployment for Ionic 2 to Firebase Hosting
description: 'Continuously deploy a PWA Ionic application to Firebase Hosting'
cover: "/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/cover.png"
date: 2017-02-23 00:00:00 Z
---

[Firebase](https://firebase.google.com/) is great to quickly integrate a noSQL database in a web application. The platform provides also [Firebase Hosting](https://firebase.google.com/docs/hosting/) which is a static files hosting service.
This is useful especially if you need to deploy a progressive web app based on [Ionic 2](https://ionicframework.com/getting-started/).

## Concept

The concept behind the continuous deployment for the above case is the following :

* You need 2 Git branches : `master` & `develop` :
  * `develop` is, obviously, where you push your daily work.
  * `master` is the production-ready branch and used to deploy the application.
  * For the purpose of this article, the Git repository is hosted on Github.
* When a stable state of the application is defined, a merge is applied from `develop` branch to `master` branch.
* The source code on `master` branch is then automatically deployed on your Firebase Hosting account, ready to be served for users.

The link between the merge on `stable` branch and the deployment on Firebase Hosting is handled by [Travis CI](https://travis-ci.org). If you don't have an account already, I recommend you to create one.

## Firebase Configuration

First thing you have to do is to configure the Firebase access for your project.

Install the [Firebase CLI](https://firebase.google.com/docs/cli/) :

````
npm install -g firebase-tools
````

Then you login to Firebase though the CLI :

````
firebase login
````

Next step is to initialize a Firebase project in the web application :

````
cd my_project/
firebase init
````

You will then have to answer few questions :

* Are you ready to proceed? (Y/n) `Y`
* What Firebase CLI features do you want to setup for this folder? `Hosting`
* What Firebase project do you want to associate as default? `Choose your project or create a new one`
* What file should be used for Database Rules? `Press ENTER`
* What do you want to use as your public directory? (public) `www`
  * Ionic 2 will publish the final files in `/www/` directory.
* Configure as a single-page app (rewrite all urls to /index.html)? (y/N) `y`
* File www/index.html already exists. Overwrite? (y/N) `N`

*Firebase initialization complete!*

Three files are created in the project directory :

* `firebase.json`
  * Configure which files are published upon deployment.
* `.firebaserc`
  * Contains informations about your Firebase project.
* `database.rules.json`
  * Contains access rules (read/write) and permissions.

### NOTE

By default Ionic 2 creates a `.gitignore` file containing the line `www/`, but we have defined in `firebase.json` to deploy the `www` directory.

So you have to delete this line in the `.gitignore` file of the `master` branch in order to be able to push the `www` directory.

## Travis Configuration

Now that Firebase is well configured, you need to activate your Github repository with your Travis account. After that, Travis will watch any push inside your `master` branch and trigger the deployment.

Connect to your Travis account and activate your Github repository :

1. On the left of the main page, click on "+" at the right of "My repositories" :
![Add repository](/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/01.png "Add repository")
2. On the next page, activate your Github repository :
![Activate repository](/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/02.png "Activate repository")
3. Click on the gear icon to edit project's "General Settings" like this :
![Edit settings](/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/03.png "Edit settings")

In order to allow Travis to connect and then deploy your code on Firebase Hosting you will have to login to Firebase in "continuous integration" mode :

````
firebase login:ci
````

You received a token. This token will be added in the Travis project's settings in the "Environment Variables" like this :
![Firebase CI Token](/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/04.png "Firebase CI Token")

The token will be passed to the Firebase CLI in order to deploy your code in the right place.

## Project Settings

Final step is to configure the deployment workflow that Travis will do.
Create `.travis.yml` at the root of your project directory with the following content :

```
language: node_js
node_js:
  - "stable"

branches:
  only:
    - master

before_script:
  - npm install -g firebase-tools

after_success:
  - firebase deploy --token=${FIREBASE_TOKEN}
```

## Time to test

Now that everything is set-up, it's time to test !

* Create a modification in your `develop` branch then commit and push it.
* Checkout your `master` branch :
  ```
  git checkout master
  ```
* Merge your previous commits and push them :
  ```
  git merge develop
  git push
  ```

Now if you connect to your Travis account, you will see that the last push is currently building.
After few minutes, the build will pass from yellow (in progress) to green (succeeded).

![Travis Building](/img/posts/2017-02-23-continous-deployment-ionic2-firebase-hosting/05.png "Travis Building")

It means that your application is currently deployed and accessible from your users. Travis will also send you a confirmation email.

You can now navigate to [https://your-project.firebaseapp.com](https://your-project.firebaseapp.com) and admire the result.
The whole list of different deployments can be viewed in the Hosting section of your Firebase account.

Bravo, you have now a successful continuous deployment workflow !

## Cherry on the cake

An interesting feature provided by Travis is the ability to display on the project README the status of the last build.
On your Travis account, navigate to your repository. At the top you should see a button displaying "build passing".
Click on it and in the popup, select the "Markdown" value. Add the result in the README and you're ready to go.

## References

* [https://marlosoft.net/posts/automatic-deploy-firebase-github-travis.html](https://marlosoft.net/posts/automatic-deploy-firebase-github-travis.html)
* [https://docs.travis-ci.com/user/languages/javascript-with-nodejs/](https://docs.travis-ci.com/user/languages/javascript-with-nodejs/)