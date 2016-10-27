# feathers-authentication-popups

[![Build Status](https://travis-ci.org/feathersjs/feathers-authentication-popups.png?branch=master)](https://travis-ci.org/feathersjs/feathers-authentication-popups)
[![Code Climate](https://codeclimate.com/github/feathersjs/feathers-authentication-popups/badges/gpa.svg)](https://codeclimate.com/github/feathersjs/feathers-authentication-popups)
[![Test Coverage](https://codeclimate.com/github/feathersjs/feathers-authentication-popups/badges/coverage.svg)](https://codeclimate.com/github/feathersjs/feathers-authentication-popups/coverage)
[![Dependency Status](https://img.shields.io/david/feathersjs/feathers-authentication-popups.svg?style=flat-square)](https://david-dm.org/feathersjs/feathers-authentication-popups)
[![Download Status](https://img.shields.io/npm/dm/feathers-authentication-popups.svg?style=flat-square)](https://www.npmjs.com/package/feathers-authentication-popups)

> Server and client utils for implementing popup-based authentication flows

## Installation

```
npm install feathers-authentication-popups --save
```

## Documentation

This package includes a few useful tools for implementing popup-based OAuth login flows.  It contains utility functions for the browser and middleware for the server.

Please refer to the [feathers-authentication-popups documentation](http://docs.feathersjs.com/) for more details.

## Using the Client Utils
There are two client utilities: one to assist with opening popups, and another to assist in cross-window communication.

## `openLoginPopup(url, options)`
Opens a centered popup window at the given `url`.

```js
import openLoginPopup from 'feathers-authentication-popups';

openLoginPopup('/auth/github');
```

1. **url** `{String}`: The URL of the new window.
2. **options** `{Object}`: optional - allows for customizing the `width` and `height` of the popup window.

The default `options` are:
* `width`: 1024,
* `height`: 630

## `authAgent`

An EventEmitter automatically assigned as a global on the `window` object to allow popup windows to send information back to the main application window.  Both windows must be on the same domain.

**Usage in the primary application window:**

```js
import {authAgent) from 'feathers-authentication-popups';

function doSomethingWithToken (token) {
	// Do something with the token
}

authAgent.on('login', doSomethingWithToken);
```

**Usage in the popup window on the same domain:**
```js
var token = readCookie('feathers-jwt');

// Trigger the 'login' event on the main window's `authAgent`
window.opener.authAgent.emit('login', token);
```

### `authAgent.on(eventName, handler)`

Adds an event listener to the `authAgent` whose handler runs every time the event with given `eventName` is triggered.
1. **eventName** `{String}`: The name of the event to subscribe to.
2. **handler** `{Function}`: A function to be executed to handle the event.

### `authAgent.once(eventName, handler)`

Adds an event listener to the `authAgent` whose handler runs only once when the event with given `eventName` is triggered.
1. **eventName** `{String}`: The name of the event to subscribe to.
2. **handler** `{Function}`: A function to be executed to handle the event.

### `authAgent.off(eventName, handler`)

Removes a handler function from the `authAgent`
1. **eventName** `{String}`: The name of the event to unsubscribe from.
2. **handler** `{Function}`: A reference to a previously-subscribed function to be unsubscribed.

### `authAgent.emit(eventName, args)

Triggers the event attached to the provided `eventName` and calls the subscribed handlers with the `args`.
1. **eventName** `{String}`: The name of the event to trigger.
2. **args** `{any}`: arguments to be passed to event handlers, usually authentication-related information (like a JSON Web Token).

## Using the Express Middleware

The Express middleware is meant to be registered as the success callback of the Feathers OAuth2 plugin.

Here's an example of a Feathers server that uses `feathers-authentication-popups`. 

```js
const feathers = require('feathers');
const rest = require('feathers-rest');
const hooks = require('feathers-hooks');
const bodyParser = require('body-parser');
const errorHandler = require('feathers-errors/handler');
const handleAuthPopups = require('feathers-authentication-popups/middleware');

// Initialize the application
const app = feathers()
  .configure(rest())
  .configure(hooks())
  // Needed for parsing bodies (login)
  .use(bodyParser.json())
  .use(bodyParser.urlencoded({ extended: true }))
  // Initialize your feathers plugin
  .use('/plugin', plugin())
  .get('/auth/success', handleAuthPopups)
  .use(errorHandler());

app.listen(3030);

console.log('Feathers app started on 127.0.0.1:3030');
```


## License

Copyright (c) 2016

Licensed under the [MIT license](LICENSE).
