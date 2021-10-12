## Safely store app secrets in ASP.NET Core
Applications often need to store secrets, such as client_id and client_secret for OAuth connections or database connection strings. In the old days, it was not uncommon to store these secrets in the code and compile them as hard coded values. Today, this practice is less desirable and external configuration settings that can be changed without requiring a recompile of an application is much preferred. However, this new capability comes with a new set of responsibilities. 

In this article, we will learn about a few different ways to secure our secrets and make them configurable by environment for our applications. It is a best practice to avoid checking in our secrets to source control where they may be inadvertantly (or intentionally) leaked.

### Create a sample web application
We will use Visual Studio to create a sample web application to demonstrate how to store application secrets. 

1) Open Visual Studio
![visual studio](images/user-secrets/1-visualstudio.jpg)
2) Click "Create a new project"
![create new project](images/user-secrets/2-createnewproject.jpg)
3) Select C# ASP.NET Core Web App (Model-View-Controller) project type
4) Select a location to create the project and give it a name. I named mine "MyWebApp".
![my web app](images/user-secrets/3-mywebapp.jpg)
5) Leave the defaults in the additional info dialog as shown below with.>NET5.0 (current), Authentication type:none, checkbox selected for Configure for HTTPS and everything else unchecked.
![additional info dialog](images/user-secrets/4-additionalinfo.jpg)

### Explore Startup.cs
Once the solution loads, find the `Startup.cs` file and open it. Inside you will find a number of items already configured. An important one for our use case is the `IConfiguration Configuration` property.

```c#
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
            services.AddControllersWithViews();
        }
    ...
```

In the Startup class constructor, the `IConfiguration configuration` parameter is provided to us by the dependency injection system using constructor injection and assigned to the `Configuration` property for later use.

For our sample app, let's create a variable to store a passkey that we could use to, for example, encrypt a string. 

```c#
        // This method gets called by the runtime. Use this method to add services to the container.
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddControllersWithViews();
            string Passkey = "Pard0nMyDu5t";
            Console.WriteLine($"passkey: {Passkey}");
        }
```

I have added two lines of code inside the Configure Services method just below the `AddControllersWithViews()` line.

In this example, I have hard coded the actual key into the variable `Passkey`. If you run the application, 

![run from debug](images/user-secrets/5-run-debug.jpg)

you will see this passkey value displayed on the console.

![console passkey](images/user-secrets/6-console-passkey.jpg)

There are several problems with this example. First, hard coding the value makes it difficult to change or rotate the key without re-compiling the application and re-deploying. Second, this value **will** get checked into source control where anyone with access to the source can see it. Not much of a secret.

In the next section, we will solve the first problem of moving the value from hard coded into an external configuration file.

### App settings in external config file
In ASP.NET Core projects, a developer can choose to store configuration values in an external file such as the appsettings.json file.

![solution explorer](images/user-secrets/7-solution-explorer.jpg)

In our project, using the solution explorer view, locate the existing appsettings.json file and open it.

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*"
}
```

You will see a json structure in this file. It currently contains 2 configuration properties `Logging` and `AllowedHosts`.

First, lets add a simple key value pair in the appsettings.json file `"Passkey": "Pard0nMyDu5t"` so the json file looks like this:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "Passkey": "Pard0nMyDu5t"
}
```

Next we will remove the hard coded value from the `Startup.cs` file and replace it with code to get the config value from the  `Configuration` object. This code will pull the value in from the appsettings.json file: `string Passkey = Configuration["Passkey"];`.

```c#
// This method gets called by the runtime. Use this method to add services to the container.
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews();
    string Passkey = Configuration["Passkey"];
    Console.WriteLine($"passkey: {Passkey}");
}
```

Now if we run the project again, we will see the same result as before, but the value is no longer hard coded in our application code and lives in an external file.

### User Secrets file
So, it is good that we have moved the value into an external json configuration file, but this file lives within our solution folder and will get checked in with our code. It is a good idea to avoid checking in secrets into your repository.

