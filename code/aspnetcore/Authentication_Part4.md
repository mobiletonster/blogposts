## Securing web APIs and SPA (single page applications)

### Introduction
Before we can properly secure our applications, we need to understand the different types of web development options and hosting or server strategies.

### Types of web sites/apps
This can get pretty involved, but generally there are 4 major categories of web sites/apps:

1. Traditional web
2. Applets - flash, silverlight, java applets
3. SPA - single page applications
4. Web Assembly - Blazor

Microsoft has a basic guide to explain the tradeoffs between these application styles in more detail:
[Choose between web app styles docs.microsoft.com](https://docs.microsoft.com/en-us/dotnet/architecture/modern-web-apps-azure/choose-between-traditional-web-and-single-page-apps)

Even within these categories, there are several sub-options and methods for creating web apps.

#### Traditional web

The definition of a "traditional" web app or site may be a bit fuzzy, but in general the server plays a larger role as opposed to other app models where much of the work is pushed to the client browser itself. The simplest versions are static sites with primarly HTML and CSS with some or no Javascript. Static sites are written by the developer or otherwise pre-generated and the server simply returns them to the client browser on request. These pages aren't dynamically generated.

There are several technologies in the .Net space to generate HTML dynamically, namely Web Forms (which is a bit dated, but still used), MVC (Model View Controler) and the newest additions of Razor Pages (not to be confused with MVC Razor Views) and server-side Blazor. MVC Razor Views, Razor Pages and Blazor use the Razor syntax and C# to generate dynamic HTML on the server.


#### Applets - Flash, Silverlight, Java applets, ActiveX

I debated heavily on whether to even include this category as they are mostly defunct today. It wasn't long ago that there was a host of Flash, Silverlight, Java applets and ActiveX based content on the web. 

Due to security concerns and a desire to move away from supporting proprietary plug-ins in the various browsers (particularly mobile browsers), the industry slowly moved away from these technologies. Flash is the last surviving technology that is still in use but should be phased out by 2021.  These applets were written in non-javascript languages such as ActionScript, C#, Java and even Visual Basic. 


#### SPA - single page applications

These are traditionally Javascript, client side monolithic apps. A single page app is rarely just a single UI page but gets its name from the number of pages the server actually sends to the client; often only one.

In a SPA, the primary page (index.html for example) contains all the references to an entire Javascript application which will be responsible for dynamically building the UI on the client side. 

These style apps really gained popularity during the late 2000's (2008-2010) with the advent of frameworks such as KnockoutJS, AngularJS and others.

SPA applications can appear very performant for the user and is one of the reasons for its popularity. Rather than reload the entire page from the server, the client side SPA can simply request data from the server in a small packet, then act on that data and re-build the page dynamically on the client. This is an efficient method for building dynamic, interactive web applications.

Today, there are several popular frameworks including Angular, ReactJS, VUE and Svelte.

#### Web Assembly - Blazor

Perhaps a bold statement, but the future of web apps probably lies with Web Assembly. Web Assembly, sometimes referred to as WASM, is designed to enable high-performance applications on web pages and can be written in many other languages besides Javascript.

Microsoft's Blazor is a Web Assembly framework using C# and Razor syntax to build components that can dynamically build HTML in the client browser. It has made a great deal of progress in the past year plus and has become a very usuable framework.

The biggest limitation of Web Assembly is that it is such a new feature that many older browsers, particularly mobile browser, may not be able to run them. Additionally, some corporate networks block Web Assembly downloads which can prevent users from getting the payloads needed to run the application.

