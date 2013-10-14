Angular Restmod  [![Build Status](https://secure.travis-ci.org/angular-platanus/angular-restmod.png)](https://travis-ci.org/angular-platanus/angular-restmod)
===============
<a href="https://twitter.com/intent/tweet?hashtags=&original_referer=http%3A%2F%2Fgithub.com%2F&text=Check+out+Restmod%2C+a+prodiver+for+%23AngularJS+that+makes+a+breeze+to+use+Rest+APIs&tw_p=tweetbutton&url=https%3A%2F%2Fgithub.com%2Fplatanus%2Fangular-restmod" target="_blank">
  <img src="http://jpillora.com/github-twitter-button/img/tweet.png"></img>
</a>

### API Bound Models for AngularJS

A breif description

## Main features

- feat 1
- feat 2

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

### Creating your first REST bound object

```js
// Create a factory for our object
angular.factory('Book', function($restmod){
	// Configure the restmod object
	$restmod('http://myapiservie.com/books');
});

// You can inject the factory object in your controllers
angular.controller('MyCtrl', function(Book){

	// We can search to that api
	$scope.books = Book.$search({ search: "Game"}); // call to http://myapiservice.com/books?search=Game
})
```

## API Documentation

### Object definitions and configuration

### Object methods

### Collection methos

Take a look at https://github.com/platanus/simple-restmod-demo for a VERY basic example.
