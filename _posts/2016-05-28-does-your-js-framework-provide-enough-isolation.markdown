---
author: volodymyr
comments: true
date: 2016-06-03 00:03:00+00:00
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

While we had to use this approach because of time constraints, I decided to investigate another options.

# Components make our code more maintainable
OK, let's go back to components again. In theory, we can treat our old application
and the new one as two components of the same solution. And since components are meant to
be isolated pluggable entities, we could use them to isolate conflicting JavaScript code,
styles and so on by using them.

Let's see what AngularJS, Angular 2, Polymer and React.js can offer us in terms of components:

<table class="table table-bordered">
  <tr>
    <th>Framework name</th>
    <th>Component abstraction name</th>
  </tr>
  <tr>
    <td>AngularJS</td>
    <td>Directive, Component</td>
  </tr>
  <tr>
    <td>Angular 2</td>
    <td>Component</td>
  </tr>
  <tr>
    <td>Polymer</td>
    <td>Element</td>
  </tr>
  <tr>
    <td>React.JS</td>
    <td>Component</td>
  </tr>
</table>

Well, not much originality there, except for the Polymer.

For testing purposes I created a mock of the product page that looks like this:

![](/images/posts/OnlineShop.png)

## Using AngularJS directives to reuse pieces of the application
Here is an example of AngularJS directive:

```javascript
angular.module('agilevision.directives')
.directive('remoteBoardControls', function() {
  return {
    restrict: 'E',
    scope: {
      deviceInformation: '=deviceInformation'
    },
    templateUrl: 'scripts/directives/remote-board-controls.html'
  };
});
```

And the template:

```html
<div class="well">
  <form class="form">
    <div class="form-group">
      <label for="lcdText">LCD Text</label>
      <input type="text" name="lcdText" id="lcdText" class="form-control">
      <br>
      <button type="button" class="btn btn-primary">Send</button>
    </div>
    <div class="form-group">
      <label for="ledBrightness">LED Brightness control</label>
      <input type="number" name="ledBrightness" id="ledBrightness" class="form-control" value="10">
    </div>
    <div class="form-group">
      <input type="checkbox" name="servo" id="servo">&nbsp;
      <label for="servo">Servo enabled?</label>
    </div>
  </form>
</div>
```

After adding the directive to the original page everything looks fine:
![](/images/posts/OnlineShopAngularDirectiveNoStyle.png)

Now we got a nice feature that allows us to test the development board right
before buying it. Cool, huh? But there is something that should make us worry
about our app. As you may noticed, the embedded form is styled with the
Bootstrap theme, while we haven't included any CSS to our directive.
What will happen if we add some style, let's say a yellow background to the
directive:

```html
<style type="text/css">
body {
  background-color: yellow; /* Is it a body of the directive? */
}
</style>
<div class="well">
<form class="form">
  <div class="form-group">
    <label for="lcdText">LCD Text</label>
    <input type="text" name="lcdText" id="lcdText" class="form-control">
    <br>
    <button type="button" class="btn btn-primary">Send</button>
  </div>
  <div class="form-group">
    <label for="ledBrightness">LED Brightness control</label>
    <input type="number" name="ledBrightness" id="ledBrightness" class="form-control" value="10">
  </div>
  <div class="form-group">
    <input type="checkbox" name="servo" id="servo">&nbsp;
    <label for="servo">Servo enabled?</label>
  </div>
</form>
</div>
```

 Will the whole page  become yellow? Ideally, only the directive background should be yellow, meaning that it's contents are completely isolated from the parent page context and vice versa.
 The answer is right below:

 ![](/images/posts/OnlineShopAngularDirectiveYellow.png)

 So looks like our directive is sharing context with the parent page.
 Meaning if someone decides to add a style or a JavaScript code to the parent
 page it will affect our directive. The same will apply to the code and styles
 added inside the directive. Does not looks like a real encapsulation!

 Furthermore, directives are reusable pieces of AngularJS code. They can be
 distributed separately to be used by different projects. Avoiding styling
 in directives can be a good idea, but what about the JavaScript code?
 We need to be **extra** sure we are not affecting the parent page in our
 directives. But what if the parent page has some conflicting code? The directive
 user may spend hours trying to find the issue.

 **When creating a directive we should be sure that**:

 * it's code doesn't affect another parts of the application
 * CSS styles don't affect the parent view/component/page
 * **ID**s and **classes** are not clashing with other parts of the application

## Using AngularJS components to reuse pieces of the application

