---
author: volodymyr
comments: true
date: 2017-01-18 00:03:00+00:00
layout: post
slug: setting-up-angularjs-environment-on-windows-10
title: Setting up AngularJS development environment on Windows 10
categories:
- JavaScript
tags:
- JavaScript
- AngularJS
- Windows
---

This is a short guide on how to setup a development environment for AngularJS
on Windows using Grunt and Bower. I had to do it recently, so decided to share
instructions, since it may save you some time, especially if you want to avoid
the trial and error path.

<!-- more -->

# Step 1: Install Git
The Git client is required by Bower and NPM for packages that are
hosted on Github. The client should be downloaded from the
[official page](https://git-scm.com/downloads) and
must be present in PATH:

![](/images/posts/angular-windows/install-git.png)

# Step 2: Install NodeJS
Go to the [official downloads](https://nodejs.org/en/download/) page and
select "Windows Installer":

![](/images/posts/angular-windows/install-node.png)

After the download is finished, launch the installer and use default settings.

# Step 3: Install Grunt CLI, Bower, and Yeoman
Open the command line prompt and execute the following command:

```
npm -g install grunt-cli bower yo generator-angular generator-karma
```

It will install Grunt and Bower to system directory so they are available for
all projects.

# Step 4: Generate the project
No need to create a boilerplate code, especially if it can be generated. Let's
use Yeoman AngularJS generator to create a sample application:

```
mkdir angular-demo
cd angular-demo
yo angular
```

Answer to all generator questions. Default values should work just fine. Once
finished, the following command can be used to run the demo application:

```
grunt server
```

A development web server will be launched at [http://localhost:9000](http://localhost:9000):
<img src="/images/posts/angular-windows/running.png" style="width: 720px">
