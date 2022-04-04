<!-- # ASP.NET Core 6.0 - Who Moved My Cheese -->

## Watch companion video on YouTube:
<iframe width="560" height="315" src="https://www.youtube.com/embed/vhNhcuht0J0" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Introduction

.NET 6.0 is out and ASP.NET Core 6.0 has shipped. There have been quite a few changes that have left a lot of people confused. For instance, "who moved my cheese", where is `Startup.cs`? In this post, I will delve into that and see where it moved as well as other changes.

Things haven't fundamentally changed in the middleware for ASP.NET Core, but some of the project structure has changed as well as where you register dependencies. To understand it better, it is instructive to start with a .NET Core 3.1 project template and upgrade it by hand to see how it compares to the new templates.

## Upgrade an old style Console Project

To begin with, let's create a new *Console Project*. I called mine `OldToNew`. I selected a .NET Core 3.1 target and will upgrade it to .NET 6.0 to see the differences. If you have been around .NET for a while, you will recognize this project structure in the `Program.cs` file.

```C#
using System;

namespace OldToNew
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello World!");
        }
    }
}
```
In .NET 6.0 the changes are intended to simplify and remove cruft from our application. One of the first features they introduced was something called [Filescoped Namespaces](https://docs.microsoft.com/en-us/dotnet/C#/whats-new/C#-10#file-scoped-namespace-declaration). Traditionally namespaces look like this:

```C#
namespace OldToNew
{
    // code goes here.
}
```

If you've been around .NET for a while you have probably never put more than one namespace in a file. You can remove the curly braces and just add a semicolon instead, marking the entire file as using one namespace.
```C#
namespace OldToNew;
// code goes here.
```

```C#
using System;

namespace OldToNew;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello World!");
    }
}
```

Visual Studio is going to complain at me because this is a .NET Core 3.1 project so before we go too far we need to edit the .NET Core 3.1 .csproj file and convert it to a .NET 6.0 app.

```XML
<!-- .NET Core 3.1 -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.1</TargetFramework>
  </PropertyGroup>

</Project>
```

```XML
<!-- .NET 6.0 -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

</Project>
```
Once we make this change, Visual Studio will be happy with the filescoped namespace. 

The next change we want to make is to remove the `using System;` line. In order to do this, we need to edit the project file again and enable [Implicit Usings](https://devblogs.microsoft.com/dotnet/welcome-to-C#-10/#implicit-usings).
```XML
<!-- .NET 6.0 -->
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```
If we enable implicit using statements most of the common using statements that we normally use will be included from the SDK by default and you no longer need to include them in your files. We can remove the `using System;` line because the compiler will automatically add the using statements for us.

```C#
namespace OldToNew;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello World!");
    }
}
```
The next feature they introduced was something called [Top Level Statements](https://docs.microsoft.com/en-us/dotnet/C#/fundamentals/program-structure/top-level-statements#:~:text=Top-level%20statements%20%28C%23%20Programming%20Guide%29%201%20Only%20one,point%20method.%20...%207%20C%23%20language%20specification.%20). The idea is to remove the "cruft" that was present in every console application or ASP.NET Core application. 

With top level statements, we can remove the `static void Main(string[] args)` method and the curly braces along with the namespace and the `class Program` declaration.

```C#
Console.WriteLine("Hello World!");
```
Once you remove all that, you can see that the only code we have left is our `Console.WriteLine()` method!

## Change Console app to a Web app (ASP.NET Core)

Currently, this is just a plain console application, but I want to convert it to an ASP.NET Core application. Before doing that let's take a look under the dependencies and frameworks node in the solution explorer.

![dependencies](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/moved-cheese/1-dependencies.jpg#screenshot)

You can see above under `Frameworks` that `Microsoft.NETCore.App` is the SDK that includes all the required packages to create a console application. Let's change the SDK type in the .csproj file to a .Web type project and see what happens: `Microsoft.NET.Sdk` to `Microsoft.NET.Sdk.Web`.

```XML
<!-- .NET 6.0 -->
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

</Project>
```
Notice what happens under dependencies now:

![dependencies for ASP.NET Core](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/moved-cheese/2-dependencies.jpg#screenshot)

The framework now includes `Microsoft.AspNetCore.App` which will bring in all the necessary packages to create an ASP.NET Core application. It will also modify the global using statements to include ASP.NET specific using statements.

Now let's remove the `Console.WriteLine("Hello World!");` line and add the ASP.NET Core specific code.

```C#
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();
app.Run();
```

We are now constructing a web application. Where does the `WebApplication` object come from? It belongs to the `Microsoft.AspNetCore.Builder` namespace.

We could run this app, but it'll be kind of boring because there are no endpoints. Let's add an endpoint to make this complete:

```C#
app.MapGet("/", () => {
    return "Hello World!";
});    
```
We just made a minimal API web application in ASP.NET Core 6.0. Let's run it and see what happens.

![hello world output from running app](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/moved-cheese/3-hello-world.jpg#screenshot) 
When we launch the default path it returns `hello world` so we have a fully functioning web application. Minimal APIs are a great way to quickly build a Web API project.

## What about Startup.cs?
The middleware flow in ASP.NET Core 6.0 is similar to that of the full Web API MVC projects and shares much of the same implementations. In previous ASP.NET Core projects you were given a `Startup.cs` class with two methods in it `ConfigureServices()` and `Configure()`.

In `ConfigureServices()` you register services for dependency injection. In `Configure()`, you outline your middleware pipeline order and structure.

What does this look like in our new project? Where do you put dependency injection? The answer is between this first and second line and register middleware between the second and third as noted below. 
```C#
var builder = WebApplication.CreateBuilder(args);
// REGISTER SERVICES HERE
var app = builder.Build();
// REGISTER MIDDLEWARE HERE
app.Run();
```

For example, if I wanted to add Authentication I would register it like this:

```C#
var builder = WebApplication.CreateBuilder(args);
// REGISTER SERVICES HERE
builder.Services.AddAuthentication(...) ...
builder.Services.AddAuthorization();
var app = builder.Build();
// REGISTER MIDDLEWARE HERE
app.UseAuthentication();
app.UseAuthorization();
app.MapGet("/", () => {
    return "Hello World!";
});  
app.Run();
```

Order matters in the middleware pipeline, so I have added `UseAuthentication()` and `UseAuthorization()` above the `MapGet()` middleware. In order for this to take effect, however, you need to annotate the endpoint that you want protected with the `[Authorize]` attribute. This will require the using statement of `Microsoft.AspNetCore.Authorization`. 

```C#
using Microsoft.AspNetCore.Authorization;

...

app.MapGet("/",[Authorize]() => {
    return "Hello World!";
});
```

The complete code of our `Program.cs` file to this point looks like this:

```C#
var builder = WebApplication.CreateBuilder(args);
// REGISTER SERVICES HERE
builder.Services.AddAuthentication();
builder.Services.AddAuthorization();
var app = builder.Build();
// REGISTER MIDDLEWARE HERE
app.UseAuthentication();
app.UseAuthorization();
app.MapGet("/",[Authorize] () => {
    return "Hello World!";
});  
app.Run();
```

If we were to run this right now, we would get an exception because we never specified the Authentication Scheme.

Let's fix it. There are a lot of different types of Authentication Schemes we could choose but in a WebAPI it is typical that we protect the application with something like a bearer token or a JWT token.

Let's go ahead and add JWT bearer to the minimal API. We will need to import the `Microsoft.AspNetCore.Authentication.JwtBearer` package via NuGet, then add the following code:

```C#
using Microsoft.AspNetCore.Authentication.JwtBearer;

...

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer();
```

So the complete block should look like this:
```C#
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;

var builder = WebApplication.CreateBuilder(args);
// REGISTER SERVICES HERE
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer();
builder.Services.AddAuthorization();
var app = builder.Build();
// REGISTER MIDDLEWARE HERE
app.UseAuthentication();
app.UseAuthorization();
app.MapGet("/", [Authorize] () => {
    return "Hello World!";
});
app.Run();
```

JWT Bearer has been added to the application so let's run it and see what happens. We now get exactly what we're looking for, a `401 unauthorized` status because we haven't supplied it with any kind of token. There would be additional steps to setup a way to aquire a token and make sure that we only allow valid tokens to be accepted, but that is beyond the scope of this article. The main point of this article is to demonstrate where to register dependencies and middleware and how to use them.

One last modification we should make is to change the [Authorize] attribute to use a 'fluent syntax' instead like this:

```C#
//FROM THIS:
app.MapGet("/", [Authorize] () => {
    return "Hello World!";
});

//TO THIS:
app.MapGet("/", () => {
  return "Hello World!";
}).RequireAuthorization();
```

They both do the same thing, however the 'fluent syntax' feels more natural in the minimal API case.

## The return of the missing Startup.cs file
Next, let's see if we can bring back our old friend the `Startup.cs` file. Minimal API structure has the potential to get messy and bloated if you put everything in a single file.

There must be a way that we can organize this better. I'll begin by adding a new class called `Startup.cs` to the project.

```C#
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {

    }

     public void Configure(WebApplication app, IWebHostEnvironment env)
    {

    }
}
```
This should look familiar from the full Web API projects templates of the past. I just pasted it into my new `Startup.cs` file and removed the namespace. The `Startup.cs` file requires an `IConfiguration` object to be passed in. Can we supply that from our `Program.cs`?

```C#
//  Program.cs file
var builder = WebApplication.CreateBuilder(args);
var startup = new Startup(builder.Configuration);
```

Yes. We can do that. The `builder` object that is created contains a `Configuration` property that we can use to pass in to the `Startup` constructor.

Next, let's try to supply the `ConfigureServices()` method with an `IServiceCollection`.

```C#
//  Program.cs file
var builder = WebApplication.CreateBuilder(args);
var startup = new Startup(builder.Configuration);
startup.ConfigureServices(builder.Services);
```

Again, `builder` contains a `Services` property that we can use to pass in to the `ConfigureServices()` method. Now we can remove all the dependency injection code from `Program.cs` and move it to the `ConfigureServices` method in `Startup.cs`.

```C#
// Startup.cs file
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer();
    services.AddAuthorization();
}
```

Next, let's try to supply the `Configure()` method with the `WebApplication` class.

```C#
//  Program.cs file
var builder = WebApplication.CreateBuilder(args);
var startup = new Startup(builder.Configuration);
startup.ConfigureServices(builder.Services);

var app = builder.Build();

startup.Configure(app, builder.HostingEnvironment);
```

Interestingly, although we are passing `app` which is of type `WebApplication` if you inspect it, the `Configure()` method is expecting an `IApplicationBuilder`, but it seems ok with it. Why? If you drill into the `WebApplication` object, you will see that it implements the `IApplicationBuilder` interface.

Let's cut our middleware pipeline code out of `Program.cs` and paste it into the `Configure()` method in `Startup.cs`.

```C#
// Startup.cs file
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
  app.UseAuthentication();
  app.UseAuthorization();

  app.MapGet("/", () =>
  {
      return "hello world";
  }).RequireAuthorization();

  app.Run();
}
```

Unfortunately, this won't actually work because the interface `IApplicationBuilder` doesn't have a `MapGet()` method or a `Run()` method. Those live in the `WebApplication` class. If we change the `Configure()` method to accept an `WebApplication` object, it should work.

```C#
// Startup.cs file
public void Configure(WebApplication app, IWebHostEnvironment env)
{
  app.UseAuthentication();
  app.UseAuthorization();

  app.MapGet("/", () =>
  {
      return "hello world";
  }).RequireAuthorization();

  app.Run();
}
```

Let's look at both complete files, 'Program.cs' and 'Startup.cs', to see how they look now.

```C#
// Program.cs file
var builder = WebApplication.CreateBuilder(args);
var startup = new Startup(builder.Configuration);
startup.ConfigureServices(builder.Services);
var app = builder.Build();
startup.Configure(app, builder.Environment);
```

```C#
// Startup.cs file
using Microsoft.AspNetCore.Authentication.JwtBearer;

public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    // This method gets called by the runtime. Use this method to add services to the container.
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddAuthentication(options =>
        {
            options.DefaultScheme = JwtBearerDefaults.AuthenticationScheme;
        }).AddJwtBearer();
        services.AddAuthorization();
    }

    // This method gets called by the runtime. Use this method to configure the HTTP request pipeline.
    public void Configure(WebApplication app, IWebHostEnvironment env)
    {
        app.UseAuthentication();
        app.UseAuthorization();

        app.MapGet("/", () =>
        {
            return "hello world";
        }).RequireAuthorization();

        app.Run();
    }
}
```
If we run it, we get the `401 Unauthorized` status which means it is working. We haven't changed too much, other than to move our configuration stuff into the `Startup.cs`, which makes it look more like the old .Net Core style.

## Can we do better?
Using `Startup.cs` does help improve organization, however I think we can do better. I've always hated in the `Startup.cs` file that the words `ConfigureServices()` and `Configure()` are so close to each other plus you're passing in an `IConfiguration` object. It can be confusing about what goes where.

Why don't we try to improve this. I don't like the name `ConfigureServices()`. What if we renamed it to `RegisterDependentServices()` instead and placed it in its own file? This would make it easier to understand what is going on.

```C#
// RegisterDependentServices.cs file
using Microsoft.AspNetCore.Authentication.JwtBearer;

public static class RegisterDependentServices
{
    public static WebApplicationBuilder RegisterServices(this WebApplicationBuilder builder)
    {
        // Register your dependencies
        builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer();
        builder.Services.AddAuthorization();
        return builder;
    }
}
```

I decided to make the class `static` and use an extension method to add it to the `WebApplicationBuilder` class. We will see how this improves the `Program.cs` file in a moment.

Next, let's create a new file `SetupMiddlewarePipeline.cs`. This file will contain the middleware pipeline.

```C#
public static class SetupMiddlewarePipeline
{
    public static WebApplication SetupMiddleware(this WebApplication app)
    {
        // Configure the pipeline !! ORDER MATTERS !!
        app.UseAuthorization();
        app.UseAuthentication();
        app.MapGet("/", () =>
        {
            return "hello world";
        }).RequireAuthorization();
        return app;
    }
}
```

Now, how does this change our `Program.cs` file?

```C#
// Program.cs file
WebApplication app = WebApplication.CreateBuilder(args)
    .RegisterServices()
    .Build();

app.SetupMiddleware()
    .Run();
```

This makes for a very clean `Program.cs` file. In fact, you could make it all one line if you wanted.

```C#
// Program.cs file
WebApplication.CreateBuilder(args)
    .RegisterServices()
    .Build()
    .SetupMiddleware()
    .Run();
```

I don't hate it. In fact, I think I like it.

## What about IConfiguration?
Before, the `Startup()` constructor expected `IConfiguration` to be injected into it. However, because `WebApplication` and `WebApplicationBuilder` both have a `.Configuration` property, we no longer need to explicitly inject it.

```C#
// RegisterDependentServices.cs file
public static class RegisterDependentServices
{
  public static WebApplicationBuilder RegisterServices(this WebApplicationBuilder builder)
  {
    // ******* Access the configuration *******
    var config = builder.Configuration;

    // Register your dependencies
    builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
        .AddJwtBearer();
    builder.Services.AddAuthorization();
    return builder;
  }
}
```

```C#
// setupMiddlewarePipeline.cs file
public static class SetupMiddlewarePipeline
{
  public static WebApplication SetupMiddleware(this WebApplication app)
  {
    // ******** Access the configuration ********
    var config = app.Configuration;

    // Configure the pipeline !! ORDER MATTERS !!
    app.UseAuthorization();
    app.UseAuthentication();
    app.MapGet("/", () =>
    {
      return "hello world";
    }).RequireAuthorization();
    return app;
  }
}
```

If we need to access the app's configuration at all, it is available and we are in good shape.

## Summary
We have examined some different ways to organize Minimal API projects or any new ASP.NET 6.0 project. I like that we are given opportunities to structure our projects in ways that may be easier to read and maintain.

I think we found our cheese and and maybe even better options. if you found this post helpful, let me know in the comments below and share it with someone else too!
