---
title: Using type interceptors with Autofac
tags: 
  - C#
  - AutoFac
  - DynamicProxy
  - AOP
  - Interceptors
  - .NET

categories: 
  - Software development

permalink: using-type-interceptors-with-autofac
id: 59d3e06faf4f1298d16e0ec0
updated: '2017-10-03 20:59:51'
date: 2017-07-23 14:00:03
---
---
Today I'd like to share my first attempt at creating a blog post.

In this post I'll try to give a starting point for using Castle Dynamic Proxy interceptors using AutoFac, though most of it also applies to Castle Windsor.

I'll try to cover how it can be helpful using examples.

If you prefer just to get some sample code instead of reading a blog post, I've added a very basic example linked below (I purposely tried to keep it simple):

[Sample code](https://github.com/maartenderaedemaeker/Simplistic-Autofac-Interceptors-POC/tree/master/AutofacInterceptorPoc)

---
# Content
* Cross cutting concerns
* Aspect oriented programming
* Proxy pattern
* Castle Dynamic Proxy
* AutoFac: interceptors
* Conclusion
---
### Cross cutting concerns
To get an idea of what cross cutting concerns are, take a look at the following code sample.
```
public void DoSomething()
{
	using(SomeTimeMeasuringHelper.Measure(() => {
		Log("Entered DoSomething");
		try {
			using(var transaction = Helper.StartTransaction())
			{
				Retry(() => {
					/* MAYBE WE CAN PUT BUSINESS LOGIC HERE? */ 
				});
			}				
		} 
        catch(Exception e)
        {
             LogError(e); 
             throw;
        }
    }))
} // try repeat exactly the same for next method
///oh, and update everywhere if this changes
```

As you can see, we have quite a bit of code here, and none of it is actually doing any business logic.

---
### Cross cutting concerns: what are they?
According to wikipedia:
> In aspect-oriented software development, cross-cutting concerns are aspects of a program that affect other concerns. These concerns often cannot be cleanly decomposed from the rest of the system in both the design and implementation, and can result in either scattering (code duplication), tangling (significant dependencies between systems), or both.
[Source: wikipedia](https://en.wikipedia.org/wiki/Cross-cutting_concern)

So, what does that mean? Let's look back to the previous sample.

---
### Cross cutting concerns: example

![Cross cutting concerns example](/images/2017/07/23/crosscuttingconcerns.png)

---
### Aspect Oriented Programming
So, where does the *Cross cutting concern* idea come from?
Turns out, it is [Aspect Oriented Programming](https://en.wikipedia.org/wiki/Aspect-oriented_software_development).
> Aspect-oriented software development focuses on the identification, specification and representation of cross-cutting concerns and their modularization into separate functional units as well as their automated composition into a working system. (Source: wikipedia)

Right, Aspect Oriented Programming should let us be able to move all these things out of our business code, awesome! But how could we implement this in .NET?

Turns out, there are multiple ways of doing this.
* One way is to use code weaving (for example: PostSharp). The tool will make sure the aspects (for example logging) will be used by modifying the outputted intermediate language.
* Another way (the way I'm discussing in this post) is to dynamically generate proxy classes. We'll use the .NET library Castle Dynamic Proxy.
---
### Proxy pattern

This brings us back to a quick refresher of the proxy pattern, applied to the example.
The real *Something* class and the *DoSomething* method will not be invoked directly. Instead, another class will do some logic, then (optionally) delegate to the real implementation.Since they both implement the same interface, and the client is calling the object using the interface, the client does not have to know that it is talking to a proxy instead to the original class.

![Proxy pattern](/images/2017/07/23/proxy-pattern.png)

So, this is all splendid, but we don't really want to manually create a proxy class for each type we want to intercept right?

---
### Castle Dynamic Proxy to the rescue
So what is Castle Dynamic proxy?
Well, it is a library that allows us to solve the issue of generating proxy classes.
* It generates dynamic proxy classes on the fly
* It allows us to be able to intercept extend behaviour of virtual methods and methods exposed by the interface.

Why the interface or virtual limitation, you might ask?
Castle Dynamic Proxy uses the proxy pattern by using inheritance (just like the ProxySomething class in the previous image).

---
### Castle dynamic proxy: how?

We can write an AutoFac interceptor by implementing the Castle Dynamic Proxy IInterceptor interface.
The interface has one method: intercept, with one argument of the type IInvocation.
That object contains information about the invocation that is currently intercepted.
By calling the Proceed method, the wrapped invocation is called.

![Interceptor example](/images/2017/07/23/interceptor-example.png)
---
### AutoFac: using interceptors
To be able to use interceptors in AutoFac, you need to install the *Autofac.Extras.DynamicProxy* package (for more information: [take a look at the AutoFac docs](http://docs.autofac.org/en/latest/advanced/interceptors.html).
In your AutoFac container setup you can configure interception. AutoFac will wrap the returned instance with a dynamic proxy.

So, how do we register the interceptor in our container building logic?

```
builder
    .RegisterType<ConsoleLoggingInterceptor>
    .As<ConsoleLoggingInterceptor>();

builder
    .RegisterType<ValuesRepository>()
    .As<IValuesRepository>()
    .EnableInterfaceInterceptors() 
    // or EnableClassInterceptors() to enable interceptors for virtual members
    .InterceptedBy(typeof(ConsoleLoggingInterceptor));
    // TODO: maybe add more interceptors
```
---
### Conclusion

So when we look at the changes to our code, we can clearly see the difference.
Before adding the interceptors our code looked like:

![Cross cutting concerns example](/images/2017/07/23/crosscuttingconcerns.png)

When we use interceptors it would look something like this:

![Cross cutting concerns example with interceptors](/images/2017/07/23/crosscuttingconcerns-intercepted.png)

So what happens if we chain these interceptors?
Take a look at the image below:

![Cross cutting concerns interceptor flow](/images/2017/07/23/InterceptorDiagram.png)

Using interceptors does have a (minor) downside.
It becomes a little bit harder to reason about your code, because of the increased complexity. (You do not see all you code at the same spot).
I would argue that the benefits of moving the cross cutting concerns away from the business logic makes it very worthwhile though.