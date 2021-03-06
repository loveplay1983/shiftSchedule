# Understanding CORS

[![Bartosz Szczeciński](https://miro.medium.com/fit/c/56/56/1*4LUSy4jH_AaeVgBsXKEorA.png)](https://medium.com/@baphemot?source=post_page-----18ad6b478e2b--------------------------------)

[Bartosz Szczeciński](https://medium.com/@baphemot?source=post_page-----18ad6b478e2b--------------------------------)

[Jan 28, 2018·5 min read](https://medium.com/@baphemot/understanding-cors-18ad6b478e2b?source=post_page-----18ad6b478e2b--------------------------------)





![img](https://miro.medium.com/max/60/1*AAtbKMYYz5wgxed7Tu6tzw.png?q=20)

![img](https://miro.medium.com/max/1258/1*AAtbKMYYz5wgxed7Tu6tzw.png)

“OK, but no”

If you ever worked with an AJAX call, you are probably familiar with the following error displayed in browser console:

![img](https://miro.medium.com/max/60/1*U2MpZQXPI-RW2JASIoeaFQ.png?q=20)

![img](https://miro.medium.com/max/1271/1*U2MpZQXPI-RW2JASIoeaFQ.png)

Failed to load https://example.com/: No ‘Access-Control-Allow-Origin’ header is present on the requested resource. Origin ‘[https://anfo.pl'](https://anfo.pl'/) is therefore not allowed access. If an opaque response serves your needs, set the request’s mode to ‘no-cors’ to fetch the resource with CORS disabled.

If you see this message it means the response failed yet you are still able to see the returned data if you go to the Network tab — what’s the idea here?

# Cross-Origin Resource Sharing (CORS)

The behavior you are observing is the effect of browsers CORS implementation.

Before CORS became standarized there was no way to call an API endpoint under different domain for security reasons. This was (and to some degree still is) blocked by the Same-Origin Policy.

CORS is a mechanism which aims to allow requests made on behalf of you and at the same time block some requests made by rogue JS and is triggered whenever you are making an HTTP request to:

- a different domain (eg. site at [example.com](http://www.example.com/) calls [api.com)](http://www.api.com)/)
- a different sub domain (eg. site at [example.com](http://www.example.com/) calls api.example.com)
- a different port (eg. site at [example.com](http://www.example.com/) calls [example.com:3001)](http://www.example.com:3001)/)
- a different protocol (eg. site at [https://example.com](https://example.com/) calls [http://example.com)](http://example.com)/)

This mechanism prevents attackers that plant scripts on various websites (eg. in ads displayed via Google Ads) to make an AJAX call to [www.yourbank.com](http://www.yourbank.com%2C/) and in case you were logged in making a transaction using *your* credentials.

If the server does not respond with specific headers to a “simple” `GET` or `POST` request — it will still be send, the data still received but the browser will not allow JavaScript to access the response.

If your browser tries to make a “non simple” request (eg. an request that includes cookies, or which `Content-type` is other than `application/x-ww-form-urlencoded`, `multipart/form-data` or `text-plain`) an mechanism called **preflight** will be used and an `OPTIONS` request will be sent to the server.

An common example of “non simple” request is to add cookies or custom headers — it your browser sends such a request and the server does not respond properly, only the preflight call will be made (without the extra headers) but the actual HTTP request the brower meant to make will not be sent.

# Access-Control-Allow-What?

CORS uses a few HTTP headers — both in request and response — but the ones you must understand in order to be able to continue working are:

## Access-Control-Allow-Origin

This header is meant to be returned by the server, and indicate what client-domains are allowed to access its resources. The value can be:

- `*` — allow any domain
- a fully qualified domain name (eg. [https://example.com)](https://example.com)/)

If you require the client to pass authentication headers (e.g. cookies) the value **can not** be `*` — it must be a fully qualified domain!

## Access-Control-Allow-Credentials

This header is only required to be present in the response if your server supports authentication via cookies. The only valid value for this case is `true`.

## Access-Control-Allow-Headers

Provides a comma separated list of request header values the server is willing to support. If you use custom headers (eg. `x-authentication-token` you need to return it in this ACA header response to `OPTIONS` call, otherwise the request will be blocked.

## Access-Control-Expose-Headers

Similarly, this response should contain a list of headers that will be present in the actual response to the call and should be made available to the client. All other headers will be restricted.

## Access-Control-Allow-Methods

A comma separated list of HTTP request type verbs (eg. `GET`, `POST`) which the server is willing to support.

## Origin

This header is part of the request that the client is making, and will contain the domain from which the application is started. For security reasons browsers will not allow you to overwrite this value.

# How to fix the CORS “error”?

You have to understand that the CORS behavior is **not an error** — it’s a mechanism that’s working as expected in order to protect your users, you, or the site you’re calling.

Sometimes the lack of proper headers is result of wrong client-side implementation (eg. missing authorization data such as API key).

There are a few ways to “fix the error” depending on the scenario you’re facing:

**A — I’m developing the frontend and have control of or know the person developing the backend**

This is the best case scenario — you should be able to implement the proper CORS response on the server which you’re calling. If the API is using `express` for node you can use the simple `cors` package. If you want to make your site properly secure, consider using a whitelist for the `Access-Control-Allow-Origin` header.

**B — I’m developing the frontend but have no control of the backend now, I need a temporary solution**

This is the second-best scenario, because it’s just the A one, but with some time constrains. To temporary fix the issue you can make your browser ignore CORS mechanism — for example use the [ACAO](https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi) Chrome extension or by disabling it completely by running Chrome with the following flags:

```
chrome --disable-web-security --user-data-dir
```

**IMPORTANT** please remember that this will disable the mechanism for **every** website for the duration of your whole browser session. Use with caution.

Another alternative here is using [devServer.proxy](https://webpack.js.org/configuration/dev-server/#devserver-proxy) (assuming you’re using webpack to serve your app) or use a CORS-as-a-service solution such as https://cors-anywhere.herokuapp.com/

**C — I’m developing the frontend, have no control of the backend and never will**

Ok, now things are getting complicated. First you should probably establish **why** the server is not sending the proper headers.

Maybe they do not allow 3rd party apps to access their API? Maybe their API is only meant to be consumed by server-side applications and not browsers? Maybe you should be sending an authorization token of sorts in the URL?

If you still think you should be able to access the data via browser, you will have to write your own proxy that will sit between the browser application and the API, similarly to how we did it in solution B.

![img](https://miro.medium.com/max/60/1*5ceGGBjNyh3qIlnSB0hDlw.png?q=20)

![img](https://miro.medium.com/max/1253/1*5ceGGBjNyh3qIlnSB0hDlw.png)

Adding a proxy in the middle

The proxy does not have to be running on the same domain as your application, as long as the proxy itself properly supports CORS when communicating with the client. The communication between proxy and the API does not have to support CORS at all.

You can either write your own platform, or use a ready made solution such as https://www.npmjs.com/package/cors-anywhere

Remember that such approach can introduce a security risk if you want to support credentials.

# More about CORS

If you wish to learn more about CORS details I recommend checking out the detailed [MDN article](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS).