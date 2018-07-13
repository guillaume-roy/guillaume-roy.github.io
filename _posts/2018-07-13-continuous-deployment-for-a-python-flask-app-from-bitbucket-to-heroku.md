---
layout: blog
title: Continuous deployment for a Python Flask app from Bitbucket to Heroku
description: 'How to set up a continuous deployment from Bitbucket to Heroku for a Python Flask application using Bitbucket Pipelines.'
cover: "/img/posts/2018-07-13-continuous-deployment-for-a-python-flask-app-from-bitbucket-to-heroku/cover.png"
date: 2018-07-13 00:00:00 Z
---

[Flask](http://flask.pocoo.org/) is a great Python framework which can be used to create a web application. In my current project I wanted to set up a continous deployment to an [Heroku](https://heroku.com) app. My sources are versionned in a Git repostory hosted on [Bitbucket](https://bitbucket.org).

## Prerequisites

Before starting this tutorial, you will need to :

* [Create a Flask app](http://flask.pocoo.org/docs/1.0/quickstart/#quickstart)
  * _I recommand you to use a Python [virtual environment](http://flask.pocoo.org/docs/1.0/installation/?highlight=venv#virtual-environments) in order to isolate your dependencies._
* Host your sources in a Git repo on [Bitbucket](https://bitbucket.org)
* Create an heroku account and create a new app in [your dashboard](https://dashboard.heroku.com/apps)

In this article, I will use 2 Git branches :
  * `master` will be used to deploy a stable version
  * `develop` for development purposes

## Create Procfile

[Procfile](https://devcenter.heroku.com/articles/procfile) is a file that specificies to Heroku what commands to launch at the start of your application.

To run your Flask application, we will use the Python package [`gunicorn`](http://gunicorn.org/) which is a small web server.

Here is the content of the Procfile :

```
web: gunicorn app:app
```

This file will be placed in the root directory of your project.

## Set up Bitbucket Pipeline

[Bitbucket Pipelines](https://bitbucket.org/product/features/pipelines) is a feature of the Bitbucket platform that allow to integrate a continuous integration and deployment in your development process.

So when a new code will be pushed on the `master` branch, Bitbucket will create a new pipeline to get your entire code and deploy it on your Heroku app.

### Configuration of bitbucket-pipelines.yml

`bitbucket-pipelines.yml` is a file that will be again in the root directory of your project. It will configure the pipeline and describe the steps to deploy the code to Heroku.

Here is the content (It's YAML so be careful with indentation) :

```
image: python:3.7.0 # Specifies Python version
clone:
  depth: full # Tell to clone the repository all deepth
pipelines:
  branches:
    master: # Only for master branch
      - step:
          name: Deploy to Heroku # Step name
          deployment: production # Target environment
          script:
            - git push https://heroku:$HEROKU_API_KEY@git.heroku.com/$HEROKU_APP_NAME.git HEAD # Push & Deploy to Heroku app
```

### Configure Bitbucket Pipeline variables

In the yml file, we used 2 pipeline variables : `HEROKU_API_KEY` & `HEROKU_APP_NAME`.

This avoids to store in the Git repo some production environment credentials.
But we still need to configure them.

* Go to your repository homepage on Bitbucket
* Click on "Settings" in the left menu
* Maybe you will have to enable Pipelines feature in 'Settings' page of the 'Pipelines' section
* Click on "Environment variables" in the "Pipelines" section
* Create and fill the `HEROKU_APP_NAME` which is the name of your Heroku app
* Create and fill the `HEROKU_API_KEY` which is the API key fo your Heroku account
  * You can find it in your [account settings](https://dashboard.heroku.com/account) at the 'API Key' section
  * Click 'Reveal' on the right to copy/paste it

## Save Python dependencies

To be able to execute your application, Heroku will need to install your dependencies in its app. In order to achieve this, you have to save the list of them in a file. Typically, this is the command to launch in the root directory of your project :

```
pip freeze > requirements.txt
```

`requirements.txt` contains now all your dependencies.

## Final step

Now your final step is to commit and push into `master` branch the newly created files and let Bitbucket Pipeline take the rest of the deployment process part. Now, everytime you push some code on `master` branch, a new version is deployed in the Heroku app.