---
author: volodymyr
comments: true
date: 2015-12-28 00:56:00+00:00
layout: post
slug: using-es7-decorators-with-babel-6
title: Using ES7 decorators with Babel 6
categories:
- Scripting Languages
tags:
- JavaScript
- EcmaScript
- ES7
- Babel
---


Though ES7 features are experimental, many of us have already tried them using
Babel 5. After upgrading to 6.x the following error can happen:

`Decorators are not supported yet in 6.x pending proposal update`

This post describes how to overcome this issue and explains why does it happen.

<!-- more -->


What's happening?
==
Most likely you know that decorators proposal to ES7 is not finalized yet. It
means that while feature is available and widely used, the way they work still can be changed.
Because of it, Babel developers decided that decorators should not be available in the new 6.x version until proposal is finalized. It means that everyone who upgraded from Babel 5.x to Babel 6.x will encounter the error above.

How to solve the issue?
==
Obviously, there is a bunch of code that already utilizes such handy stuff as __ES7 decorators__.
Not having it in Babel 6 made people downgrade to the 5.x, but luckily, there is a Babel 6 plugin called __Transform Decorators Legacy__ by GitHub user <a href="https://github.com/loganfsmyth">loganfsmyth</a> that can be found <a href="https://github.com/loganfsmyth/babel-plugin-transform-decorators-legacy">here</a>.

Modify your __.babelrc__ so it looks like this:

    {
      "presets": <your_list_of_presets>,
      "plugins": "babel-plugin-transform-decorators-legacy"
    }

More detailed information about the issue and the current status can be found at the <a href="http://phabricator.babeljs.io/T2645">Babel Phabricator</a>
