---
title: How To Set Up a Node.js Application for Production on Ubuntu 16.04
date: 2016-09-22 14:40:20
toc: true
author: Matt Huddleston
tags:
- Node.js
- NGINX
- Ubuntu 16.04
- Matt
---

### Introduction

Node.js is an open source JavaScript runtime environment for easily building server-side and networking applications. The platform runs on Linux, OS X, FreeBSD, and Windows. Node.js applications can be run at the command line, but we'll focus on running them as a service, so that they will automatically restart <!-- more --> on reboot or failure, and can safely be used in a production environment.

In this tutorial, we will cover setting up a production-ready Node.js environment on a single Ubuntu 16.04 server. This server will run a Node.js application managed by PM2, and provide users with secure access to the application through an Nginx reverse proxy. The Nginx server will offer HTTPS, using a free certificate provided by Let's Encrypt.


### Prerequisites

This guide assumes that you have an Ubuntu 16.04 server, configured with a non-root user with sudo privileges, as described in the initial server setup guide for Ubuntu 16.04.

It also assumes that you have a domain name, pointing at the server's public IP address.

Let's get started by installing the Node.js runtime on your server.

### Install Node.js

We will install the latest current release of Node.js, using the NodeSource package archives.

First, you need to install the NodeSource PPA in order to get access to its contents. Make sure you're in your home directory, and use `curl` to retrieve the installation script for the Node.js 6.x archives:

```sh
cd ~
curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
```
You can inspect the contents of this script with `nano` (or your preferred text editor):
```sh
nano nodesource_setup.sh
```
And run the script under `sudo`:
```sh
sudo bash nodesource_setup.sh
```
The PPA will be added to your configuration and your local package cache will be updated automatically. After running the setup script from nodesource, you can install the Node.js package in the same way that you did above:
```sh
sudo apt-get install nodejs
```
The `nodejs` package contains the `nodejs` binary as well as `npm`, so you don't need to install `npm` separately. However, in order for some `npm` packages to work (such as those that require compiling code from source), you will need to install the build-essential package:
```sh
sudo apt-get install build-essential
```
The Node.js runtime is now installed, and ready to run an application! Let's write a Node.js application.
```
Note: When installing from the NodeSource PPA, the Node.js executable is called nodejs, rather than node.
```






Markdown is a lightweight markup language based on the formatting conventions that people naturally use in email.  As [John Gruber] writes on the [Markdown site][df1]

> The overriding design goal for Markdown's
> formatting syntax is to make it as readable
> as possible. The idea is that a
> Markdown-formatted document should be
> publishable as-is, as plain text, without
> looking like it's been marked up with tags
> or formatting instructions.

This text you see here is *actually* written in Markdown! To get a feel for Markdown's syntax, type some text into the left window and watch the results in the right.

### Tech

Dillinger uses a number of open source projects to work properly:

* [AngularJS] - HTML enhanced for web apps!
* [Ace Editor] - awesome web-based text editor
* [markdown-it] - Markdown parser done right. Fast and easy to extend.
* [Twitter Bootstrap] - great UI boilerplate for modern web apps
* [node.js] - evented I/O for the backend
* [Express] - fast node.js network app framework [@tjholowaychuk]
* [Gulp] - the streaming build system
* [keymaster.js] - awesome keyboard handler lib by [@thomasfuchs]
* [jQuery] - duh

And of course Dillinger itself is open source with a [public repository][dill]
 on GitHub.


### Todos

 - Write Tests
 - Rethink Github Save
 - Add Code Comments
 - Add Night Mode

License
----

MIT

**Free Software, Hell Yeah!**

[//]: # (These are reference links used in the body of this note and get stripped out when the markdown processor does its job. There is no need to format nicely because it shouldn't be seen. Thanks SO - http://stackoverflow.com/questions/4823468/store-comments-in-markdown-syntax)


   [dill]: <https://github.com/joemccann/dillinger>
   [git-repo-url]: <https://github.com/joemccann/dillinger.git>
   [john gruber]: <http://daringfireball.net>
   [@thomasfuchs]: <http://twitter.com/thomasfuchs>
   [df1]: <http://daringfireball.net/projects/markdown/>
   [markdown-it]: <https://github.com/markdown-it/markdown-it>
   [Ace Editor]: <http://ace.ajax.org>
   [node.js]: <http://nodejs.org>
   [Twitter Bootstrap]: <http://twitter.github.com/bootstrap/>
   [keymaster.js]: <https://github.com/madrobby/keymaster>
   [jQuery]: <http://jquery.com>
   [@tjholowaychuk]: <http://twitter.com/tjholowaychuk>
   [express]: <http://expressjs.com>
   [AngularJS]: <http://angularjs.org>
   [Gulp]: <http://gulpjs.com>

   [PlDb]: <https://github.com/joemccann/dillinger/tree/master/plugins/dropbox/README.md>
   [PlGh]:  <https://github.com/joemccann/dillinger/tree/master/plugins/github/README.md>
   [PlGd]: <https://github.com/joemccann/dillinger/tree/master/plugins/googledrive/README.md>
   [PlOd]: <https://github.com/joemccann/dillinger/tree/master/plugins/onedrive/README.md>
