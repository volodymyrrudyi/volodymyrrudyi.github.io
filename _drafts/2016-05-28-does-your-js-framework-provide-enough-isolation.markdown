---
author: volodymyr
comments: true
date: 2016-05-28 00:03:00+00:00
layout: post
slug: does-your-framework-provides-enough-isolation
title: Does your client-side framework provide enough isolation?
categories:
- JavaScript
tags:
- JavaScript
- Polymer
- AngularJS
- Angular2
- React
---

Modern web applications are complex. They can be really big. And what is the best
way to reduce the complexity of something big? Right, divide it into smaller
manageable chunks. Component is a widely-used abstraction nowadays. One can find
components in **AngularJS**, **Angular 2**, **ReactJS** and **Polymer**. But do these
components really reduce the complexity of our apps ? Read the detailed answer
with many examples in this blog post.
<!-- more -->

# Prehistory
This article was conceived in a pain. The pain that happens when you try to
extend the existing application with a module built with tools and approaches
that didn't exist at the moment the original project was designed.

My task was to integrate a modern AngularJS **SPA** with the existing
**non-SPA** project that was built using traditional technologies like *JSP* and *jQuery*.
Something like this:

![](/images/posts/EmbedAngular.png)

The AngularJS application itself should be completely standalone to make it possible to
use it as a standalone mobile application.

Of course, development of the SPA with *AngularJS* and the *REST* backend is a pure
pleasure &mdash; it's easy to implement, test and deploy. But once we started
embedding the app inside the existing application we realized there is a problem:
styles and *JavaScript* code of the "host" page started to interfere with the our
application's code.

# What do we have?
Traditional non-SPA projects that have sophisticated UI usually have many things
in common:

* lots of logic on the client-side using jQuery or similar libraries (popups,
  input validation, animations, etc)
* complex CSS rules, frequently with nasty stuff like the *!important* directive,
  styles applied to elements and inline styles

Obviously, if we just insert something into such page, it will be affected by the
existing code. What should we do then? Add more *!important* directives? Include
the application being embedded first? Or...

# ...Use an <iframe>
I'm almost sure by the moment you saw the word **iframe**, you thought how miserable I am.
No worries, sometimes we need to deal with compromises. While it's not considered as a best-practice, at the moment it's the only way to resolve the problem described in a way it works
on all popular browsers without polyfills.

The *iframe* approach will give us the following:

* completely isolated parent and embedded applications. Styles and JavaScript
  code from the parent don't affect the embedded app and vice versa
* small efforts to implement
* warm feeling of early 2000-s :)

On the other hand we will need to deal with the following:

* no location history due to the fact parent URL won't be modified during the
  routing inside the   iframe. This issue can be resolved by sending message from parent app to an embedded app and vice-versa.
* to make the embedded app responsive we also need to notify it about the viewport changes
* issues with CORS, cookies if the embedded application needs to make calls to some service located on the domain of the host app.

While we had to use this option because of time constraints, I decided to investigate another solution.

# Components make our code more maintainable
OK, let's go back to components again. In theory, we can treat our old application
and the new one as two components of the same solution. And since components are meant to
be isolated pluggable entities, we could use them to isolate conflicting JavaScript code,
styles and so on by using them.
