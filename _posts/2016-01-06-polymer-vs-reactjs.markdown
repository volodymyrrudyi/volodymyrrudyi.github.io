---
author: volodymyr
comments: true
date: 2016-01-06 00:03:00+00:00
layout: post
slug: polymer-vs-reactjs
title: Polymer and React.js &mdash; Clash of Titans
categories:
- JavaScript
tags:
- JavaScript
- React
- Polymer
- Google vs Facebook
---

Recently, I had a chance to apply **React.js** and **Polymer** in real-world applications.
While both React and Polymer share the same idea of **components**, they use rather
different approaches to achieve the component-oriented design of the front-end.
In this post I'm describing pros and cons of both tools that are important for me.

<!-- more -->

# Who is who?
**React.js** is the JavaScript library that allows to decompose UI into small reusable Components
brought to us by Facebook.

**Polymer** is the library to create reusable web-components developed by Google.

As you can see, both libraries are maintained by world's biggest companies, so they are constantly being improved by the most talented engineers.

# The Approach
Both Polymer and React assume that modern UI should be built using the component-based architecture, rather than
assuming it's an HTML document animated with a help of JavaScript.

Polymer was created as a library that provides [Web Components](https://en.wikipedia.org/wiki/Web_Components)-compatible elements for building web application UIs. Since Web Components are not fully supported by all popular browsers, **polyfills** are required to make them work properly. The most popular polyfill library is **webcomponents.js**.

React.js does not rely on the Web Components standard and provides it's own implementation of the component-based UI architecture.

Here is a table with a comparison of used approaches:


<table class="table table-bordered">
  <thead>
    <tr>
      <th>Feature</th>
      <th class="text-center">React</th>
      <th class="text-center">Polymer</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Component architecture</td>
      <td class="text-center">Own</td>
      <td class="text-center">WebComponents Standard</td>
    </tr>
    <tr>
      <td>Components are reusable</td>
      <td class="text-center">Yes</td>
      <td class="text-center">Yes</td>
    </tr>
    <tr>
      <td>Custom HTML tags</td>
      <td class="text-center">Yes</td>
      <td class="text-center">Yes</td>
    </tr>
    <tr>
      <td>Templates support</td>
      <td class="text-center">Yes</td>
      <td class="text-center">Yes</td>
    </tr>
    <tr>
      <td>Shadow DOM</td>
      <td class="text-center">Yes</td>
      <td class="text-center">Yes</td>
    </tr>
    <tr>
      <td>How to reuse a component?</td>
      <td class="text-center">Loaded as a JavaScript module</td>
      <td class="text-center">HTML import</td>
    </tr>
    <tr>
      <td>Compilation/preprocessing</td>
      <td class="text-center">Code should be transpiled since it uses JSX syntax</td>
      <td class="text-center">No compilation or preprocessing required. HTML imports can be optimized if needed</td>

    </tr>
    <tr>
      <td>Server-side rendering</td>
            <td class="text-center">Completely supported and allows building isomorphic applications</td>
      <td class="text-center">Not fully supported and limited to prerendering </td>
    </tr>
    <tr>
      <td>What's included in the standard distribution</td>
      <td class="text-center">Only the library, no components</td>
      <td class="text-center">The library, set of basic components and Material Design elements</td>
    </tr>
    <tr>
      <td>Central registry of components</td>
      <td class="text-center">Yes, NPM</td>
      <td class="text-center">Yes, <a href="https://elements.polymer-project.org/">Polymer Catalog</a></td>
    </tr>
  </tbody>
</table>

As you can see, libraries are somewhat different. The best part of Polymer is that once Web Components standard is completely supported by all major browsers, it will get lots of goodies like shadow DOM, templates and HTML imports "for free". On the other hand, React.js allows to create **isomorphic** applications, meaning that components are being rendered on the server and in the browser in the same way. It's a very important feature that influences load time, SEO and performance in general. Besides it, React.js components are pretty similar to web components and most likely at some point there will be a way to distribute a React.js component as a web-component.

# Installation

Polymer is distributed solely using the Bower tool. There are two reasons for it:

* it's intended to be a client-side only library
* Bower supports component distribution from repositories based on tags which are heavily used in the Polymer

ReactJS is available both as a Bower component and  a NPM module. *Preferred* way is using NPM.

# Preprocessing and compilation in React.js
One of the things that can be unusual for React.js newcomers is the fact it implies a heavy usage of preprocessing and translation. First of all, to simplify component code React.js team introduced a **JSX** syntax. Basically, it's an extension of the JavaScript syntax that allows you to replace DOM manipulation code with a DOM nodes XML. It means that *you can insert an HTML-like code right inside your JavaScript module":

    export default class MyComponent extends React.Component {
      render() {
        return (
          <div>
            <h1>This is a React.js component</h1>
            <p className="text-center">Note, "className" is used instead of "class"
            to match the DOM API spec.
            </p>
          </div>
        );
      }
    }

Most likely you noticed that even without DOM markup this code looks bit different from other JavaScript.
It's because in most cases the React.js community uses *ES6* in their applications. ES6 is the latest version of the JavaScript standard. The problem is it's not completely supported both by browsers and NodeJS. To overcome this issue, **all ES6 code is also being translated** to a ES5 that can be processed by any modern web-browser and NodeJS.


All required translation can happen in several ways:

  * code can be transpiled directly in the browser. While it's handy for prototyping and testing, such approach will result in a rather poor performance.
  * code can be compiled into ES5 during the build process. This approach is used to deploy UI to the production. After the compilation code is also being minified and obfuscated.
  * code can be compiled on the fly by a development web server that allows hot code reloading without refreshing a browser


# Component Development
After developing components for both libraries and dealing with components developed by
Polymer and React communities,  I can say that from the development point of view strengths of both libraries sometimes are turning into weaknesses.

## Polymer
HTML nature of the components without any additional translation required, makes it very easy to work with components developed by someone else. Inside the component there is a clear separation between markup and JavaScript code.  On the other hand, sometimes components are "too HTML-ish". They can contain DOM manipulations directly in the code, styling right in the markup and look very imperative.

## React
While simplifying a lot of things, JSX can become very ugly, especially when dealing with repeating components that should be created using **map()** or similar functions. Another issue with the translation process is that sometimes the new ES6 standard is not completely supported by any existing implementation, including Babel [or changes too often](/scripting%20languages/2015/12/28/babel-6-decorators.html). But I must admit, that usage of ES6 makes React.js components way more readable than Polymer elements written in ES5.


# Communities
Both communities are very active and helpful, though React.js community seems more active to me, maybe because of the hype around the library.

# Summary
React.js and Polymer are amazing libraries that can be used to build [awesome](https://github.com/enaqx/awesome-react) [things](https://github.com/Granze/awesome-polymer). React.js is designed to be simple and powerful tool to build UI of any level of complexity, while Polymer aims to be a framework for creating reusable web components.

# Links

* [Official ReactJS Website] (https://facebook.github.io/react/)
* [Official Polymer Website] (https://www.polymer-project.org/1.0/)
