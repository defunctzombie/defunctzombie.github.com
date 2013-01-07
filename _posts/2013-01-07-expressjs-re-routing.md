---
title: expressjs re-routing
layout: post
---
One of the most common questions I get asked when I talk about express is how I manage my routes. While I generally point out things like [app.param()](http://expressjs.com/api.html#app.param) and other middleware tricks, I am left just saying that I don't have a great solution. Well, I am happy to say that as of **express 3.0.6** I have a better answer; use the router!

# Routes, why do I care?

Maybe you don't like routers, but they do serve a purpose. If you are writing a web app or even a headless api, you can benefit from the simple separation and organization a nice router will provide. Just think of it as a URL *if* statement on steroids!

The typical express application looks something like this:

```javascript
var app = express();

/// top level routes

app.get('/', function(req, res, next) { ... } );

/// blog post routes

app.get('/post/:post_id', function(req, res, next) { ... });
app.post('/post/:post_id/comment', function(req, res, next) { ... });
app.del('/post/:post_id/comment', function(req, res, next) { ... });

/// etc
```

As the number of routes you want your app to handle grows, you might find that this single file approach gets out of control (justifiably so) and move to a multi-file approach where you export handler functions.

```javascript
var post = require('./routes/post');
app.get('/post/:post_id', post.get);
app.post('/post/:post_id/comment', post.addComment);
app.del('/post/:post_id/comment', post.removeComment);
```

The problem is that your ```routes/post``` file is now void of the actual route information. This can make it a bit more challenging to see what is going on and debug.

# Router to the rescue!

Instead of exporting individual functions from your routes files. You can now use the express router without creating a whole new express app.

Imagine we have the following routes.js file

```javascript
var express = require('express');
var router = new express.Router();

// we can use all of the HTTP verbs on the router
router.get('/bar', function(req, res, next) { ... });
router.post('/baz', function(req, res, next) { ... });

// at the end we export this router as a single unit
module.exports = router;
```

And in app.js

```javascript
var app = express();

// this is for the routes in this file
app.use(app.router);

// this creates /foo/bar and /foo/baz
// /foo could be any prefix
app.use('/foo', require('./routes').middleware);

// our typical app routes can still go here
app.get('/', function(req, res, next) { ... });
```

What we have done is left the relevant path information in the routes.js file and "mounted" those routes under the ```/foo``` url. Now we have basic routes and handlers defined separately from where the parent application may be mounting them.

Note: You could ```app.use(require('./routes').middleware);``` if you wanted to host the routes from the site root.

# Complete app example

I have created a [gist](https://gist.github.com/4472280) which you can clone or review below as a more complete example.

## app.js

Our main app.js (server.js, index.js, whatever.js) file does not change much. The only major difference is that instead of using app.get for our ```/user``` and ```/post``` routes in the file, we use ```app.use``` and inject them under their respective parent urls.

<script src="https://gist.github.com/4472280.js?file=app.js"></script>

## routes/user.js

We now put all of the routes we want to contain as one unit in this file. Notice that this file does not know that it will be served under ```/user```. It could be served from any route by a parent file. Using this approach you could reuse sets of routes in multiple places (not suggesting this is what you want :)

<script src="https://gist.github.com/4472280.js?file=routes/user.js"></script>

## routes/post.js

Like the app, the router has a ```.param()``` function which allows you to consolidate parameter handling. In this example, we see that since all of these routes will need a post, we use the ```:post_id``` parameter from the url to make sure a post is loaded before any route is run.

<script src="https://gist.github.com/4472280.js?file=routes/post.js"></script>

Now go forth and bring order to those routes! Remember that you will need express version ```3.0.6```.
