---
title: How to deploy a Flask App to Heroku
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'Flask is a "micro" framework for Python. It is called a micro framework because they want to keep the core simple but extendable. For example: Flask does not have built in form validation or a database abstraction layer, but there are numerous plugins available to achieve those.'
tags:
  - Heroku
  - Flask
  - Python
  - pip
  - gunicorn
  - HerokuCLI
date: 2016-12-17 16:06:02
---
## Introduction
As of late I have gained a growing interest into machine learning, most notably [Tensor Flow](https://www.tensorflow.org/). That is out of the scope of this blog post, but all of the machine learning libraries are built in [Python](https://www.python.org/). So I dove into Python and made an API to do some text processing using [Flask](http://flask.pocoo.org/) and [FlaskRESTful](http://flask-restful-cn.readthedocs.io/en/0.3.5/).

Flask is a "micro" framework for Python. It is called a micro framework because they want to keep the core simple but expandable. For example: Flask does not have built in form validation or a database abstraction layer, but there are numerous plugins available to achieve those. 

Once I was satisfied with the functionality of my API I wanted to put it on [Heroku](https://www.python.org/) so I could have access to it away from my local computer. This turned out to be a little trickier than I expected, so hopefully this tutorial will save you some time. :)

## Prerequisites
You will need Flask, [gunicorn](http://gunicorn.org/), a [Heroku](https://www.heroku.com/) account and the [HerokuCLI](https://devcenter.heroku.com/articles/heroku-cli). Don't worry we will be using the free version of Heroku!

Using [pip](https://pypi.python.org/pypi/pip), Python's recommended tool for installing packages you can install them using:

<button class="right copy btn" data-clipboard-target="#cli"><i class="fa fa-clipboard"></i></button>
<div id='cli'>
```
pip install flask gunicorn
```
</div>

To install the HerokuCLI go [here](https://devcenter.heroku.com/articles/heroku-cli). On a Mac it is effortless, as they have an OS X Installer.

## Step 1: Create a simple Flask App
We need to have a working Flask app so that we have something to push to Heroku. Lets make a simple one right now. In a new folder create a file called `app.py` and put the following code inside of it:

<button class="right copy btn" data-clipboard-target="#cli2"><i class="fa fa-clipboard"></i></button>
<div id='cli2'>
```
// ./app.py
from flask import Flask

app = Flask(__name__)

@app.route('/')
def index():
	return 'Yo, it's working!'

if __name__ == "__main__":
	app.run()
```
</div>

## Step 2: List Python Dependencies
We need to list the dependencies that are required for the Heroku environtment. To list them we can run `pip freeze` from a terminal window:

![List the python dependecies](pip-freeze.png)

We need to put those dependencies in a file called `requirements.txt`. We can do that with one command:

<button class="right copy btn" data-clipboard-target="#cli3"><i class="fa fa-clipboard"></i></button>
<div id='cli3'>
```
pip freeze > requirements.txt
```
</div>

## Step 3: Create Procfile
The last thing that we need is a [Procfile](https://devcenter.heroku.com/articles/procfile). In it's simplest form you put one process per line that you want to be ran on your Heroku environment. 

In our case we want to run our Flask application and we will use gunicorn to do this. Create a file named `Procfile` and put the following in it:

<button class="right copy btn" data-clipboard-target="#cli4"><i class="fa fa-clipboard"></i></button>
<div id='cli4'>
```
web: gunicorn app:app
```
</div>

We can verify that our application works by running `python app.py` and then go to `127.0.0.1:5000` in the browser to see it in action:

![Verify that our app is working](working.png)

## Step 4: Create a new app in Heroku

Lastly, we need to create a new app in Heroku.

![This is where we create a new app in Heroku](new-app.png)

Follow the following instructions:

> Note: Your Heroku app will not be called `flask-deploy-blog`

![Final instructions for setting up a Python Heroku App](finishing.png)

At first this process was odd to me because we do not explicitly tell Heroku what type of app we will be running. Heroku goes through different build packs that they support to determine it. Heroku then goes through `requirements.txt` and installs all the dependencies for us!

At this point we are done and our app is now online with its own unique url! I hope this helped you out and if you have any questions along the way please don't desitate to ask me in the comments! :)