Let's take a look on the AngularJS component code:

```javascript

angular.module('ngAppDemo', [])
.component('remoteBoardControls', {
    bindings: {
      deviceInformation: '='
    },
    templateUrl: 'remote-board-controls.html'
  }).controller('ngAppDemoController', function($scope) {
});

```

Basically, to convert our directive to a component, we just need to replace
the **directive** call with the **component** call. It's pretty easy. But it
also has the same effect:
![](/images/posts/OnlineShopAngularDirectiveYellow.png)

So the same set of rules and limitations apply to components, that may be reused
by different applications.

# Angular 2 Components
OK, it seems that AngularJS is not well suitable for creating standalone components,
but what about the Angular 2? Let's do some TypeScript coding here to see how good is it.
I created a sample Angular 2 app component that contains our form:


```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'remote-board-controls-app',
  templateUrl: 'app/app.html',
})
export class AppComponent { }
```

And the HTML template:

```html
<style type="text/css">
body {
  background-color: yellow;
}
</style>
<div class="well">
<form class="form">
  <div class="form-group">
    <label for="lcdText">LCD Text</label>
    <input type="text" name="lcdText" id="lcdText" class="form-control">
    <br>
    <button type="button" class="btn btn-primary">Send</button>
  </div>
  <div class="form-group">
    <label for="ledBrightness">LED Brightness control</label>
    <input type="number" name="ledBrightness" id="ledBrightness" class="form-control" value="10">
  </div>
  <div class="form-group">
    <input type="checkbox" name="servo" id="servo">&nbsp;
    <label for="servo">Servo enabled?</label>
  </div>
</form>
</div>
```

The result is pretty interesting:

![](/images/posts/OnlineShopAngular2ComponentNotYellow.png)

As you can see, styles from the component were not applied to the parent page,
**but** Bootstrap styles from the parent page were applied to the component!
While for this particular example it's completely OK, sometimes we may want
to avoid such behavior. Luckily, Angular 2 supports several approaches of
component isolation which are called "**view encapsulation strategies**".

## Angular 2 View Encapsulation Strategies
There are three view encapsulation strategies in Angular 2: *Emulated*, *Native* and
*None*. Here is a table with a brief description of each strategy:

<table class="table table-bordered">
  <tr>
    <th>View encapsulation</th>
    <th>Description</th>
    <th>Isolation of styles</th>
    <th>Isolation of the JavaScript code</th>
    <th>Isolation of DOM elements</th>
    <th>When to use?</th>
  </tr>
  <tr>
    <td>None</td>
    <td>Don't isolate anything</td>
    <td>No</td>
    <td>No</td>
    <td>No</td>
    <td>For maximum performance when there is no need to isolate components</td>
  </tr>
  <tr>
    <td>Emulated</td>
    <td>Isolate styles by rewriting them in a way they affect only the component that contains them</td>
    <td>Yes</td>
    <td>No</td>
    <td>No</td>
    <td>To minimize the influence of styles inside the component on the application that uses the component with a minimal performance loss</td>
  </tr>
  <tr>
    <td>Native</td>
    <td>Use the Shadow DOM to render templates</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>To implement fully isolated components that should look as a black box for applications that reference them and prevent conflicts of CSS, JavaScript and DOM elements in both directions</td>
  </tr>  
</table>

By default, Angular JS uses the **Emulated** view encapsulation to provide a basic
isolation and avoid major performance and compatibility issues.

# Isolation of ReactJS Components
Approach, used in the ReactJS, for implementation of components differs from other frameworks.
A special extension to the JavaScript language is used, called JSX to describe the template of the components
right inside their code:

```
var RemoteControlApp = React.createClass({
  render: function(){
    return <div>
      <div class="well">
        <form class="form" id="control-form">
          <div class="form-group">
            <label for="lcdText">LCD Text</label>
            <input type="text" name="lcdText" id="lcdText" class="form-control" />
            <br/>
            <button type="button" class="btn btn-primary">Send</button>
          </div>
          <div class="form-group">
            <label for="ledBrightness">LED Brightness control</label>
            <input type="number" name="ledBrightness" id="ledBrightness" class="form-control" value="10" />
          </div>
          <div class="form-group">
            <input type="checkbox" name="servo" id="servo" />&nbsp;
            <label for="servo">Servo enabled?</label>
          </div>
        </form>
      </div>
    </div>;
  }
});
```

And here is the result:

![](/images/posts/OnlineShopReactNotYellowNoStyles.png)

