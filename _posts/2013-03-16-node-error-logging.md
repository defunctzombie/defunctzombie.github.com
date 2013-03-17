---
title: node.js error logging
layout: post
---
One of the first things you should do for any application you plan to deploy is make sure you have proper error logging in place. You should know when things are failing and should be able to act on failure information.

Lets start with a basic node application that might have an error

## inflexible error logging

```javascript
function will_error(cb) {
    return cb(new Error('foobar'));
}

will_error(function(err) {
    if (err) {
        return console.log(err);
    }
});
```

It is clear that when we call `will_error`, our app will get back an error that we should handle. By just using `console.log` as we have done, we are losing valuable error information.

Not only are we losing error information to help in debugging, but we are also not doing anything to alert ourselves to potential problems our app is having. The first step to better error information is to introduce a basic logging library.

## basic logging layer

One step in the right direction is to use a simple logging library like [book](https://github.com/shtylman/node-book). Book will become the point of capture for our log entries allowing us to make use of them in a more flexible manner.

```javascript
var log = require('book');

function will_error(cb) {
    return cb(new Error('foobar'));
}

will_error(function(err) {
    if (err) {
        return log.error(err);
    }
});
```

Now, instead of logging to console, we are logging through `book` which provides a few log events and also logs to console for us. The application is identical to what we had before, except now we can leverage our logging library to get more information from our errors.

## introduce an alerting solution

Lets say we want to leverage an alerting service like [sentry](http://getsentry.com/welcome/) to know when things are breaking. We can easily do this when we use a simple logging layer.

The [bookrc](https://github.com/shtylman/node-bookrc) module allows us to educate our book logging layer without affecting application code.

Simply alter the ```require('book')``` line to ```require('bookrc')``` and put the following file in the root of your project.

<script src="https://gist.github.com/5178667.js?file=bookrc.js"></script>

Now when we run our application and an error happens, we will be alerted via the sentry web interface.

<center>
[![sentry](/img/post/node-error-logging/sentry.png)](https://app.getsentry.com/dummy/dummy/group/3885521/)
</center>

This provides us with a lot more information about the specific error.

* error level
* stacktrace with relevant code blocks
* location of logging statement
* git commit id
* loaded modules

All of this information is valuable when tracking down errors in deployed applications and we got it all for free without having to change our application code!

### how it works

The `bookrc` module simply looks for a `bookrc.js` file and loads that as itself. What we put into this `bookrc.js` file is completely up to us. I could have easily added logging to a file, additional alerting or other filters to the `book` logger we created. Book logging is very flexible and can easily be extended with many different plugins.

## further reading

If you are using the [express](http://expressjs.com/) web framework. Check out the [connect-raven](https://github.com/shtylman/node-connect-raven) module which provides error middleware for sentry logging. Proper express/connect error logging is a post in itself.

See [gist/5178667](https://gist.github.com/shtylman/5178667) for some additional examples

## Now go and purge those useless console.logs!



