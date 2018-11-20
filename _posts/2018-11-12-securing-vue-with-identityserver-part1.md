---
layout: post
title: Securing a Vue.js app and API with IdentityServer - Part 1
date: '2018-11-12T09:00:00.001+10:00'
author: Richard Banks
---
People regularly ask me how authentication should be performed when calling a secured HTTP API from a Single Page Application.

This blog series provides a worked example, from beginning to end, showing you how to build a SPA with Vue.js, connect it to a HTTP API built with ASP.NET Core and to secure it with IdentityServer v4.

Because you're busy and you might not want to read the entire series just to solve that one small problem you have, here's what we'll cover in each part.

  * Part 1 - Overview and Solution Structure (this post)
  * [Part 2 - Creating and configuring your IdentityServer](/2018/11/securing-vue-with-identityserver-part2.html)
  * [Part 3 - Adding Google authentication to IdentityServer](/2018/11/securing-vue-with-identityserver-part3.html)
  * [Part 4 - Creating and securing an ASP.NET Core Web API](/2018/11/securing-vue-with-identityserver-part4.html)
  * [Part 5 - Creating the Vue.js client](/2018/11/securing-vue-with-identityserver-part5.html)
  * [Part 6 - Calling an HTTP API from Vue.js](/2018/11/securing-vue-with-identityserver-part6.html)
  * [Part 7 - Securing a router view in Vue.js](/2018/11/securing-vue-with-identityserver-part7.html)
  * [Part 8 - Calling a secured API from Vue.js](/2018/11/securing-vue-with-identityserver-part8.html)
  * [Part 9 - Refreshing identity tokens with Vue.js](/2018/11/securing-vue-with-identityserver-part9.html)

> __Note:__ _You can find the source code for this post series on [GitHub](https://github.com/rbanks54/vue-and-identityserver)._



## Overview

Our sample application will be calling an API and displaying the data returned. It's deliberately simple so you can focus on the authenticatation of users and how to manage the identity tokens.

The end-to-end flow in the final application is: 
* A person browses to our web site to load the Vue.js JavaScript client.
* They trigger the sign-in process by navigating to a secured page.
* They authenticate with IdentityServer using a username/password or Google sign in. 
* The Vue.js client calls our secured API with the person's identity token, and shows the result in the browser.
* The Vue.js client automatically refreshes the identity token so that our person won’t have to keep logging in at regular intervals.

The solution has 3 main components

1. __IdentityServer__: Our secure token server (STS). It runs using ASP.NET Core and will be configured for username/password and Google authentication.
1. __Web Server__: Our ASP.NET Core Web API. It will also be serving the static content for the Vue.js web application.
1. __Single Page Application__: Our Vue.js client application. We’ll use Axios to make the API calls.

Enough with the preamble. Let’s get started!

## Solution Setup

Since we don't want to tightly couple the deployment of our authentication service with the deployment of the HTTP API, we'll have two projects. One for the IdentityServer and one for our ASP.NET Core Web API.

We're going to start by creating a new ASP.NET Core Web API project in Visual Studio called `VueApi`.

![new web API project](/assets/images/2018-11/new_web_api_project.png)
 
Ensure you create an API project with _No Authentication_ and with _support for HTTPS_. What’s the point of securing an API if you transmit all your data in plain text, huh?

![new web API project details](/assets/images/2018-11/new_web_api_project_details.png)

And... That’s it for now! The default project template provides a `Values` controller that's sufficient for our initial needs.

It's time to move to the next step.

__Up Next:__ [Part 2 - Creating and Configuring your IdentityServer](/2018/11/securing-vue-with-identityserver-part2.html)