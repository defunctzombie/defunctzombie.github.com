---
layout: post
title: the copy & paste virus
---
>If there is bad code, I will find it, and I will copy it.

Years ago (long before the web 2.0 days) I wrote JavaScript. Then I stopped. These days, I again write a lot of JavaScript. Other people also write a lot of JavaScript and not a single day goes by that someone doesn't write some new jQuery plugin or other interesting JavaScript client library. This is fantastic! All of these developers are solving their problems and sharing the solution with everyone else.

However, when I look back on the JavaScript that I would see someone write years ago and the JavaScript I see being distributed today, I don't see much difference. Sure, some styles and html technologies are better, but I still see everyone delivering JavaScript which all looks like a variation of the following.

{% highlight javascript %}
(function(window) { // or $ depending if it is a jQuery plugin
    var run,
        out,
        find,
        coconut,
        tricycle,
        bus,
        ... 500 other variables ...
        potato,
        tomato,
        yarn,
        dog,
        flag,
        hippo,
        pencil,
        pen;

    function something() {
    };

    ... hundreds of lines of some functions, classes, who knows what all referencing itself

    global.foo = {
        something: function() {
        },
        another: function() {
        },
    }
})(window); // or jQuery again depending
{% endhighlight %}

That's it. One giant file full of code that self references, nests over and over and is contained within a closure just for fun! Why are we doing this? Do we (as developers) hate clarity? Do we enjoy writing a giant closure to do nothing?

Think about how you write code and libraries in other languages. How often do you dump everything into a "mega" file. How often to do put all of it inside one single outer function *just cause*? **NEVER**, that is how often. In other languages you use *import*, *require*, *include*, etc.. so why do we continue to allow our JavaScript to look like unmaintainable shit? No module system.

**WRONG!** There is **NO** reason. JavaScript has a super simple module system and countless people are overlooking it! I am talking about the commonjs require system. Here is all you need to know about commonjs requires: They work, they require no language extensions, and they CAN do what you need. That's it. Everything else is a tooling and education problem. Period.

## commonjs intermission

Since not everyone is familiar with commonjs, let me do a super quick primer with the basics.

* every file is a module
* everything in a file is private to the file unless explicitly exported
* any valid JavaScript type can be exported through **module.exports**
* the **require** function is used to load a module

That's it! Now all you have to do is move some of that code into a separate file and *require* it from your main file. Lets say I am writing a basic client library to do some stuff.

{% highlight javascript %}
// main file for our new client side lib
var parser = require('./parser');
var tree = require('./tree');

// we don't export this, so it is private to the file
var private = 'foo';

// modules can also just export a function
module.exports = function() {
};
{% endhighlight %}

Now we create a *parser.js* and *tree.js* files alongside out main entry file

{% highlight javascript %}
// parser.js
module.exports.parse = function(str) {
};
{% endhighlight %}

{% highlight javascript %}
// tree.js
var Tree = function() {
};

...

module.exports = Tree;
{% endhighlight %}

I have kept the above examples brief specifically to show the super simple nature of the commonjs module system. No complex dependency maps, no global closures, no spaghetti.

## all nails and no hammer

So those of you who are still reading are certainly wondering how you are going to take the above code and make it work in a browser. No browser supports the "require" syntax and in the end you wanted to have one file in your *script* tag. Well, I guess that's that, end of that fun experiment. **NO**. Stop thinking about it as if you don't have a computer in front of you. As a developer, one of the best improvements to productivity can be gained from your tools. The same is true in the above examples. Instead of manually putting your code in a giant closure, wondering what syntax you need for AMD loaders, and what to close over, **LET THE TOOL DO THE WORK**

I am going to say that one more time, since I don't think enough people appreciate this.

#<center>LET THE TOOL DO THE WORK</center>

## the hammer

Today is the day you say **NO MORE** to massive, unorganized repositories of JavaScript files and start using modules. You don't have to add any extra metapackages or publish your code in any special place to do this. To get you started, I have provided a very very basic tool which will build your multi file JavaScript projects called [reunion](https://github.com/shtylman/reunion). I have even provided a makefile snippet to use it when creating the final code bundle for others.

#### use [reunion](https://github.com/shtylman/reunion) in your codebase NOW with minimal effort on your part!

You don't have to dive in 100% from day 1. If you have a jQuery plugin, you can still use that wonderful global $ **var $ = window.jQuery** in your module works!. However, these first steps will make your code that much easier to follow and eventually you will be able to *require('jQuery');* just like anything else!

## no overhead

Reunion require functionality is **TINY**. The minified require function comes in at ~140 bytes! Since reunion puts everything into a single closure for you, you still benefit from excellent minification. Even more, each module is put into a function which means it can be minified locally and allows for the minifier to reuse more of the shorter variable names.

It wasn't until I started writing node.js code and modules that I realized how stupid front end file organization and usage was and cotninues to be for so many people. As soon as I started writing modules and separating code, JavaScript got that much easier and maintainable. Tests are easier to localize and common components can be shared much easier. More importantly, given the right tool, I could package any module however I wanted!

What about other loaders? We are back to a tooling issue. The great thing about commonjs requires is that they are super easy to tool around! If you want AMD output, just send me a pull request for and AMD target. That's it! Now only one place has to worry about the right closure format. Everyone else just has simple and clean JavaScript!

## the future

Reunion is a very simple tool. In the next post, I am going to talk about tools like [npm](http://npmjs.org), [script](https://github.com/shtylman/node-script), and [browserify](https://github.com/substack/node-browserify) and how they can completely revolutionize how you develop and share JavaScript.

And remember! **LET THE TOOL DO THE WORK!**
