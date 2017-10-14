---
title: Create a URL Shortener with Cloud Firestore and Cloud Functions
toc: true
author: Matt Huddleston
author_img: /img/matt.jpg
author_link: 'https://huddlehouse.io'
excerpt: 'In this tutorial we will be making a Free Serverless URL Shortener that tracks clicks and Geo Location information using Google Firebase, Cloud Firestore and Cloud Functions.'
tags:
  - Cloud Firestore
  - Cloud Functions
  - Firebase
  - Firebase Hosting
  - Serverless
  - Node.js
date: 2017-10-14 16:02:52
---
## Introduction

In this tutorial we will be making a Free Serverless URL Shortener that tracks clicks and Geo Location information using <a href="https://firebase.google.com/" target="_blank">Google Firebase</a>, <a href="https://firebase.google.com/products/firestore/" target="_blank">Cloud Firestore</a> and <a href="https://firebase.google.com/products/functions/" target="_blank">Cloud Functions</a>.

#### <a href="https://pug.gl" target="_blank">View Finished Shortener (pug.gl)</a>
#### <a href="https://github.com/HuddleHouse/severless-url-shortener" target="_blank">View code on GitHub</a>

![Final product and picture of pug.gl](final.png)

I am always amazed by how quickly things change in the world of programming. It makes me feel bad for all the developers that are stuck at work using PHP on massive unmaintained Symfony projects. Sadly, I used to be one of those developers. If that sums up you, then this blog is for you! Get a taste of modern programming!

At work we wanted to have a link shortener using our own URL that we could generate short URL's through an API and track stats, such as when they were clicked and the Geo Location of the Click. There are services out there that do just that, but they are surprisingly expensive (<a href="https://bitly.com" target="_blank">bitly.com</a> enterprise is $995/month!!).

