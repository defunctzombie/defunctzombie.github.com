---
title: npm - git with the program
layout: post
---
[npm](https://npmjs.org) is a simple package manager for javascript projects. All it takes to use npm for your project is a simple file called ```package.json```. This file contains some basic metadata about your project (similar to setup.py for pip, gemspec for ruby, etc).

Example package.json file

```json
{
    "name": "awesome-package-name",
    "version": "0.0.0",
    "description": "the answer to life and the universe",
    "main": "./entry-file.js"
}
```

Thats all you need for a basic description of your project. If you have a public project and are hosting it publicly, I also suggest adding a repository field so others can find your code.

```json
{
    "repository": {
        "type": "git",
        "url": "git://github.com/shtylman/awesome-package-name.git"
    }
}
```

For versioning, I suggest you take a quick look at [semver](https://github.com/mojombo/semver/blob/master/semver.md)

## dependencies

Now, if your module is awesome (and there is no reason for it not to be), then you will most likely use other modules. Fortunately, this is really easy and all you need to add is a ```dependencies``` field.

```json
{
    "dependencies": {
        "mime": "1.2.7",
        "yummy": "1.0.1"
    }
}
```

I recommend using [exact versions]({% post_url 2012-02-27-pinning-your-hopes %}) so when you revisit your code months later, you can easily identify what versions you had installed and the module worked with.

Now you can just run ```npm install``` and npm will fetch the exact versions of the dependencies specified.

If you want to add more dependencies later, just edit the file. ```npm ls``` can show you what you have installed but not added (extraneous).

## where does git fit in?

The above dependency specification should look very familiar to anyone currently writing javascript modules and publishing them to npm. What many don't realize is that you can also specify git repositories!

```json
{
    "dependencies": {
        "dom": "git://github.com/shtylman/dom.git"
    }
}
```

Now when you ```npm install```, npm will fetch the latest version of the master branch from the git repo you specified!

If you want a specific tag or commit just specify a ```#``` followed by the tag or commit id.

```json
{
    "dependencies": {
        "dom": "git://github.com/shtylman/dom.git#v0.0.1"
    }
}
```

When specifying a commit sha1, you don't have to specify the whole sha. I have found that using just the first 6 characters is good enough as it will most likely be unique in your repo.

```json
{
    "dependencies": {
        "dom": "git://github.com/shtylman/dom.git#449852"
    }
}
```

## but wait! there's more!

As of node v0.8.14, you can make your github urls even shorter!

```json
{
    "dependencies": {
        "dom": "shtylman/dom#v0.0.1"
    }
}
```

Note that just like the long form url, you can specify a ```#commit```. So if you can't publish to npm, or have some other naming issue, you can easily reference code which lives on github.

*Keep in mind that the shortform syntax does not work on npm < 1.1.64. So if you have an older version of node, you will need to make sure your npm is up to date with ```npm install -g npm```*

## use cases and wrap-up

* works with users or organizations! the format is simply ```owner/repo#commit```
* private repos work great with the git approach so long as the proper ssh keys are setup in github
* you don't have to ```npm publish``` to use a module
* if your module name is already taken and you are not creative enough to make a new one
