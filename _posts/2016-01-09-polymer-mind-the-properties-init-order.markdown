---
author: volodymyr
comments: true
date: 2016-01-09 00:03:00+00:00
layout: post
slug: polymer-vs-reactjs
title: Polymer 1.0 &mdash; One Thing You Should Remember About Properties
categories:
- JavaScript
tags:
- JavaScript
- Polymer
- Google
---

At some point Polymer components can grow from several lines of code to several hundred, depending
on their complexity. Those components can be throughly tested and work perfectly. But at some point they can just stop working after simple code formatting change. It's because almost all Polymer components have a small time bomb inside...

<!-- more -->

Well, to be honest, I on purpose dramatized it, because actually the problem is the combination of:

  * component's code quality issue
  * non-obvious and not well documented Polymer element properties behavior
  * bad luck

# What's happening?

Ok, let's create the following element which is responsible for greeting everyone:

{% highlight html %}
<script>
  Polymer({
    is: "polymer-greeter",
    properties: {
      name: {
        type: String
      },
      message: {
        type: String,
        observer: "_onMessageChanged"
      },
      _greeting: {
        type: String,
      }
    },

    _onMessageChanged: function(value){
      this.set("_greeting", value + " " + this.name);
    },
    ready: function() {
      this.textContent = this._greeting;
    }
  });
</script>
{% endhighlight %}

And the HTML document that uses our custom component:

{% highlight html %}
<!DOCTYPE html>
<html lang="en">

<head>
  <script src="http://www.polymer-project.org/1.0/components/webcomponentsjs/webcomponents-lite.js"></script>
  <link rel="import" href="greeter.html">
</head>

<body>
  <polymer-greeter message="Hello, " name="John"/>
</body>

</html>
{% endhighlight %}

You can find this code on [Plunker](http://plnkr.co/edit/4cHhJ8?p=preview) and  check it's working correctly.
You'll see something like:

> Hello, John

Now let's just reformat the code and sort public properties alphabetically:

{% highlight html %}
<script>
  Polymer({
    is: "polymer-greeter",
    properties: {
      message: {
        type: String,
        observer: "_onMessageChanged"
      },
      name: {
        type: String
      },
      _greeting: {
        type: String,
      }
    },

    _onMessageChanged: function(value){
      this.set("_greeting", value + " " + this.name.toUpperCase());
    },
    ready: function() {
      this.textContent = this._greeting;
    }
  });
</script>
{% endhighlight %}

Again, you can check out [this snippet](http://plnkr.co/edit/3b9mfY?p=preview) on Plunker.
As you can see, now there is a JS error:

> Uncaught TypeError: Cannot read property 'toUpperCase' of undefined

So, **changing the order of properties declaration** resulted in a different component behavior.
Yes, we can consider this issue as a result of a poor code quality, but it won't be 100% true.

# Why it's happening?

Let's try to think why we are facing this issue. First of all, this code has *implicit* dependency on the property initialization order. Property observers are called whenever corresponding properties are changed. But there is no guarantee of some particular property initialization order. It means, **property observers should never assume that other properties are initialized**. As a result, observer code that accesses other properties **must** have safeguard ``` if (this.otherProperty !== undefined ) ``` or something like that:

{% highlight html %}
<script>

  Polymer({
    is: "polymer-greeter",
    properties: {
      message: {
        type: String,
        observer: "_onMessageChanged"
      },
      name: {
        type: String
      },
      _greeting: {
        type: String,
      }
    },

    _onMessageChanged: function(value){
      if (this.name !== undefined){
        this.set("_greeting", value + " " + this.name.toUpperCase());
      } else {
        this.set("_greeting", value + " NONAME");
      }
    },
    ready: function() {
      this.textContent = this._greeting;
    }
  });
</script>

{% endhighlight %}

Does it mean such check will be enough? [Of course no](http://plnkr.co/edit/7ZVDVZ?p=preview), since it will prevent the error, but will introduce another issue:

> Hello, NONAME

The  JavaScript error disappeared, but still we are seeing an incorrect message, since we set both the message and the name properties.


# How can we solve the issue?
So what should we do? The answer is: we must use a multiple properties observer:

{% highlight html %}
<script>

  Polymer({
    is: "polymer-greeter",
    properties: {
      message: {
        type: String
      },
      name: {
        type: String
      },
      _greeting: {
        type: String,
      }
    },
    observers: [
      'messageInfoUpdated(name, message)'
    ],

    messageInfoUpdated: function(name, message){
      if (this.name !== undefined){
        this.set("_greeting", message + " " + name.toUpperCase());
      } else {
        this.set("_greeting", message + " NONAME");
      }
    },
    ready: function() {
      this.textContent = this._greeting;
    }
  });
</script>
{% endhighlight %}

Great, it's working now, [take a look at the snippet](http://plnkr.co/edit/7i3dEB?p=preview)...

# Summary
* properties are not so simple, as they seem to be
* one should not rely on the property initialization order
* a small warning in the official documentation can reduce the amount of similar errors


# Useful links
* [Polymer Developer's Guide &mdash; The chapter about properties](https://www.polymer-project.org/1.0/docs/devguide/properties.html)
* [Plunker](https://plnkr.co/) &mdash; a nice tool to create, fork and execute code snippets
* [Related PR on GitHub](https://github.com/Polymer/docs/issues/1156) to add a notice to the Polymer documentation
