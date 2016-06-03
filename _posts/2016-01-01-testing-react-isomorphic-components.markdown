---
author: volodymyr
comments: true
date: 2016-01-01 00:56:00+00:00
layout: post
slug: testing-react-js-isomorphic-components
title: Testing Isomorphic ReactJS Components with Mocha and Should
categories:
- Scripting Languages
tags:
- JavaScript
- ReactJS
- testing
---

__ReactJS__ framework allows to decompose your application UI into small reusable parts called *components*.
Components are small, self-explanatory and don't rely on any external state,
meaning they are easy to test. The framework comes with a set of test tools called __React Test Utils__.

If you are writing an isomorphic JavaScript application, you probably want to be able to run components
tests without a browser. But once you try to use React Test Utils the following error may occur:

    ReferenceError: document is not defined
        at Object.ReactTestUtils.renderIntoDocument (node_modules/react/lib/ReactTestUtils.js:70:15)

In this post I'm explaining the reason of this error, how to resolve it and give some examples of using __Mocha__ and __Should__ to test a ReactJS component.

<!-- more -->

Dude, where is my *window.document*?
==

Let's consider the following test example:


    import React          from 'react';
    import should         from 'should';
    import TestUtils      from 'react-addons-test-utils';
    import {ProductsList} from 'components/products';


    describe('Products list component test', function(){
      it('Should render products', function(){
        // ...
        const productsList = TestUtils.renderIntoDocument(<ProductsList products={products} />);
        // ...
      });
    }

If we execute this test with __mocha__:

    mocha --compilers js:babel-core/register --recursive

It will result in the error mentioned above: ```ReferenceError: document is not defined```
This is happening because the code is being executed __without a browser__, meaning *document*, *window* and *document.window* are __absent__, while renderIntoDocument expects DOM to be available!

Meet the jsdom
==
Luckily, there is a <a href="https://github.com/tmpvar/jsdom">jsdom</a> library which provides a DOM implementation in JS.
To make DOM available in your tests, you can use <a href="https://github.com/rstacruz/mocha-jsdom">mocha-jsdom</a>. It can be used without any additional configurations:

    import React          from 'react';
    import should         from 'should';
    import jsdom          from 'mocha-jsdom';
    import TestUtils      from 'react-addons-test-utils';
    import {ProductsList} from 'components/products';


    describe('Products list component test', function(){
      jsdom(); // Provide a DOM for ReactJS

      it('Should render products list', function(){

        // ...

        const productsList = TestUtils.renderIntoDocument(<ProductsList products={products} />);
        // ...
      });

    });

Now you can run as any other test with mocha:

    mocha --compilers js:babel-core/register --recursive

Output:

    Products list component test
      ✓ Should render  list of products

It should work now
==
Now you can use the __should__ library style assertions to write tests:

    listItems.should.have.lengthOf(1);
    listItems[0].should.not.be.empty;
    listItems[0].textContent.should.be.equal('Test product 1');

  
Check the <a href="https://github.com/volodymyrrudyi/mocha-jsdom-reactjs-example">full example on Github</a>.
