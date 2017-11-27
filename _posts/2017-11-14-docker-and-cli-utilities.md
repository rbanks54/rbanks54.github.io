---
layout: post
title: Using docker to avoid installing cli utilities
date: '2017-11-14T12:00:00.001+10:00'
author: Richard Banks
modified_time: '2017-08-15T12:00:00.001+10:00'
---

With Docker, and containers in general, becoming more and more prevalent I'm finding there's no longer a compelling reason to install command line utilities anymore. Instead, I now look for docker images containing a CLI utility I want to use, and I simply pull that image locally and run a container for it instead.

Why? A few reasons spring to mind. The first is the isolation that a container provides, giving me confidence that I'm limiting my exposure a utility that misbehaves. The other is the ability to always run the latest version without worrying about locally upgrading, and having the flexibility to run older versions side by side with the latest if I so choose.

As an extra bonus, it also means I am effectively able to commit the specific CLI utilities I need for a project into source control without needing to commit any binaries, or worry about developers all having the same versions of a utility installed.

Of course, there's tradeoffs to this approach. Invocing CLI utilities via a `docker run` command is not as easy, and there's a very minor overhead in spinning up a container just to run a utility, but I view these as minor hindrances for the benefits gained.

### A simple example ###

For example I wrote a post last year on [using Jekyll on the Windows Subsystem for Linux (WSL)](/2016/08/jekyll-on-bash-on-ubuntu-on-windows.html), which I using to check my blog locally before pushing it to GitHub Pages. And now? Well, I now use a GitHub Pages images and run a container to check my blog:

```
docker run -t --rm -v "%CD%":/usr/src/app -p "4000:4000" starefossen/github-pages
```

To make it simpler, I've put that command in a .cmd script file and checked it into git so I have it no matter what machine I'm editing my blog on.

![github pages docker script](/assets/images/2017-11/docker-jekyll.jpg)

### What about a JavaScript build environment? ###

Another thing I've wanted to do for a while is eliminate the local installs of node, npm and yarn. I really don't like the hassle of checking if I have the right versions of node each time I want to work on a project. It's so much easier to just pull a docker image for node/npm/yarn and run a batch file to execute a command in a container.

And now I can! No more local node installs, and instead I just use the `node:alpine` container image from the [docker store](https://store.docker.com/images/node) with some local command scripts to save typing out `docker run` commands. Here's one for running yarn:

```
docker run -it --rm -v "%CD%":/usr/src/app -w /usr/src/app node:alpine yarn %*
```

I created simple variations of the script for running node and npm as well, and put all scripts in the root of my source tree (and checked them into source control)

Once that's done, things will work pretty much as you'd expect. I just need to make sure I'm executing my commands from the appropriate folder since they're not on my path.

### Example with Vue.js ###

Let's see how we'd use the node image to build the artifacts for a (very) small [Vue.js](https://vuejs.org) app using webpack. Don't worry too much about the vue specifics, it's more to show you how a _"use docker images as cli tools"_ approach works:

First, we start with a basic `index.html` for our vue app to run in:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Docker all the things</title>
  </head>
  <body>
    <div id="app"></div>
    <script src="./dist/build.js"></script>
  </body>
</html>
```

Then we add a `./js` folder for our source and create an `app.vue` file in it. 

``` html
<template>
        <p>{{ "{{ message "}}}}</p>
</template>

<script>
    export default {
        data() {
            return {
                message: 'Hello Vue.js!'
            }
        }
    }
</script>
```

The we add a `main.js` file where we'll have our vue entry point.

``` js
import Vue from 'vue'
import App from './app.vue'

new Vue({
    el: '#app',
    render: h => h(App)
})
```

OK. Basics in place. Now I want to use webpack to process the app.vue template, run babel.js over the js files, bundle all of it together into a single distributable file, minify the result, produce a source map for debugging and place the output in a `./dist` folder.
I won't put the webpack.config.js file in here as it would make the post a little long, and webpack configuration is too much of a sidetrack given we're talking about docker containers.

Regardless of our configuration, when using webpack the normal approach is to install it as a globally avaiable command using

```
npm install -g webpack
```

But a global install doesn't make sense, especially since I no longer have node installed and when the container I run node from gets destroyed as soon as the command completes.

We need to do a local install instead.

```
npm install --only=dev webpack
```

And in my `package.json` file I add a line to the scripts section so I can call it using `npm run`.

``` json
  "scripts": {
    "webpack": "webpack",
    "watch": "webpack --watch"
  } 
```

When I want to execute webpack I can now do one of two things. I could call `npm run webpack` from the command line or create a `webpack.cmd` file as follows 

```
npm run webpack %*
```

and just run `webpack` from the CLI like I normally would.

Creating a cmd file just to wrap the `npm run` command may feel like overkill, but a number of code editors that have the ability to call webpack assume it is available as a command.  The cmd file helps in those situations. For me, it's when I use Visual Studio's Web Essentials Task Runner.
![dockerised webpack inside visual studio](/assets/images/2017-11/webpack_and_vs.jpg)

The _only_ downside of this approach is that the file watcher in the container doesn't detect changes, so I need to manually trigger the rebuild rather than letting the file watcher kick things off for me. It's a minor annoyance but not a big deal for me, personally, but you might not like it.

_Technical aside:_ The file system watcher should work once the new Linux Containers on Windows (LCOW) feature matures a bit more. It's currently an experimental feature in Docker foor Windows and at the time of writing there's a bug with permissions/chmod in LinuxKit that stops npm installing dependencies properly which means I can't get it working just yet.

Long story, short? I'll continue to look for ways to run command line utilities via containers, and even in more complex use cases like a javascript build chain, it's not hard to get things working with minimal effort.