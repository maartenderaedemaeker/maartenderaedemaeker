---
title: Finding vulnerable Javascript dependencies in your project with RetireJS
tags:
  - Security Testing
  - Javascript
categories:
- Software development
permalink: using-retirejs
id: 59d3e06faf4f1298d16e0ec5
updated: '2017-09-26 14:59:42'
date: '2017-07-28 13:51:35'
author: Maarten De Raedemaeker
---

[RetireJS](https://github.com/RetireJS/retire.js) is a tool that allows you to check your javascript dependencies for known vulnerabilities. 
It works both for client-side dependencies and NodeJs dependencies. 
You can find the demo project used in this blog post [here](https://github.com/maartenderaedemaeker/RetireJsDemo).
# Table of contents
* [OWASP Top 10: using vulnerable dependencies](#owasp)
* [Using retirejs from the command line](#commandline)
* [Possible next steps](#next-steps)

# <a name="owasp"></a>OWASP Top 10: using vulnerable dependencies
In this post I'll try to get a way of helping to prevent one of the risks described in the OWASP top 10: using components with known vulnerabilities. If you're unfamiliar with the concept, you can take a look at [my post about solving this issue for a .NET project](https://maartenderaedemaeker.be/2017/08/13/using-owasp-dependency-check/#owaspTopTen).
# <a name="commandline"></a> Command line version
The command-line version can be installed using npm. 

```
npm install -g retire
```

You can call retire by executing 

```
retire
```


in the command-line.

You might prefer to keep it as a local dependency instead of a global one. You can do it like this:
```
npm install retire --save-dev
```

You can execute it like this: 
```
node node_modules/retire/bin/retire
```

You'll notice that it seems bit inconvenient, so you can add a script to your package.json to make it easier to access.

````
"scripts": {
	  "retire": "node node_modules/retire/bin/retire"
},
```
Now you can run it by executing ```npm run retire```.

When we run it, we can see a list of vulnerabilities found in our project.
We can also redirect it to a file using the ```--outputpath``` argument.

You will probably get an output resembling [this](https://gist.github.com/maartenderaedemaeker/2ff80d960c971514ce6fd589c3e31bd8).

# <a name="next-steps"></a> Possible next steps
You could integrate running this process in your local development process, or integrate it in your build server. You could for example make your build fail based on the exit code retirejs returns. You could also include it in a Zed Attack Proxy scan (see [my previous post about ZAP](https://maartenderaedemaeker.be/2017/08/27/getting-started-with-zed-attack-proxy)), there is a plugin enabling scanning of client-side dependencies for vulnerabilities, you can find out more about it [here](https://github.com/h3xstream/burp-retire-js). At the moment of writing, the plugin is still in alpha stage though.



