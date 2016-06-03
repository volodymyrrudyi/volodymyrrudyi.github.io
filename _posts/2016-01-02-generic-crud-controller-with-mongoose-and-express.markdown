---
author: volodymyr
comments: true
date: 2016-01-02 00:56:00+00:00
layout: post
slug: generic-crud-controller-with-express-and-mongoose
title: Writing a generic CRUD controller using Mongoose and Express.js
categories:
- JavaScript
tags:
- JavaScript
- Mongoose
- Express.js
---

Often, web applications have __REST API__ for multiple models with the same set of
supported methods, e.g. __CRUD__ operations. In this case, we have to write a lot of boilerplate
for every model, which is very annoying and breaks the DRY principle.

In this post I'll describing a possible solution to this problem for REST API backends
that use __Express.js__ and __Mongoose__.

<!-- more -->

# The data model
Let's say we have an application, that should expose three entities via the REST API &mdash; countries, cities and languages and allow CRUD operations on them. We'll have these Mongoose models:

__Country__:

    const CountrySchema = new mongoose.Schema({
      isocode: String,
      name: String,
      description: String
    });

__City__:

    const CitySchema = new mongoose.Schema({
      name: String,
      description: String,
      country: {type: mongoose.Schema.Types.ObjectId, ref: 'Country'},
    });

__Language__:

    const LanguageSchema = new mongoose.Schema({
      isocode: String,
      name: String
    });

As you can see, data model is pretty simple. We assume that *City* is a nested resource of *Country*, while
language is a standalone resource.


# The REST API

Our api will look like this:

__Countries__:

    GET    /countries            - returns the list of all countries
    POST   /countries            - creates a new country with data specified in the request body
    GET    /country/{:id}        - returns a country with the given id
    GET    /country/{:id}/cities - returns all cities of the country
    PUT    /country/{:id}        - updates a country by id with data specified in the request body
    DELETE /country/{:id}        - removes a country by id


__Cities__:

    GET    /cities     - returns the list of all cities
    POST   /cities     - creates a new city with data specified in the request body
    GET    /city/{:id} - returns a city with the given id
    PUT    /city/{:id} - updates a city by id with data specified in the request body
    DELETE /city/{:id} - removes a city by id

__Languages__:

    GET    /languagues     - returns the list of all languagues
    POST   /languagues     - creates a new language with data specified in the request body
    GET    /language/{:id} - returns a language with the given id
    PUT    /language/{:id} - updates a language by id with data specified in the request body
    DELETE /language/{:id} - removes a language by id


Note, that while city belongs to a particular country, API endpoints of the city are
not nested under

    /country/{:country_id}/

while Country API includes the following method:

    /country/{:country_id}/cities

that returns all cities of the given country. This was made to simplify the structure of the API and such approach is completely RESTful.


# Straightforward approach
Traditional approach which is actually a hidden form of copy paste will be something like this:

    router.get("/countries", (req, res) => {
      // ...
    });

    router.post("/countries", (req, res) => {
      // ...
    });

    router.get("/country/:id", (req, res) => {
      // ...
    });

    router.get("/country/:id/cities", (req, res) => {
      // ...
    });

    router.put("country/:id", (req, res) => {
      // ...
    });

    router.delete("/country/:id", (req, res) => {
      // ...
    });

    router.get("/cities", (req, res) => {
      // ...
    });

    router.post("/cities", (req, res) => {
      // ...
    });

    // And so on ...


As you can see, we are repeating ourselves here. It becomes very annoying after the first or the third time.
Obviously, it can be refactored somehow.

# Generic approach

What if we could do something like this:

    var api = Router();

    api.use('/v1/countries', new CountriesController().route());
    api.use('/v1/cities',    new CitiesController().route());
    api.use('/v1/languages', new LanguagesController().route());

And controllers are just classes that look like this:

    class CountriesController extends BaseController {

      constructor(){
        // Use 'Country' Mongoose model with 'isocode' as a primary key
        super(Country, "isocode");
      }
    }

# Implementation
Most of the magic happens inside the *BaseController* class. First, we need some data to be able
to create our REST endpoints. This data is passed to the constructor:

    class BaseController{

      constructor(model, key){
        this.model = model;
        this.modelName = model.modelName.toLowerCase();
        this.key = key;
      }

      // ...
    }

The following attributes are being populated:

  * **model** - Mongoose model that should be exposes via the REST API
  * **key** - primary key of the model, e.g. country isocode.
  * **modelName** - name of the model to use in responses

To avoid callback hell and improve readability of the code, *ES6 Promises* are used.

## Create
Creates a MongoDB record and returns a promise which can be resolved to
get a response

    create(data) {
      return this.model
        .create(data)
        .then((modelInstance) => {
          var response = {};
          response[this.modelName] = modelInstance;
          return response;
        });
    }

## Read
Reads a model by it's primary key

    read(id) {
        var filter = {};
        filter[this.key] = id;

        return this.model
        .findOne(filter)
        .then((modelInstance) => {
          var response = {};
          response[this.modelName] = modelInstance;
          return response;
        });
    }


## Update
Update is implemented as pair of read + model.save() calls to trigger all handlers
defined on the model.

    update(id, data) {
      var filter = {};
      filter[this.key] = id;

      return this.model
        .findOne(filter)
        .then((modelInstance) => {
          for (var attribute in data){
            if (data.hasOwnProperty(attribute) && attribute !== this.key && attribute !== "_id"){
              modelInstance[attribute] = data[attribute];
            }
          }

          return modelInstance.save();
        })
        .then((modelInstance) => {
          var response = {};
          response[this.modelName] = modelInstance;
          return response;
        });
    }

## Delete
Deletes a model by it's primary key

    delete(id) {
      const filter = {};
      filter[this.key] = id;

      return this.model
        .remove(filter)
        .then(() => {
          return {};
        })
    }

## List
Lists all models


    list() {
      return this.model
        .find({})
        .limit(MAX_RESULTS)
        .then((modelInstances) => {
          var response = {};
          response[pluralize(this.modelName)] = modelInstances;
          return response;
        });
    }


## Routing magic
Actual magic happens in the route() method which creates an Express.js router that
handles all required HTTP methods:

    route(){
        const router = new Router();

        router.get("/", (req, res) => {
          this
            .list()
            .then(ok(res))
            .then(null, fail(res));
        });

        // ...
        return router;
    }

# Testing

To be sure our code really works, let's write some tests using *mocha* and *should*:

    describe('BaseController Unit Test', () => {

      beforeEach(function(done) {
        if (mongoose.connection.db) return done();
        mongoose.connect('mongodb://localhost/generic-controller-example', done);
      });


      const controller = new BaseController(Country, 'isocode');


      it('Test create()', (done) => {
        const data = {
          'isocode' : faker.address.countryCode(),
          'name' : faker.address.country()
        };

        controller.create(data)
        .then((response) => {
          response.should.have.property('country');
          response.country.isocode.should.equal(data.isocode);
          response.country.name.should.equal(data.name);
          done();
        })
        .then(null, done);
      });

      // ... Please, see the complete test suite on the Github
    });

That's it for now, folks. Thanks for reading. Please, feel free to reach me out if you have any questions.
You can also post your questions via comments.

# Useful links
Here are links that might be useful:

  * <a href="https://github.com/volodymyrrudyi/generic-express-mongoose-controller-example">The complete example on the Github</a>
  * <a href="http://mongoosejs.com/docs/guide.html">Mongoose Docs</a>
  * <a href="https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise">MDN Promise Documentation</a>
