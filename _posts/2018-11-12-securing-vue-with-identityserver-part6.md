---
layout: post
title: Securing a Vue.js app and API with IdentityServer - Part 6
date: '2018-11-12T11:30:00.001+10:00'
author: Richard Banks
---
This is part 6 of a series showing you how to secure a Vue.js app with IdentityServer and call an ASP.NET Core Web API.

  * [Part 1 - Overview and Solution Structure](/2018/11/securing-vue-with-identityserver-part1.html) 
  * [Part 2 - Creating and Configuring your IdentityServer](/2018/11/securing-vue-with-identityserver-part2.html)
  * [Part 3 - Adding Google Authentication to IdentityServer](/2018/11/securing-vue-with-identityserver-part3.html)
  * [Part 4 - Creating and securing an ASP.NET Core Web API](/2018/11/securing-vue-with-identityserver-part4.html) 
  * [Part 5 - Creating the Vue.js client](/2018/11/securing-vue-with-identityserver-part5.html)
  * Part 6 - Calling an HTTP API from Vue.js (this post)
  * [Part 7 - Securing a router view in Vue.js](/2018/11/securing-vue-with-identityserver-part7.html)
  * [Part 8 - Calling a secured API from Vue.js](/2018/11/securing-vue-with-identityserver-part8.html)
  * [Part 9 - Refreshing identity tokens with Vue.js](/2018/11/securing-vue-with-identityserver-part9.html)

> __Note:__ _You can find the source code for this post series on [GitHub](https://github.com/rbanks54/vue-and-identityserver)._

## Calling An API from Vue.js

Before we call our secured API, let’s just make sure we can call our unsecured `/api/values` API first.

The default vue app we generated has a simple `About` page, so we’ll just repurpose that page instead of adding any new routes to our app.

To make our API calls we’re going to use the [Axios](https://github.com/axios/axios) library.

From your command line, ensure you’re in the vue-app folder (i.e. where your JS client source is) and then run 
```
yarn add axios
```

Next up, let’s edit the `src/views/About.vue` component to display the values return from the `/api/values` endpoint.

Change the template to be:
```html
<template>
  <div class="about">
    <h1>This is an about page</h1>
    <div v-for="(item,index) in values" :key="index">
      <p>{{item}}</p>
    </div>
    <button @click="callApi">Call API</button>
  </div>
</template>
```

And then add a script section to the component (in the same file) for responding to the button click event

```js
<script>
import axios from 'axios'

export default {
  data() {
    return {
      values: ["no data yet"],
    }
  },
  methods: {
    async callApi() {
      try {
        const response = await axios.get("https://localhost:5000/api/values");
        this.values = response.data;
      } catch (err) {
        this.values.push("Ooops!" + err);
      }
    }
  }
}
</script>
```

Save the file. If you enabled the watch option in your build task you should now be able to switch to the browser and refresh the page.

Re-build the app (as explained in post 5) navigate to the About page, click the `Call API` button. You should see the following:

![results from unsecured API call](/assets/images/2018-11/unsecured_results_view.png)

Sure, it's not the most exciting thing in the world but we’ve proven that we can call our backend correctly and that everything works as expected. When doing software development it's always good to build something small and then iterate on it until it's complete.

Up Next: [Part 7 - Securing a router view in Vue.js](/2018/11/securing-vue-with-identityserver-part7.html)