Now, what's interesting about this code, is the fact the component is being rendered inside a so-called
Virtual DOM. But there is one problem. While styles are completely isolated (and in fact styling ReactJS components is quite painful process),
HTML elements, generated by the Virtual DOM, **can have conflicting ID, classes, etc**.
So if some component has an element with the ID attribute specified, ReactJS will generate the same HTML markup if used twice on the same page. It makes very difficult to implement really reusable components using the ReactJS.
Also, there aren't any checks against this inside the ReactJS library and it can be tricky to debug such issues.

# Google Polymer Components
Finally, we got to the Google Polymer. Since it uses the Shadow DOM, we already know the result,right?
But let's check it, just in case!

So here is an example of our component, now with a Polymer-flavored:


```html
<link rel="import" href="https://polygit2.appspot.com/components/polymer/polymer.html">

<dom-module id="board-remote-controls">

  <template>
    <style type="text/css">
    body {
      background-color: yellow;
    }
    </style>
    <div class="well">
    <form class="form" id="control-board">
      <div class="form-group">
        <label for="lcdText">LCD Text</label>
        <input type="text" name="lcdText" id="lcdText" class="form-control">
        <br>
        <button type="button" class="btn btn-primary">Send</button>
      </div>
      <div class="form-group">
        <label for="ledBrightness">LED Brightness control</label>
        <input type="number" name="ledBrightness" id="ledBrightness" class="form-control" value="10">
      </div>
      <div class="form-group">
        <input type="checkbox" name="servo" id="servo">&nbsp;
        <label for="servo">Servo enabled?</label>
      </div>
    </form>
    </div>
  </template>

  <script>
    Polymer({
      is: "board-remote-controls"
    });
  </script>
</dom-module>
```

And the result:

![](/images/posts/OnlineShopPolymerNotYellowStyles.png)

Something really tricky is going on here. Why were the styles from the parent page
applied to our component? What if we try to insert the same element twice and it has the element with ID?
Right, there will be a conflict! Does it mean Polymer uses non-native Shadow DOM implementation?
Actually, yes and no. By default, similar to Angular 2, Polymer uses **Shady DOM** for rendering elements,
which is a light-version of the **Shadow DOM** and has limitations, similar to the Emulated view encapsulation of the Angular 2 framework.

To use the full power of the Shadow DOM, we need to configure the Polymer accordingly:

```html
<script>
  /* this script must run before Polymer is imported */
  window.Polymer = {
    dom: 'shadow'
  };
</script>
```

A brief summary of each DOM rendering mode as a table:

<table class="table table-bordered">
  <tr>
    <th>DOM Rendering Mode</th>
    <th>Description</th>
    <th>Isolation of styles</th>
    <th>Isolation of the JavaScript code</th>
    <th>Isolation of DOM elements</th>
    <th>When to use?</th>
  </tr>
  <tr>
    <td>Shady DOM</td>
    <td>Isolate styles by rewriting them in a way they affect only the component that contains them</td>
    <td>Yes</td>
    <td>No</td>
    <td>No</td>
    <td>To minimize the influence of styles inside the component on the application that uses the component with a minimal performance loss</td>
  </tr>
  <tr>
    <td>Shadow DOM</td>
    <td>Use the Shadow DOM to render templates</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>Yes</td>
    <td>To implement fully isolated components that should look as a black box for applications that reference them and prevent conflicts of CSS, JavaScript and DOM elements in both directions</td>
  </tr>  
</table>

# Summary
Modern frameworks allow us to create sophisticated web applications using different
approaches. To reduce the complexity of code, most of the applications can be divided
into components. But we should take into account limitations and features of every framework/library,
since not all of them provide enough isolation of components to create a truly standalone
modules that can be inserted into any other application without causing side effects.

Google Polymer and Angular 2 are most promising, since both libraries utilize the power
of the Shadow DOM standard with a growing support among popular browsers.

# Links

* [Examples on GitHub](https://github.com/volodymyrrudyi/js-frameworks-isolation-examples)
* [ReactJS Docs](https://facebook.github.io/react/)
* [AngularJS Docs](https://docs.angularjs.org/api)
* [Angular 2 Docs](https://angular.io/docs/ts/latest/)
* [Polymer Docs](https://www.polymer-project.org/1.0/docs/devguide/feature-overview)
* [Shadow DOM Specification](https://www.w3.org/TR/shadow-dom/)
* [Webcomponents Site](http://webcomponents.org/)
