---
title: Improving your application's performance using HTTP caching response headers
tags:
  - HTTP
  - Caching
  - Performance
permalink: http-caching
id: 59e9fd0baa906d22b026178f
updated: '2017-10-23 11:01:46'
date: 2017-10-20 15:41:31
author: Maarten De Raedemaeker
---
Today's blogpost will be about how you can improve your web application's performance by adding caching using HTTP headers.
These headers allow us to limit the number of requests clients have to make, reducing load and improving client speed.
# Cache-control
The main header we'll be discussing is the cache-control header. This header is <a href="https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control#Browser_compatibility">well supported</a>. It allows us to instruct how proxies and the end user's browser should handle a resource, cache-wise.
Since there are a few bugs around with "immutable", "stale-while-revalidate" and "stale-if-error", I'll focus on the more basic options.
* The first thing to discuss is **cacheability**
    * *Public*: response can be cached by any cache
    * *Private*: response should not be cached by a shared cache, only by end user.
    * *No-Cache*: can be cached, but each request has to be <a href="#revalidation">revalidated</a>. This means that a shared cache can still hold on to a resource, but checks with the server before returning the cached content.
    * *No-Store*: disables caching.
* The second part is about **expiration**.
Important options here are:
    * *max-age=[numberofseconds]*: this option defines how long both local and shared cached can cache a response.
    * *s-max-age=[numberofseconds]*: this option does the same as max-age, but excludes private (local) caches, so it only works for shared caches.

Combined these could form the following headers:

**Example 1**: local / private caching for one hour, not on shared proxy.

```
Cache-Control: max-age=3600 private 
```

**Example 2**: can be cached both locally and on shared proxies for 24 hours.
```
Cache-Control: max-age=86400 public
```

# Vary
Allows deciding if a response needs to be cached based on if a set of request headers differ. 
You could use it to make sure content-negotiated content is cached correctly. For example, an XML response is already cached, you want to make sure to get another response when you ask for JSON by letting the caching vary on the "Accept-Encoding" headers. 
```
Vary: Accept-Encoding
```
# Validation
There are two main ways of validating if a resource has changed.
## Strong validation: ETags 
If the browser sends back an ETag header with a specific value, the client can later pass on the ETag in a following request to check if the content has changed. 

For example:

**Client**
```
GET /someResource
```
**Server**
```
200 OK
ETag: "abcdef"

// SOME CONTENT
```

**Client**
```
GET /someResource
If-None-Match: "abcdef"
```

**Server**
```
304 Not Modified
```

## Weak validation: Last-Modified

This is a similar principle, though it depends on a date instead of a unique value.

**Server**
```
Last-Modified Sun, 05 Feb 2017 10:34:56 UTC
```

**Client**
```
If-Not-Modified-Since: Sun, 05 Feb 2017 10:34:56 UTC
```

**Server**
```
304 Not Modified
```
# Revalidation <a name="revalidation"></a>
* *immutable*: this tells the agent requesting the content that this resource will never change. The browser would be allowed to keep the resource, even after a hard refresh.
* *must-revalidate*: this forces the client to check if the content has changed each time using the If-Modified-Since or If-None-Match mechanisms.
* *proxy-revalidate*: this forces the proxy to check if the content has changed each time using the If-Modified-Since or If-None-Match mechanisms.
# Conclusion
These headers give you some tools to get some nice performance gains for your web application using HTTP caching. Combined with a reverse proxy service such as <a href="https://www.cloudflare.com/">Cloudflare</a>, these can take a great load of your servers.

## Sources & further reading
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching
* https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching
* https://blog.fortrabbit.com/mastering-http-caching
* https://devcenter.heroku.com/articles/increasing-application-performance-with-http-cache-headers
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Cache-Control
* https://www.mnot.net/cache_docs/
* https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Vary
