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

### Hosting models for SPA 
Securing traditional applications is usually pretty straightforward and I have covered that in parts 1-3 of this series, but SPA applications have a variety of hosting options that can require special attention.

#### Separate UI server & backend server
In this model, the UI layer or SPA application can be served up from its own server, such as a Nodejs server or even as static assets from a CDN or other location. The backend services are served from a separate server and depending upon architecture could be served using Nodejs, ASP.NET Core API, Java Spring Boot, PHP, or other technology.

This model is popular for large teams of dedicated developers working separately on the FE (front end) and the BFF (back end for front end). This way teams can iterate somewhat independantly and use different stacks to do their work.

##### CORS

This model can introduce some complexities that need to be understood. If the UI will make direct calls into the API servers and they are on slightly different domains (for example fe.domain.com  and bff.domain.com) then you will need to deal with CORS (cross origin resource sharing). Cookies may or may not be sharable with the BFF depending on the the domain, so most often Bearer tokens in the Authorization Header are used to authenticate the request.

##### Proxy

You could also proxy all requests through the Nodejs server responsible for serving the front end. Again, passing cookies gets complicated in this model as the headers would need to be duplicated when constructing the outbound call to the actual API. Typically, it is more common to use a bearer token instead.

#### Combined UI & Backend Server
In this particular model, development can  occur separately or together. For instance, a web developer can build the entire FE while using Nodejs to serve the pages during development. Then a production build can be made and merged with (placed into the wwwroot folder, for example) the ASP.NET Core backend prior to deployment or after deployment.

Additionally, you can create a project that combines the UI & the ASP.NET Core project and develop together and deploy together. There is special middleware to support SPA applications during development and even support SSR (server side rendering) using nodejs as a middleware service.

This model avoids issues with CORS and doesn't require a proxy. Additionally, traditional Auth approaches such as cookies can easily be used. Usually those cookies would be unreadable by the FE code, however (http only). To help the front end know about authorization and context there are approaches that can be used to provide and cache that information in the FE layer.

#### Blazor Client Web Assembly
Blazor Client is technically a SPA model, but since it uses C#/Razor as its view layer differs slightly from traditional JS SPA frameworks.

When talking to the backend, if served seperately from the backend, it may still need to deal with CORS and cookie limitations.







