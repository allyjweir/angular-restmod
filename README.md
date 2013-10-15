Angular Restmod  [![Build Status](https://secure.travis-ci.org/angular-platanus/angular-restmod.png)](https://travis-ci.org/angular-platanus/angular-restmod)
===============
<a href="https://twitter.com/intent/tweet?hashtags=&original_referer=http%3A%2F%2Fgithub.com%2F&text=Check+out+Restmod%2C+a+prodiver+for+%23AngularJS+that+makes+a+breeze+to+use+Rest+APIs&tw_p=tweetbutton&url=https%3A%2F%2Fgithub.com%2Fplatanus%2Fangular-restmod" target="_blank">
  <img src="http://jpillora.com/github-twitter-button/img/tweet.png"></img>
</a>

TODO: Summary and Motivation

Take a look at https://github.com/platanus/simple-restmod-demo for a VERY basic example.

## Breaking Changes

    Wait for it....

## Getting Started

### Get the code

- Using bower `bower install angular-restmod`
- Downloading the code for [development](https://github.com/angular-platanus/angular-restmod/raw/master/dist/angular-restmod.js) or [production](https://github.com/angular-platanus/angular-restmod/raw/master/dist/angular-restmod-min.js)

### Dependencies

You will need to add Angular to your project

### Adding restmod module as a dependency to your project

```js
angular.module('myApp', ['plResmod'])
```

## Overview

You can use Restmod to make a RESTfull API resource available to your app.

This is done by building a model to wrap de resource:

```javascript
module('library').factory('Book', function($restmod) {
    return $restmod('/api/books', {
        createdAt: { type: 'rails-date' }, // this requires the rails-date serializer to be registered
        chapters: { hasMany: 'Chapter' }
    })
});
```

And then injecting the model in any controller or service:

```javascript
module('library').controller('BooksController', function($scope, Book) {

    // Call to GET /api/books
    $scope.books = Book.$search();

    $scope.loadBook = function(_idx) {

        // Call to GET /api/books/XX
        $scope.book = this.books[_idx].$fetch();

        // Call to GET /api/books/XX/chapters?active=true
        $scope.chapters = $scope.book.chapters.$search({ active: true });
    }

    $scope.addChapter = function(_name) {

        // Call to POST /api/books/XX/chapters with data = { name: _name }
        $scope.chapters.$create({ name: _name })
    }

});
```

By default, api calls URLs are build using the provided REST url builder.

When using the default URL builder, the api must comply with a series of basic rules:
* Returned resources must provide a primary key or an url property.
* Nested resource url are built by appending the nested resource partial url to the master resource url.
* TODO: complete this list.

## Configuration

For most scenarios, no configuration is required to start working with restmod. Depending on your needs you may want to do some tweaking, the following methods are available to change the library behaviour globaly

```javascript
module('app').configure(function($restmodProvider) {
    // To change the default URL builder
    $restmodProvider.setUrlBuilder(RestUrlBuilderFactory('pid', '/api/v1'));

    // To disable automatic attribute renaming
    $restmodProvider.setAttributeRenaming(false);

    // IDEA: To specify wrapped or inlined JSON object responses
    // $restmodProvider.setExpectWrapedObjects(true);

    // IDEA: set custom $http interceptor
    // $restmodProvider.setHttpInterceptor(interceptor);

    // IDEA: global extension registration
    // $restmodProvider.setGlobalBase();
})
```

## General usage

### Building a model

Models are built using the `$restmod` service, the recommended way of

The first argument for the `$restmod` function is the resource base url, if not given, then resource is considered an **anonymous** resource.

After the url, the `$restmod` function accepts a random number of arguments that build up the mixin chain. In the example below, the mixin chain is composed of an attribute description object and a building function.

```javascript
module('MyModule').factory('Book', function($restmod) {
    return $restmod('/api/books', {
        // This an argument -> behavior map used to configure each of the model attributes.
        createdAt: { serialize: 'rails-date', ignore: true },
        pages: { init: 0 },
        chapters: { hasMany: 'Chapter' }
    }, function() {
        // This is another way of configuring the model by direct builder method calling.
        // The following code has the same effect as the argument map above.
        this.attrSerializer('createdAt', 'rails-date')
            .attrIgnored('createdAt', true)
            .attrDefault('pages', 0)
            .hasMany('chapters', 'Chapter');
    });
});
```

#### The model builder mixin chain

Every argument passed to the `$restmod` factory function after the url is considered a builder unit. When building a model, units are processed in argument position order and pased to the model builder to modify the object definition. This chaining of definitions is called the mixin chain.

The following builder units are supported:

* *object:* A regular object to be provided to the builder's `describe` method.
* *function:* A function to be called in the builder context.
* *string:* A string holding another model's name, the other model's mixin chain is then injected in the current model's chain (this is some kind of inheritance.)

TODO: example

#### Abstract models

Abstract models are added to provide a way of holding a resusable mixin chain without the overload of building a full fledge model type.

TODO: usage

### Default values

To setup an attribute's default value use the `init` attribute property

    pages: { init: 10 }

or the `attrDefault` builder method

    builder.attrDefault('pages', 10);

If the given value is a function, it will be called every time a new object is built.

### Manipulation of model objects

To create a new object instance use `$build`, this will not produce any requests.

    var newBook = Book.$build({ title: 'Papelucho', pages: 2000 });

To store an object changes use `$save`, depending on the object being new or not, this will produce a POST or a PUT request.

    newBook.$save();

To retrieve an object's updated information from the server use `$fetch`. Since this is an asynchronous method, the object will not be modified until the server response is received, more on this later on Asynchronous Requests and Promises. Also, an object can only be fetch if it is bound to a given url, for this to happen the object must comply with every restriction imposed by the selected url builder.

    newBook.$fetch();

To build and save an object all in one step use `$create`

    var newBook = Book.$create({ title: 'Papelucho', pages: 2000 });

To look for an object by primary key use `$find`

    var oldBook = Book.$find(1);

#### Asynchronous methods and promises

Every method that produces a server request is considered asynchronous, whenever an asynchronous methods is called on an object, it's $promise property is updated with a model return promise. The model return promise passes as argument the model instance (instead of the response).

    var book = Book.$find(1);
    console.log(book) // {}
    book.$promise.then(function() {
        console.log(book); // { title: 'bla', pages: 10 }
    });

The `$then` method is also provided in every object that produces requests to allow for easy method chaining.

    Book.$find(1).$then(function(_book) {
        console.log(book); // { title: 'bla', pages: 10 }
    });

### Seach and collections

Use the `$search` method to begin a search.

    // This will produce a GET request to /api/books?title=Ulises
    result = Book.$seach({ title: 'Ulises' });

The returned object is a **collection**, collections are extended array objects that can be used to manage a particular group of model objects. Collections receive every model class method on creation.

Collections also have a `$fetch` and a `$then` method.

    // The passed object is optional, if can be used to modify a single query.
    result.$fetch({ min-pages: 10 });

Also, collections can be specialized into new collections by calling `$search`.

    // newResult is a new collection that filters by title and min-pages
    newResult = result.$search({ 'min-pages': 10 });

Even though every method available in the model class is available in the model collections, most manipulation methods are *context sensitive*, meaning that calling $create on a collection that is bound to a given url/resource can produce a POST request to an url diferent from the base resource url. This is specially relevant when using **relations**.

### Relations

A Restmod models can be associated or bound to another model using *relations*.

#### hasMany relation

This kind of relation generates a new foreign model collection in each of the host model instances.

Given

   module('MyModule')
     .factory('Book', function($restmod) {
     	return $restmod('/api/books', {
	     	authors: { hasMany: 'Author' },
	     	chapters: { hasMany: 'Chapter' }
	     });
	 })
     .factory('Chapter', function($restmod) { return $restmod(null); })
     .factory('Author', function($restmod) { return $restmod('/api/author'); });

Then calling `chapters` will return a collection of `Chapter` objects, the collection will be empty until `$fetch` is called and the server returns.

    var chapters = Book.find(1).chapters.$fetch();

Also, `authors` will return objects bound to **api/authors/:id**, but `chapters` will produce objects bound to **api/books/1/chapters/:id** (because chapters is an anonymous resource).

Calling $create on a relation will produce a POST request to the nested resource and will automatically add the new object to the array.

#### hasOne relation

TODO

### Decoding / Encoding

TODO

### Ignored attributes and the request mask

TODO

### Callbacks

TODO

## Extending

TODO

