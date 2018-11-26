---
layout: post
title: Securing a Vue.js app and API with IdentityServer - Part 9
date: '2018-11-12T13:00:00.001+10:00'
author: Richard Banks
---
This is part 9 of a series showing you how to secure a Vue.js app with IdentityServer and call an ASP.NET Core Web API.

  * [Part 1 - Overview and Solution Structure](/2018/11/securing-vue-with-identityserver-part1.html) 
  * [Part 2 - Creating and Configuring your IdentityServer](/2018/11/securing-vue-with-identityserver-part2.html)
  * [Part 3 - Adding Google Authentication to IdentityServer](/2018/11/securing-vue-with-identityserver-part3.html)
  * [Part 4 - Creating and securing an ASP.NET Core Web API](/2018/11/securing-vue-with-identityserver-part4.html) 
  * [Part 5 - Creating the Vue.js client](/2018/11/securing-vue-with-identityserver-part5.html)
  * [Part 6 - Calling an HTTP API from Vue.js](/2018/11/securing-vue-with-identityserver-part6.html)
  * [Part 7 - Securing a router view in Vue.js](/2018/11/securing-vue-with-identityserver-part7.html)
  * [Part 8 - Calling a secured API from Vue.js](/2018/11/securing-vue-with-identityserver-part8.html)
  * Part 9 - Refreshing identity tokens with Vue.js (this post)

> __Note:__ _You can find the source code for this post series on [GitHub](https://github.com/rbanks54/vue-and-identityserver)._

## Refresh tokens

You’ll find that if you leave the Vue app running for long enough that eventually the identity token will expire and you’ll need to sign in again.

If this happens too regularly users will complain of a poor user experience and get a bit annoyed with your application. Let's have a look at how we can configure our application to automatically refresh the identity token once it gets close to expiring.

Ideally, we don't want our users to know that the token has been refreshed. We want  token renewal to occur without interrupting the user experience or breaking the behaviour of the application in any way.

The `oidc-client` provides this functionality for us via its automatic silent renewal feature.

Let’s add this to our code by adjusting the configuration of the `UserManager` in `services/security.js`.

Add the following properties where we create the UserManager object

```js
    automaticSilentRenew: true,
    silent_redirect_uri: 'https://localhost:5000/static/silent-renew.html',
    accessTokenExpiringNotificationTime: 10,
```

You'll note that `silent-renew.html` is a static page, not a Vue component. This is to avoid the overhead of creating a new Vue instance, and to limit any risk of interrupting work happening in the main application. The `silent-renew.html` page is simply a callback to complete the token renewal process and we're only using it to update our token stored in the browser's local storage. We don't need to update and any of Vue’s state information.

Now, because we’re changing the client to support silent renewal, we need to adjust the client registration in IdentityServer as well.

We're going to shorten our token lifetime to 90 seconds to avoid waiting 2 hours for tokens to expire, and we'll also add a new `RedirectUri`, enable `AllowOfflineAccess` and disable the `RequireConsent` flag. For this post we’ll also be using a sliding expiration approach so that as long as the tokens keep getting regularly refreshed, you’ll remain logged in.

Here’s the extra client configuration code you need in `Config.cs` of your `IdentityServer` project

```cs
  AllowOfflineAccess = true,
  AccessTokenLifetime = 90, // 1.5 minutes
  AbsoluteRefreshTokenLifetime = 0,
  RefreshTokenUsage = TokenUsage.OneTimeOnly,
  RefreshTokenExpiration = TokenExpiration.Sliding,
  UpdateAccessTokenClaimsOnRefresh = true,
  RequireConsent = false,

  RedirectUris = {
      "https://localhost:5000/callback",
      "https://localhost:5000/static/silent-renew.html"
  },
```

Back in our `vue-app`, we now need to implement the silent-renew callback page. We'll need to include the `oidc-client` and a simple call to trigger the callback completion method.

Create a static folder in `/public`, and then create a `/public/static/silent-renew.html` page with the following content.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Silent Renew Token</title>
</head>
<body>
    <script src='oidc-client.min.js'></script>
    <script>      
        console.log('renewing tokens');
        new Oidc.UserManager({userStore: new Oidc.WebStorageStateStore({ store: window.localStorage })})
            .signinSilentCallback();
    </script>
</body>
</html>
```

Since this is static content and we're not building anything with Webpack, we need to copy `oidc-client.min.js` from `vue-app\node_modules\oidc-client\dist\oidc-client.min.js` into the `/public/static` folder.

If you haven’t already done so, build and restart both your identity server and your vue-app sites.

Browse to the `/about` page and ensure you can still make the secure API call.

Now you'll just have to wait 90 seconds to see if your token refreshes. You should be able to see network activity using your browser’s developer tools, and you should see the `"renewing tokens"` message in the console.
 
![token renewal](/assets/images/2018-11/token_renewal.png)

## Conclusion

That's it! We're done!

I hope you've found this series useful and that it helps you in your development efforts!

If you have any feedback, reach out and let me know. Also, if you notice any bugs or issues with the code, head over to GitHub project and let me know there.