---
layout: post
title: npm, why do I care?
---

**tl;dr;** npm makes it super easy to share and use javascript code for both client and server environments in more ways than you think. It certainly did for me.

## The beginning

Javascript, while already a popular language, has recently seen an even larger boost in popularity. Most of this (maybe even all) can be attributed to the creation of the [node.js](http://nodejs.org) framework and the community of great modules built up on top of it ([expressjs](http://expressjs.com), [socket.io](http://socket.io), [pg](https://github.com/brianc/node-postgres), [redis](https://github.com/mranney/node_redis)) just to name a few of my favorites. This can be attributed to the fact that [node.js core](http://nodejs.org/api/) itself is very small (could be even smaller) and encouraged developers to instead make [userland modules](https://github.com/joyent/node/wiki/node-core-vs-userland).

Node.js adopted the commonjs require system and as a result created a very simple [module system](http://nodejs.org/api/modules.html#modules_modules). This provided a very simple and consistent way for developers to write server side modules using separated components, smaller files, and no global superobjects. Many modules were even as simple as a single function that did one thing all documented on a simple README and some meta information in a package.json file.

Common.js module syntax is really easy to pickup, looks like normal javascript, and can work in any current javascript environment and browser.

{% highlight javascript %}
// example of what a simple module could look like

// load module 'foobar' cause we want to use it for stuff
var foobar = require('foobar');

// load a local file ./local_file.js relative to this file
var stuff = require('./local_file');

// export some things from our module
module.exports.do_something = function() {
    // use stuff and foobar to do XYZ here
};
{% endhighlight %}

## What do I use to do X?

In the early days (not all that long ago), the easiest way to find modules was to consult the node.js [modules wiki](https://github.com/joyent/node/wiki/Modules) which was a semi-authoritative list of "published" modules. In many cases, installing a module meant cloning the repository locally or some other manual process to copy the files over.

Not long into the life of node, various module "installers" started popping up. NPM (node package manager) grew into one of the more popular such systems due to the ease of use and simple nature. To install a module all you had to do was a simple *npm install \<name\>* and anything "published" to the npm registry would be downloaded, installed, and also have the dependencies installed. All of this would be local to a project which meant no sudo crap, no permissions crap, and no hunting all over your disk for where the modules were installed; they all ended up in a local *node_modules* folder. Why there? Cause that is where they are, done. Stop thinking about it and just look there.

## sharing, not just for kindergarten.

Before npm and the idea of re-usable javascript components being easily shared. It was a disaster to find and use javascript code. Most code out there latched on to some global namespace, made you copy it into your source tree, or was billed as some sort of jquery plugin since apparently everyone was using that already. So they just latched on to the global $ with the hope nothing collides. Basically, easy to install javascript components was not a thing in widespread circulation.

After the ideas behind a javascript registry, suddenly all sorts of javascript libraries started to have a package.json file and were published to npm. Now to try out someone's code, you no longer had to curl, wget, cp or whatever. A simple *npm install* would do! Npm was just the simple tool to download modules from whatever (registry, github, tarbals, etc)

## wait... I can use this in firefox

Since many of the modules in npm were just pure javascript (many having no dependency on even node.js apis), some people started getting the bright idea that they could just re-use the code in the browser. I don't know who published what first, but [@substack](https://twitter.com/substack) created what would become arguably the most well known tool for doing just that, [browserify](https://github.com/substack/node-browserify). Using browserify, you could easily take many modules (or really any code written using commonjs requires) and create a single source file without having to wrap your code manually, deel with global namespaces, or create manual makefiles using *cat* to assemble the final file. For me, this began to completely change how I write and thing about client side code.

This tool really solidified the idea (and selling point) that the same code was being used on the client and server and I could benefit from this without much more effort! Now, I could use some third party code really easily without having to copy it into my source tree, and setup all sorts of global variables and script tags on my pages. No longer did I need to have some crappy makefiles or random global variables just to re-use components between files. I could develop client side software in a reasonable and modular way.

## not all ponies and rainbows

Although it was immediately clear to me that this was a great re-use of modules, there was a problem with packaging some modules automatically. Some modules like [engine.io-client](https://github.com/LearnBoost/engine.io-client) were designed to work on the server and could not be packaged for the client. These packages used hacks like *//if node* comments which were detected by a custom build tool and ignored. This made the code harder to follow and also introduced preprocessor garbage into the codebase. Browserify's solution was no better; it ignored the perfectly valid syntax *(require)('foo')* during the packaging. This meant that just reading the code, you would have no idea that something would be replaced.

This lead to an explosion is nameing shenanigans. A bunch of modules called "*-browserify" or "*-component" or "*-ender" started popping up. And modules like engine.io continued to use the incomprehensible comment style or custom browserify require syntax to ignore specific sections.

There has to be a better way. One that could leverage the existing packaging mechanism everyone was already using without creating confusion.

## package.json

The fundamental thing all of these approaches are trying to solve is the re-use of javascript code on the server and client. When a module is pure js, there is no problem, however, when certain components don't exist on the browser, these hacks come out to play. NO!

All javascript modules have a package.json file accompanying them. This file has some basic information like the module name, version, entry point, and maybe some dependencies (other modules). This is the **perfect** place to educate any sort of "bundler" about what things can't exist on the client. The solution is simple. List which files need replacements.

Lets use the engine.io-client module as an example. The standard module is littered with *//if node* comments. Lets remove all these comments and create "shim" files to replicate the functionality of those modules. Shimming is not a "4 letter word" nor should it cause bad thoughts. Shimming allows you to write consistent and straightforward code and let tooling take care of the rest.

To do this we simple add the following to the [package.json](https://github.com/shtylman/engine.io-client/blob/master/package.json#L28) file.

{% highlight javascript %}
"shims": {
    "xmlhttprequest": "./shims/xmlhttprequest.js",
    "ws": "./shims/ws.js",
    "debug": "./shims/debug.js",
    "./lib/transports/flashws_check.js": "./shims/flashws_check.js"
}
{% endhighlight %}

The above simply says that whenever any sort of browser build tool encounters the "xmlhttrequest" module, it will ignore the one in node_modules and instead use the developer provided file "xmlhttprequest.js" in the shims folder of the source tree. That's it!

The benefit should be immediately obvious. The majority of the javascript code is identical. When reading it, we only need to understand (or duplicate) the interfaces which are being used by our code. When running in node.js, things will run as before. When being packaged for the browser, the shims will be used.

## even better

Since this is something that more and more people are trying to do with modules, it is easy to imagine that the xmlhttprequest module could provide a client side version of itself. Instead of making a separate module, giving it some random xmlhttprequest-<whatever package system> name, it could simply have a "shim" field. In fact, if it had a shim field, our engine.io module could use the xmlttprequest module and not have to shim it itself!

If xmlhttprequest provided a sensible shim for use on the client (not unreasonable and actually very doable), then the engine.io shim section gets even simpler! Since the xmlhttprequest module is in the best position to identify how best to shim itself for the exposed API, it only needs to list the shim field for itself once and everyone benefits!

{% highlight javascipt %}
"shims": {
    "debug": "./shims/debug.js",
    "./lib/transports/flashws_check.js": "./shims/flashws_check.js"
}
{% endhighlight %}

Knowing that javascript code has the cool property of client and server use, you should not shy away from having such fields in the package.json file. It is a natural extension to the capabilities of the system and prevents a bunch of duplicate code, modules, and discovery.

## smoke, magic, and rice

Everything I have said about shimming and packaging client side modules using the shim field is available today! In fact, I use the above concepts using my [script](https://github.com/shtylman/node-script) tool which was designed specifically with this feature in mind. Script will detect the shim field and act appropriately. In fact, this is exactly how I package modules like engine.io or superagent for client side use.

Script ships with a command line tool called "bundle" which can be used in makefiles and such to manually create files. I recommend using the api, or better yet, the middleware to package your files.

For those using connect or express (or other middleware based systems), you can check out [enchilada](https://github.com/shtylman/node-enchilada) which is middleware for serving js files which have been processed through script. This is a simpler way to get started with better packaging for your client side javascript with minimal effort.

{% highlight javascript %}
var app = express();

// serves up all your javascript files under /public, handling all require() calls
// visit any .js route which maps to a file under public and it just works
app.use(enchilada(__dirname + '/public'));
{% endhighlight %}

## light at the end of the tunnel

The moment you switch over to using a commonjs require style syntax for your client side modules, you won't look back! Not only will you have access to countless javascript modules which live in npm (and github), but your code organization will also improve. Client side javascript doesn't have to suck and writing node.js code has shown me that more than ever!

**Let the tools do the work!!** Instead of thinking about javascript as if it was 1990, think about how you can make your life easier! Think about how you write programs in other languages. Modules, files, re-usable and shared components. Javascript and the commonjs syntax provide for that and much more. Any js file you write today can be [instrumented](http://esprima.org/) and delivered however you want! (AMD, inline, commonjs, single massive file, etc...)

## overcrowding, or not?

Coming back to npm for a moment. I want to address an important issue regarding packages and publishing. Once a module is published to npm, you cannot publish your same name module to npm. This NOT THE END OF THE WORLD!! Npm is just a registry. In many cases, if the module is not used you can request the name for your new module. In other cases, you don't need to publish to npm AT ALL.

Not many people know, but npm recognizes a number of different right-hand-side values for dependencies. Lets say the 'emitter' module name is already taken but you really really really want to use that name. Instead of making a module called 'emitter2' or 'emitter-component' or any other random name (assuming you can't make a unique creative name), then just point npm to github!

{% highlight javascript %}
"dependencies": {
    "emitter": "shtylman/emitter#v0.1.0"
}
{% endhighlight %}

npm will understand the above dependency as a github repo and look there. It will grab the code from github under user/project#hash(or tag) location. That's it! You don't have to publish to npm or come up with another name! Please do note that using common names for your components might lead to developer confusion, that is why people come up with different names and maybe you should too!

Oh yea, the location of a dependency can be any git repo (doesn't have to be github). Just specify the full url to the git repo (or tarball) and npm will download that instead. This is great if you have private modules! See the npm [json config](https://github.com/isaacs/npm/blob/master/doc/cli/json.md#dependencies) for valid values.

Using npm for your server and client side packages works great! Just think of npm as the tool that downloads js code and dependencies for your project.

## the future

Start looking at how you write client side javascript TODAY! Stop attaching to global objects. Stop putting everything into gigantic single files. Stop thinking that your source files have to be the same as your "compiled" distributed version. None of these things are true nor do they make for readable javascript code.

Think about npm as just a tool to download modules and dependencies. Consider using some of the alternative ways to create dependencies (remember to always [pin them](/post/pinning-your-hopes/)!) and don't shy away from coming up with a new name or simply using your github. At the end of the day, npm is just about discovering available javascript code and where it lives.

## examples

The following is a short list of various modules which I have refactored to play nice with shims and being packaged for the client. They should all work under script and enchilada. Big thanks to the developers of these useful modules! I use them, that is why I picked on them :)

* [engine.io-client](https://github.com/shtylman/engine.io-client)
* [superagent](https://github.com/shtylman/superagent)
* [jquery-qrcode](https://github.com/shtylman/jquery-qrcode)

Encourage the authors of modules you use to add shims and make their js modules even more **AWESOME!**

