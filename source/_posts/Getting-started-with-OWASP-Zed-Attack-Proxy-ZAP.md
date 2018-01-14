---
title: Getting started with OWASP Zed Attack Proxy (ZAP)
tags:
  - Security Testing
  - .NET
  - C#
  - Zed attack proxy
categories:
  - Software development
permalink: getting-started-with-zed-attack-proxy
id: 59d3e06faf4f1298d16e0ec6
updated: '2017-08-28 07:25:13'
date: 2017-08-10 18:01:46
author: Maarten De Raedemaeker
---
# Table of contents
* [Introduction](#intro)
* [OWASP Top 10](#owasp-top-ten)
* [OWASP Zed Attack Proxy](#zap)
* [Setting up ZAP](#setup)
* [Setting up the attack context](#context)
* [Executing your first attack](#attack)
* [Conclusion](#conclusion)

# <a name="intro"></a>Introduction
In this blogpost I'll attempt to explain how to get started with Zed Attack Proxy (ZAP) and how to set up your first attack. It speaks for itself that  **you ONLY attack applications you own or have explicit permission to attack**.
The source code used for this blog post can be found [on github](https://github.com/maartenderaedemaeker/Automated-SecurityTesting-Demo/tree/vulnerabilities-example).
# <a name="owasp-top-ten"></a>OWASP Top 10
OWASP (Open web application security project) is an organization trying improve awareness about web application security issues.
Every few years they create a top 10 of web application security risks. They categorize the risks based on how common the issues are and the impact when the risk is exploited. You can find out more about them on [their website](https://www.owasp.org/index.php/Main_Page).
# <a name="zap"></a>The OWASP Zed attack proxy project
OWASP Zed Attack Proxy is an open source security tool maintained by OWASP. It can be used to find security issues in your web application. It acts as a [reverse proxy](https://en.wikipedia.org/wiki/Reverse_proxy) server so it can sit in the middle and observe / modify your browser traffic. You can do an automated scan where the spider tool crawls through your application while executing attacks or manually execute specific targetted attacks. You can use the Fuzzer tool to modify a specific part of your request with random data or words from word lists to try find vulnerabilies. Zap can be extended with plugins. Which can be installed from the 'marketplace' in the application. Beware though, since some of them are used to try testing exploiting vulnerablities using malicious files, your antivirus will probably go crazy. 

To add some clearity to the input fuzzing concept:

<blockquote class="twitter-tweet" data-lang="nl"><p lang="nl" dir="ltr">QA Engineer walks into a bar. Orders a beer. Orders 0 beers. Orders 999999999 beers. Orders a lizard. Orders -1 beers. Orders a sfdeljknesv.</p>&mdash; Bill Sempf (@sempf) <a href="https://twitter.com/sempf/status/514473420277694465">23 september 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

# <a name="setup"></a>Setting up ZAP
To be able to get started with ZAP, we need to install the application from [their site](http://www.zaproxy.org/). Since ZAP acts as a reverse proxy, we need to setup our browser proxy settings to point to ZAP, so our requests get tunneled.
By default, the proxy port is set to '8080'. You can change this by clicking on 'Tools' > 'Options' > 'Local proxy', as shown in the image below.
![ZAP-Proxy](/images/2017/08/10/ZAP-Proxy.png)

If you want to intercept HTTPS traffic, you'll also need to generate & trust a new SSL certificate. This can also be done from the 'Tools' > 'Options' menu, but this time selecting the 'Dynamic SSL Certificates'.

![ZAP-Cert](/images/2017/08/10/ZAP-Cert.png)

From here, you can save the certificate to a local file and install the certificate as trusted locally.

In your browser, make sure to set the proxy settings accordingly. You could install a tool like [Foxy proxy (for chrome)](https://chrome.google.com/webstore/detail/foxyproxy-basic/dookpfaalaaappcdneeahomimbllocnb) or other tools to be able to easily enable / disable the proxy settings.

Once everything is setup correctly and you visit a site in your browser it will be added to the "Sites" tab in ZAP. You'll notice any site you visit turns up here, so before we execute any real attacks, we'll want to make sure our [attack context](#context) is set up properly, so we don't attack anything we do not have permission for.

![Sites](/images/2017/08/10/Sites.png)

# <a name="context"></a>Settings up the attack context
Before we start executing attacks, it is important to set up our context so that we don't accidentually attack anything we shouldn't. 
While just browsing, you can right click on the site, select 'include in context' > 'new context'.

![configuring_context](/images/2017/08/10/configuring_context.png)

After we configured the sites we want to test, we'll also delete / disable the default context by right clicking it and selecting delete or disable.

You can start a quick scan by entering your site's url in the quick start menu.

![Quick-scan](/images/2017/08/10/Quick-scan.png)
# <a name="attack"></a>Executing your first attack
For the demo purposes of my application I've created two vulnerabilities in my application. A SQL injection issue and an XSS flaw. You can take a look at the source [here](https://github.com/maartenderaedemaeker/Automated-SecurityTesting-Demo/blob/vulnerabilities-example/SecurityTestingDemo/Controllers/VulnerabilityController.cs).

* The SQL injection.

The SQL injection is quite straight forward. I have a table where I display all the 'employees' the current user is 'supervisor' for. You can also filter on a name. The url would be something like this '/Vulnerability/SqlInjection?name=Test'. When the user is not logged in, this shows something like this:

![Sql_example](/images/2017/08/10/Sql_example.png).

In the code we can see the query is built using string concatenation.
```
var query = $"SELECT Id, Name, PhoneNumber, DateOfBirth FROM Employees WHERE SupervisorId='{userId}'";

if (name != null)
{
    query += $" AND name = '{name}'";
}
```

Let's see if ZAP can find any SQL injection issues here.

We'll right click the url, click attack and start scan.

![url_list](/images/2017/08/10/url_list.png)

![active_scan](/images/2017/08/10/active_scan.png)

And, as we expect, ZAP has found an issue:

![sql-inject](/images/2017/08/10/sql-inject.png)

We can even get all data to show up with the right request. When getting ```/Vulnerability/SqlInjection?name=' OR '' = '```
The application just shows all data.

![injected](/images/2017/08/10/injected.png)

* XSS

I have to admit I had to do a lot wrong to be able to create an XSS vulnerability in my ASP.NET MVC application.

First of all I needed to use ```@Html.Raw(Model)``` in my view, instead of a safe variant, but then still it would not work, I had to set ```[ValidateInput(false)]]``` on my controller. Even then my browser was fighting back (even without adding any CSP headers!).

![xss_chrome](/images/2017/08/10/xss_chrome.png)

(I know the error message in chrome is in dutch, it basically says chrome has detected unexpected code on the page and has blocked it to protect user data).

Turns out I also had to add the ``` X-XSS-Protection ``` header with value '0'. To be able to avoid chrome's interference. 

Seems like our framework does a lot for us here already :).
After these modifications ZAP found the Cross site scripting issue.

![xss](/images/2017/08/10/xss.png)

# <a name="conclusion"></a> Conclusion
Zed attack proxy helps us developers to be able do quite a bit of testing on our applications to find security related issues. It is rather easy to get started with it, and can help us getting started in improving our web application security.