To solve this problem, ASP.NET Core provides a way to store the secrets in a file that lives on your development machine but outside of your solution folder so it isn't managed by source control.

In Visual Studio, the easiest way to set this up is to right click on the project and find the menu item: **Manage User Secrets**.

![manage user secrets menu](images/user-secrets/8-manage-user-secrets.jpg)

This will create a new empty configuration file called secrets.json that is external to the solution but attached to the project for development purposes. 

![usersecrets.json file](images/user-secrets/9-secrets-json.jpg)

Let's move our key value pair secret from appsettings.json into our secrets.json file instead. Remove `"Passkey": "Pard0nMyDu5t"` from appsettings.json and into secrets.json.

![move passkey to secrets.json file](images/user-secrets/10-passkey-to-secrets.jpg)

Then run the program again and see that it still works.

So the question you may be asking yourself is where is this file actually located? 

To find out, hover over the secrets.json tab and you will see the full path to the file.

![hover secrets.json tab](images/user-secrets/11-hover-tab.jpg)

From theimage you will see that it is located in `C:\Users\{user}\AppData\Roaming\Microsoft\UserSecrets\7ad9d3cc-65f2-49cf-b672-fcb3dd7930b5\secrets.json`

So, Microsoft has a folder called UserSecrets where all the secrets are stored on your machine. For each project you add, it generates a random GUID for a subfolder name in which it places a `secrets.json` file.

So the next question you might ask is "how does our project know to look in that particular subfolder with the GUID"?

To discover how this gets connected, right click on your project and select the **Edit Project File** menu option.

![Edit Project File](images/user-secrets/12-edit-project-file.jpg)

This will open a new tab with a name `MyWebApp.csproj` with the following information inside:

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net5.0</TargetFramework>
    <UserSecretsId>7ad9d3cc-65f2-49cf-b672-fcb3dd7930b5</UserSecretsId>
  </PropertyGroup>

</Project>
```

Notice the XML node called `<UserSecretsId>`? This is how the project knows where to find the `secrets.json` file located outside of the project in the predetermined location.

### Deployment Configuration
So we have now accomplished moving hard coded secrets out of our code and into an external file, and then into an external file that lives outside of our project for development so those secrets will never get checked into source control. But what happens when we try to deploy our application? Where should our secrets live?

There are mutiple options for setting configuration values on the server and it depends upon what type of server you are deploying to.

#### Environment Variables
First lets discuss what a Windows Server VM might look like and where the configuration values should be placed. One of the easiest places to put configuration values is in **Environment Variables** on the server.

From an administrator command prompt you can use the `setx` command to see current Environment Variables. To assign a configuration value such as our passkey, you would run the `setx` command like this:

```cmd
setx Passkey Pard0nMyDu5t /M
```

If you need to remove a value, unfortunately the `setx` command doesn't support a delete option. Instead, use the `REG delete` command like this:

```cmd
REG delete "HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Environment" /F /V Passkey
```
After setting the environment variable, you will need to restart the application. If you are testing this on a dev machine, you will need to restart Visual Studio to access the updated environment variable.

To set Environment Variables from a UI, search for Environment Variables in Windows search:

![search env vars](images/user-secrets/13-edit-environment.jpg)

Click on the Control Panel item in the search results to launch.

![system properties dialog](images/user-secrets/14-system-properties.jpg)

Next, click the **Environment Variables** button.

![environment variables dialog](images/user-secrets/15-new-env.jpg)

 On the Environment Variables dialog screen, click the **New** button to add a new variable.

 ![new system variable dialog](images/user-secrets/16-new-sys-var.jpg)

Enter a variable name and value in the dialog and click ok.

![newly added system variable](images/user-secrets/17-added-passkey.jpg)

You will find a new value in the System variables list.

Again, to make it visible in Visual Studio, you will need to restart the IDE.





