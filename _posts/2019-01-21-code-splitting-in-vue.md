---
layout: post
title: Code splitting in Vue
date: '2019-01-21T08:30:00.000+11:00'
author: Richard Banks
---
I have a small personal project I'm building with Vue and Webpack v4. I noticed that the production bundle size was starting to get a bit large (3.4 Mb).

A bundle of this size will be slow to load and people on slow connections might think the web app is failing to load and leave the site.

Time to do some code splitting.

If you've not heard of the term before, code splitting aims to reduce the size of the JavaScript loaded for a site to just the code needed to serve up the initial view. Any JavaScript needed after that is loaded asynchronously, and on demand.

With Vue this is really quite easy!

How easy, you ask? Let's have a look.

### Route splitting
Since we want the initial bundle to only contain code need for loading the site, we don't need any of code used elsewhere in our app. Splitting out components loaded only when people navigate elsewhere is perfect for this.

To given you an example, here's the extent of my changes needed to split out an admin component into a separate bundle:

``` js
// Original code
import AdminContainer from 'pages/AdminContainer'

// Changed to
const AdminContainer = () => import(/* webpackChunkName: "admin-container" */ 'pages/AdminContainer');
```

And that comment for the `webpackChunkName`? It's not even required! That's just so I can name the bundle the way I want.

And that's it!
A one line change and we're all done. Wow!

### OK, Not quite
Yes. I lie a little. I also had to add this single to my `.babelrc` file to handle the dynamic import statement:

``` js
  "plugins": ["@babel/plugin-syntax-dynamic-import"]
```

Pretty cool, right?

### It's not just for Vue components

I use [PdfMake](http://pdfmake.org) to create PDF files in the browser, and have put all the code related to generating PDFs in a module I named `pdfGenerator` (I know, so creative!).

Instead of having all the PDF generation code in my initial bundle, I can split it out into it's own bundle and only load it on demand.

We can still use the dynamic import statement, but we have to keep in mind that the dynamic import returns a promise, and when the promise resolves our callback will be passed the module that was loaded. 

So, my code for generating a PDF now becomes something like this

``` js
const pdfGenerator = () => import('@/services/pdfGenerator');

export default {
  //.. rest of Vue component
  methods: {
    pdfGenerator().then(module => {
        var generate = module.default; //the default export from the imported module
        generate(this.someValue, this.someOtherValue);
    }
  }
};
```

It's a little more code, but still really easy to use dynamic imports and the approach means we can separately load JavaScript needed on a function specific basis, not just when Vue components are loaded.

### Conclusion
By using just some basic route-based splitting and delaying the loading of the PDF functionality until it's needed, I turned what was a 3.4 Mb bundle into a 1.2 Mb bundle in about 20 minutes of work, including testing time.

I'm pretty happy with that, and there's still more I can do to further optimise the initial bundle size and make the load time of my web app even quicker.