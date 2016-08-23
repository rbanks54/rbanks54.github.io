---
layout: post
title: Jekyll on Bash on Ubuntu on Windows
date: '2016-08-09T12:00:00.000+10:00'
author: Richard Banks
tags: 
modified_time: '2016-08-09T12:00:00.000+10:00'
---
After more than 10 years I've decided to move my blog. I've been using [Blogger](http://blogger.com) for all that time but over recent years it feels like Blogger has fallen in to disuse and neglect, while other platforms have arisen and seem to work wonderfully well.

The main reason for not moving has been inertia. Plain and simple. I've written over 600 posts and the thought of manually hacking away at them until they're looking right on whatever new platform I'd use always felt way too big, and way too time consuming. That said, I decided to bite the bullet and I switched over to using GitHub Pages, which runs Jekyll under the hood.

This post isn't really about that move though (or the fact that I'm just going to leave my old posts as is until someone complains). It's about the experience of using Windows 10's new "Windows Subsystem for Linux" (WSL) for more than just kicking the tyres; using it for actual work.

## Tell me more

I won't explain the steps to set up the Bash shell in Windows here, but it is pretty simple. Just follow [the installation guide](https://msdn.microsoft.com/en-us/commandline/wsl/install_guide) from Microsoft and you should be fine.

I also need to give a lot of credit to [Dave Rupert](http://daverupert.com) who put together a handy little guide on how to get [Jekyll working on BoUoW](http://daverupert.com/2016/04/jekyll-on-windows-with-bash/) (Bash on Ubuntu on Windows). Without that, there's a few things I would've been stuck on given the docs for the Linux subsystem are still "patchy".  

So now I could run a Jekyll build locally, but given I was importing my Blogger content I needed to run [jekyll-import](http://import.jekyllrb.com/docs/blogger/) to get everything pulled across. Sadly, it didn't work. Well, not straight away.

I had errors related to `zlib` and various other missing pieces. Naively, I tried using `apt-get install zlib` but no packages could be found and I also had errors when I tried using `bundler` and `nokogiri`.

A bit more noodling about, poking at Google and crossing my fingers and I ran across another post by Dave Rupert where he talked about getting [Ruby on Rails running on BoUoW](http://daverupert.com/2016/06/ruby-on-rails-on-bash-on-ubuntu-on-windows/). In it he showed how to install the zlib library (it's zlibc, doh!) and handling a specific problem with nokogiri and how it references it's dependencies. The trick was the `--use-system-libraries` flag when manually installing the gem.

``` bash
gem install nokogiri -- --use-system-libraries
```

That little gem right there (sorry, bad pun intended) was just what I needed to get things clicking and for the jekyll-import script to run successfully. Happy days!

Since then it's been a pretty simple case of running jekyll build and jekyll serve from within the Bash prompt for testing and working with my site content and layout.

It's been surprisingly easy and now that things are working I'm basically just rotating between the two Jekyll commands to see the progress of my changes. And before you ask, there's a limitation that stops file change notifications being visible in the Bash shell, and this is turns makes the jekyll file watcher crash. 

``` bash
jekyll b --incremental
jekyll serve --no-watch --skip-initial-build 
```

For the jekyll people out there, yes, I can combine those two commands into one. It's just been during the migration that I've been running them as two steps. 

Oh - a quick update. You can actually run jekyll in watch mode. Just make sure you use the `--force_polling` flag as shown here: 

``` bash
jekyll serve --force_polling --incremental 
```
![bash console](/assets/images/2016-08-11_jobouow.jpg)


Finally, when I'm happy with my changes, I simply switch over to my usual windows console and do a git commit and push from there without a problem.

Oh, a few things I noted from using the Bash prompt on Windows:

* Running bash hosted inside ConsoleZ causes some strange behaviours. Cursor up doesn't show the previous command so I've had to use `Ctrl`+`P` instead.
* Similarly Ctrl+C doesn't send the break signal inside ConsoleZ (e.g. to stop the Jekyll server).  I had to use `Ctrl`+`Shift`+`C` instead.
* Paste doesn't work with Ctrl+V as expected either. We're back to mouse based pasting for now (it's a beta so these things are expected I suppose).

Other than that, I'm really pleased and just how well this is working. Well done to both Microsoft and Canonical/Ubuntu for making this happen!