So I found a really solid article on <a href="" target="_blank">[coligo.io](https://coligo.io/create-url-shortener-with-node-express-mongo/) where they make a URL Shortener with NodeJS, Express and Mongo. With only a few modifications (and additions) to the code in the article I used <a href="https://firebase.google.com/products/firestore/" target="_blank">Cloud Firestore</a>, <a href="https://firebase.google.com/products/functions/" target="_blank">Cloud Functions</a> and <a href="https://firebase.google.com/" target="_blank">Google Firebase</a> to make <a href="https://pug.gl" target="_blank">pug.gl</a>.

It is hosted for **free**, with **no servers**!! I repeat: **free**, with **no servers**! Well servers that I manage, but you get the point.

## Features of <a href="https://pug.gl" target="_blank">pug.gl</a>

* Generate short URL's on [pug.gl](https://pug.gl)
* Build Routes and API
	* **/:encoded_id** - GET: Redirects to the Long URL
	* **/:encoded_id/test** - GET: Redirects to Long URL without saving click information in Firestore
	* **/:encoded_id/stats** - GET: Returns JSON of the stats for that particular Short URL
	* **/api/shorten** - POST: Returns Short URL {url: string, source: string}
	* **/api/stats** - GET: Returns JSON of the stats for ALL short URL's

The stats that we will save for each click are:

```
{
	url_id: string,
	clicked_on: Date,
	long_url: string, //long URL that was shortened
	ip: string, // IP address of the click
	city: string, // City of the IP
	country: string,
	lat: number,
	lon: number,
	region: string,
	zip: number
}
```

## Global Requirements
You don't necessarily need the versions I list. Technology changes quick and I want to guarantee if you have the versions listed installed it will work! I do the same in the `package.json` file later.

* NodeJS (v6.11.1)
* NPM
* <a href="https://www.npmjs.com/package/typescript" target="_blank">typescript</a> (v2.1.4)
* <a href="https://www.npmjs.com/package/firebase-tools" target="_blank">firebase-tools</a> (v3.13.1) CLI for Firebase

## Getting Started

Head on over to <a href="https://firebase.google.com/" target="_blank">Firebase</a> and create a new project.

Then from the terminal make a new folder and `cd` into it, then initialize Firebase:

```
mkdir url-shortener
cd url-shortener
firebase init
```
After running `firebase init` you will be giving a few options. By moving up and down select `Functions` and `Hosting` by clicking the space bar.

![Initializing Firebase](firebase-init.png)

During the initialization it will ask you a few questions. Hit `yes`, `enter` and `yes` to finish. You will now have the following file structure:

```
// File structure after intializing Firebase

- functions
	- node_modules
	index.js
	package.json
- public
	index.html
.firebaserc
firebase.json
```

`cd` into the functions folder and install these dependencies (I know `firebase-admin` and `firebase-functions` will already be installed, but I want to make sure you have the same versions as I do here as a new version could break things):

```
cd functions
npm install --save --save-exact firebase-admin@5.4.2
npm install --save --save-exact firebase-functions@0.7.1
npm install --save --save-exact express@4.16.2
npm install --save --save-exact cors@2.8.4
npm install --save --save-exact body-parser@1.18.2
npm install --save --save-exact geoip-lite@1.2.1

// For Typescript
npm install --save-dev --save-exact @types/node@8.0.34
// To restart automatically for local dev if you want it
npm install --save-dev nodemon
```

your `functions/package.json` file:

```
// functions/package.json

{
  "name": "functions",
  "description": "Cloud Functions for Firebase",
  "scripts": {
    "watch": "tsc *.ts providers/*.ts -w",
    "start": "nodemon index.js"
  },
  "dependencies": {
    "body-parser": "1.18.2",
    "cors": "2.8.4",
    "express": "4.16.2",
    "firebase-admin": "5.4.2",
    "firebase-functions": "0.7.1",
    "geoip-lite": "1.2.1"
  },
  "private": true,
  "devDependencies": {
    "@types/node": "8.0.34",
    "nodemon": "1.12.1"
  }
}
```

## Creating Files and Change .js to .ts

In the functions folder change `index.js` to `index.ts` and create the following files to match this structure:

![File Structure](file-structure.png)

> Note: I make a sencond public folder in the functions directory and moved the `index.html` from the original one to it.

## Building the Service Providers

Lets start by watching our files to compile Typescript to Javasctipt on change. Note: if this doesn't work make sure you added the `scripts` section to your `package.json`. Run this command in a new Terminal window and leave it running:

```
cd functions
npm run watch
```

Now lets set up our config files and then lets build the services. First lets edit `firebase-admin.ts` to look like:

```
// functions/firebase-admin.ts

const admin = require('firebase-admin');
const functions = require('firebase-functions');

admin.initializeApp(functions.config().firebase);

export const fbAdmin = admin;
```
We intialize Firebase and export it so we can use it across our app. Now lets add our config variables:

```
// functions/config.ts

var conf = {
    webhost: 'http://your-short-domain-goes-here.com/',
    redirect: 'https://storagepug.com'
};

export const config = conf;
```

> Note: The `webhost` variable is the url that will return the short URL, set this to the default firebase for now. The default is not a short URL, but if you own a short domain name you can link it to Firebase Hosting and then swap that out here


The redirect var is the URL to redirect to if the Short URL is invalid. Now lets build the service to encode and decode the hash. For more information on how this is working visit the original post <a href="https://coligo.io/create-url-shortener-with-node-express-mongo/" target="_blank">here</a>.

```
// functions/providers/hash-service.ts

var alphabet = "123456789abcdefghijkmnopqrstuvwxyzABCDEFGHJKLMNPQRSTUVWXYZ";
var base = alphabet.length; // base is the length of the alphabet (58 in this case)

export const encode = (num) => {
    var encoded = '';
    while (num){
        var remainder = num % base;
        num = Math.floor(num / base);
        encoded = alphabet[remainder].toString() + encoded;
    }
    return encoded;
};


export const decode = (str) => {
    var decoded = 0;
    while (str){
        var index = alphabet.indexOf(str[0]);
        var power = str.length - 1;
        decoded += index * (Math.pow(base, power));
        str = str.substring(1);
    }
    return decoded;
};
```

Now lets build all the functions to read and write to Google Cloud Firestore. For more information on reading and writing to Firestore the docs are <a href="https://firebase.google.com/docs/firestore/" target="_blank">here</a>.

```
// functions/providers/firebase-service.ts

import {fbAdmin} from "../firebase-admin";

var db = fbAdmin.firestore();
var counterRef = db.collection('counters');
var urlRef = db.collection('urls');
var clickRef = db.collection('clicks');

export const updateCounter = () => {
    console.log('updateCounter');
    counterRef.doc('url_count').get()
        .then(doc => {
            if (!doc.exists) {
                console.log('No such document!');
            } else {
                console.log('Document data:', doc.data());

                let data = doc.data();
                counterRef.doc('url_count').update({seq: data.seq + 1});
            }
        })
        .catch(err => {
            console.log('Error getting document', err);
        });
};

export const getCounter = () => {
    console.log('getCounter');
    return counterRef.doc('url_count').get();
};

export const saveUrl = (url, counter) => {
    console.log('saveUrl');
    return urlRef.doc(String(counter)).set(url)
};

export const findUrl = (url) => {
    return urlRef.where('long_url', '==', url).get();
};

export const addClick = (click) => {
    return clickRef.add(click);
};

export const getClickStatsForId = (id) => {
    return clickRef.where('url_id', '==', id).get();
};

export const getClickStats = () => {
    return clickRef.orderBy('url_id').get();
};

export const getUrls = () => {
    return urlRef.get();
};

export const findById = (id) => {
    return urlRef.doc(String(id)).get();
};
```

## Build the Express App

We are going to build an Express app that we will put on a Firebase Cloud Function.

```
// functions/shortener.ts

import {
    addClick, findById, findUrl, getClickStats, getClickStatsForId, getCounter, getUrls, saveUrl,
    updateCounter
} from "./providers/firebase-service";
import {config} from "./config";
import {encode, decode} from "./providers/hash-service";
import {isUndefined} from "util";
var express = require('express');
var path = require('path');
var bodyParser = require('body-parser');
var geoip = require('geoip-lite');
var cors = require('cors');

var app = express();
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static(path.join(__dirname, 'public')));
app.use(cors());

app.get('/', function(req, res){
    res.sendFile(path.join(__dirname, 'index.html'));
});

app.post('/api/shorten', function(req, res){
    var longUrl = req.body.url;
    var shortUrl = '';

    // checks to see if the URL exists
    findUrl(longUrl).then(snapshot => {
        if(snapshot.docs.length) {
            // There already is a short url, so return that
            // We take the counter id from Firestore and run it against the base58 encode function
            let id = snapshot.docs[0].id;
            shortUrl = config.webhost + encode(id);
            res.send({'shortUrl': shortUrl});
        }
        else {
            // Make a new short URL and store it in Firestore
            let date = new Date();
            let url = {
                long_url: longUrl,
                created_at: date
            };

            getCounter().then(doc => {
                let counter = doc.data().seq;
                console.log(counter)
                saveUrl(url, counter).then((ret) => {
                    console.log(ret);
                    updateCounter();
                    shortUrl = config.webhost + encode(counter);
                    res.send({'shortUrl': shortUrl});
                });
            }).catch(err => {
                console.log('Error getting document', err);
            });
        }

    }).catch(err => {
        console.log('Error getting documents', err);
    });

});

app.get('/api/stats', function(req, res){
    // Get all the click stats from Firestore
    getClickStats().then(snapshot => {
        if (snapshot.docs.length) {
            let clicks = {};
            snapshot.forEach(doc => {
                let click = doc.data();

                if(isUndefined(clicks[click.url_id])) {
                    clicks[click.url_id] = {
                        numClicks: 0,
                        longUrl: click.long_url,
                        urlId: click.url_id,
                        shortUrl: config.webhost + encode(click.url_id),
                        clickData: []
                    };
                }
                clicks[click.url_id].numClicks += 1;
                clicks[click.url_id].clickData.push(click);
            });

            getUrls().then(snap => {
                snap.forEach(doc => {
                    let url = doc.data();
                    if(isUndefined(clicks[doc.id])) {
                        clicks[doc.id] = {
                            numClicks: 0,
                            longUrl: url.long_url,
                            urlId: doc.id,
                            shortUrl: config.webhost + encode(doc.id),
                            clickData: []
                        };
                    }
                });
                res.send(clicks);
            }).catch(err => {
                console.log('Error getting document', err);
            });


        }
    }).catch(err => {
        console.log('Error getting document', err);
    });
});

app.get('/:encoded_id', function(req, res){
    var base58Id = req.params.encoded_id;
    var id = decode(base58Id);

    findById(id).then(doc => {
        if (!doc.exists) {
            console.log('No such URL!');
            res.redirect(config.redirect);
        } else {
            console.log("URL Found");
            // Redirect to the Long URL
            res.redirect(doc.data().long_url);

            // Get the IP address of the click
            var ip = getClientAddress(req.headers['x-forwarded-for']);
            // Use geoip-lite to get stats from that IP and save to Firestore
            var geo = geoip.lookup(ip);
            var lat = '';
            var lon = '';

            if(geo.ll) {
                lat = geo.ll[0];
                lon = geo.ll[1];
            }
            let date = new Date();
            let click = {
                url_id: id,
                clicked_on: date,
                long_url: doc.data().long_url,
                ip: ip,
                city: geo.city || '',
                country: geo.country || '',
                lat: lat,
                lon: lon,
                state: geo.region || '',
                zip: geo.zip || ''
            };
            addClick(click);
        }
    }).catch(err => {
        console.log('Error getting document', err);
    });
});

app.get('/:encoded_id/test', function(req, res){
    // Redirect to the long URL but do not collect stats
    var base58Id = req.params.encoded_id;
    var id = decode(base58Id);

    findById(id).then(doc => {
        if (!doc.exists) {
            console.log('No such URL!');
            res.redirect(config.redirect);
        } else {
            console.log("URL Found");
            res.redirect(doc.data().long_url);
        }
    }).catch(err => {
        console.log('Error getting document', err);
    });
});

app.get('/:encoded_id/stats', function(req, res){
    var base58Id = req.params.encoded_id;
    var id = decode(base58Id);

    // Get the stats for the Short URl and return them in JSON
    getClickStatsForId(id).then(snapshot => {

        if (snapshot.docs.length) {
            let clicks = [];
            snapshot.forEach(doc => {
                console.log(doc.id, '=>', doc.data());
                clicks.push(doc.data());
            });
            res.send({
                numClicks: snapshot.docs.length,
                clickData: clicks
            });
        }
        else {
            res.send({
                numClicks: 0,
                clickData: []
            });
        }
    })
});

function getClientAddress(request){
    return (request || '').split(',')[0];
}

export const shortener = app;
```

## Set up Firestore on Firebase

Go to the Firebase console and click Databases on the left, then change to use Cloud Firestore. Select to start in Locked Mode.

![Set up Google Cloud Firestore to run in Locked Mode](firestore-init.png)

We need to add a counter for the Hashing Algorithm to work, so lets make a collection in Firestore. The collection name should be counters, the Document ID should be url_count, and the field should be seq and be a number with the value 100.

![Set up Google Cloud Firestore collection](firestore-collection.png)

## Create Firebase Cloud Function to Expose the Express App

Change `index.ts` to look like this:

```
// functions/index.ts

import {shortener} from "./shortener";
const functions = require('firebase-functions');

// Uncomment this out to run locally, comment the function below out as well
// shortener.listen(3100, function(){
//     console.log('Server listening on port 3100');
// });

exports.shorten = functions.https.onRequest(shortener);
```

## Create Frontend View

Change `index.html` to look like this:

```
// functions/public/index.html

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="icon" type="image/x-icon" href="favicon.ico">
    <title>URL Shortener - pug.gl</title>
    <link href='https://fonts.googleapis.com/css?family=Raleway' rel='stylesheet' type='text/css'>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    <link href="css/styles.css" rel="stylesheet">
</head>
<body>

<div class="site-wrapper">
    <div class="site-wrapper-inner">
        <div class="main-container">
            <div class="inner cover">
                <img _ngcontent-c4="" alt="" class="logo" src="https://firebasestorage.googleapis.com/v0/b/storpug-main.appspot.com/o/pug%20body.png?alt=media&amp;token=d5a42474-c6c3-43ad-a36d-b7eebd2a7250">
                <span class="glyphicon glyphicon-link"></span>
                <h1>URL Shortener</h1>
                <h4>StoragePug</h4>
                <div class="row">

                    <div class="col-lg-12">
                        <div class="input-group input-group-lg">
                            <input id="url-field" type="text" class="form-control" placeholder="Paste a link...">
                            <span class="input-group-btn">
                  <button class="btn btn-shorten" type="button">SHORTEN</button>
                </span>
                        </div>
                    </div>

                    <div class="col-lg-12">
                        <div id="link"></div>
                    </div>

                </div>

            </div>
        </div>
    </div>
</div>

<script src="https://code.jquery.com/jquery-2.1.4.min.js"></script>
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
<script src="js/shorten.js"></script>
</body>
</html>
```

Now lets add the CSS and JS helper files. Change them in both public folders!

```
// functions/public/css/styles.css && public/css/styles.css

.btn:focus, .btn-shorten:focus{
    outline: 0 !important;
}

html,
body {
    height: 100%;
    background-color: #607d8b;
}

body {
    color: #fff;
    text-align: center;
    font-family: 'Raleway', sans-serif;
}

.btn-shorten {
    color: #ffffff;
    background-color: #ffa000;
    border: none;
}

.btn-shorten:hover,
.btn-shorten:focus,
.btn-shorten:active,
.btn-shorten.active {
    color: #ffffff;
    background-color: #ffa000;
    border: none;
}

.site-wrapper {
    display: table;
    width: 100%;
    height: 100%;
    min-height: 100%;
}

.site-wrapper-inner {
    display: table-cell;
    vertical-align: top;
}

.main-container {
    margin-right: auto;
    margin-left: auto;
    margin-top: 80px;
}

.inner {
    padding: 30px;
}

.inner h4 {
    padding-bottom: 30px;
}
.logo {
    max-width: 175px;
    margin-right: auto;
    margin-left: auto;
    display: block;
    margin-top: 60px;
    margin-bottom: 20px;
}
.glyphicon-link {
    font-size: 2em;
}

.inner h1 {
    margin-top: 5px;
}

#link {
    display: none;
    padding-top: 15px;
}

#link a{
    color: #ffa000;
    font-size: 1.5em;
    margin-right: 20px;
}

@media (min-width: 768px) {
    .main-container {
        width: 100%;
    }
}

@media (min-width: 992px) {
    .main-container {
        width: 700px;
    }
}
```

Now for the helper JS (SORRY ABOUT THE JQUERY!!)

```
// functions/public/js/shorten.js && public/js/shorten.js

// add an event listener to the shorten button for when the user clicks it
$('.btn-shorten').on('click', function(){
    $('#link').empty();
    // AJAX call to /api/shorten with the URL that the user entered in the input box
    $.ajax({
        url: '/api/shorten',
        type: 'POST',
        dataType: 'JSON',
        data: {url: $('#url-field').val()},
        success: function(data){
            // display the shortened URL to the user that is returned by the server
            var resultHTML = '<a class="result" target="_blank" href="' + data.shortUrl + '">'
                + data.shortUrl + '</a>';
            $('#link').html(resultHTML);
            $('#link').hide().fadeIn('slow');
        }
    });

});
```

## Edit firebase.json and Deploy!

We are going to serve the whole express app from Cloud Functions. For more info watch David East's Youtube video <a href="https://www.youtube.com/watch?v=LOeioOKUKI8" target="_blank">here</a>.

```
// firebase.json

{
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],
    "rewrites": [
      {
        "source": "**",
        "function": "shorten"
      }
    ]
  }
}
```

The key takeaway from this is that we are telling Firebase Hosting to serve from our Cloud Function instead of a static `index.html` file.

If you still have `npm run watch` running then you are good to deploy! If you don't then in your functions folder run `tsc *.ts providers/*.ts -w`. Now Deploy (not from your functions folder)!

```
firebase deploy
```

![Firebase Deployment Success](success.png)

My finished shortener is <a href="https://short.progblog.io/" target="_blank">https://short.progblog.io</a>

Link a short domain to it and then you have your own Free Serverless URL Shortener!

## Extras

When you go to <a href="https://pug.gl/2P/stats" target="_blank">https://pug.gl/2P/stats</a> you get the stats in JSON for the clicks on the shortlink. You can then use those to display useful information. For instance I used
<a href="http://www.chartjs.org/" target="_blank">ChartJS</a> to build charts for the URL's. Here is what that looks like.

![Use stats to build charts](charts.png)

Happy Coding! :)
