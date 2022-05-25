## Building a Custom Configuration Provider for ASP.NET Core
In a previous post, we learned how to use the Configuration system to store user secrets safely so those secrets would not get checked into source control. In a follow-up post, we learned how to use Azure Key Vault to store secrets. In this post, we will address scenarios that may require special code to retrieve rotating or changing secrets either from the Key Vault or another source.

Typically, when your application starts up, the configuration values are read from Environment Variables, appsettings.json files, or other sources. Once read, the values are stored in a Configuration object and not reloaded again (unless certain reload conditions are set and met). If a value were to change and you wanted your application to get the new secret value, you would have to reload the Configuration or restart the application.

If your system depends on frequently rotating key values, restarting the application frequently may not be a good option, nor would reloading the entire configuration. Fortunately, the Configuration system has a way to retrieve the current value of a key without reloading the entire configuration using a Custom Configuration Provider.

### ASP.NET Core Program.cs registration of configuration providers
In a typical WebApi (non-minimal api) or MVC application, the Program.cs file looks like this:

```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```
The somewhat inocuous looking `.CreateDefaultBuilder(args)` contains a lot of default code to setup a typical WebHost. Included in this method is a section of code that establishes a number of different default Configuration Providers, including Environment Variables, appsettings.json files, UserSecrets.json files and default Azure Configuration providers as well as others. If you want to add another Configuration Provider, you can add it to this method, which we will do later.

For Minimal API's, the setup looks like this:
```csharp
var builder = WebApplication.CreateBuilder(args);
// register DI Services here

var app = builder.Build();
// setup middleware pipeline here

app.Run();
```

### Custom Configuration Provider
The Custom Configuration Provider is a class that implements the Microsoft.Extensions.Configuration.IConfigurationProvider interface.  (See [Microsoft Docs - Custom Configuration Provider](https://docs.microsoft.com/en-us/dotnet/core/extensions/custom-configuration-provider) documentation for more information.)

This interface is used to retrieve configuration values from a configuration source. In our code, we will inherit from the ConfigurationProvider class:

```csharp
public class CustomConfigurationProvider: ConfigurationProvider
{
    // This method is optional, but is used to load the configuration provider with data from a source.
    public override void Load()
    {
        // The Load event can house custom code to load data from a source, such as reading a file or a database.
        // Data.Add(Key, Value);
    }

    // This method gets called each time a value is requested from the ConfigurationProvider.
    public override bool TryGet(string key, out string value)
    {
        if (Data.TryGetValue(key, out value))
        {
            value = "Changed the value";
            return true;
        }
        return false;
    }
}
```

If we examine the `ConfigurationProvider` class, we can see that it is an abstract class that implements the `IConfigurationProvider` interface:
```csharp
namespace Microsoft.Extensions.Configuration
{
    //
    // Summary:
    //     Base helper class for implementing an Microsoft.Extensions.Configuration.IConfigurationProvider
    public abstract class ConfigurationProvider : IConfigurationProvider
    {
        //
        // Summary:
        //     Initializes a new Microsoft.Extensions.Configuration.IConfigurationProvider
        protected ConfigurationProvider();

        //
        // Summary:
        //     The configuration key value pairs for this provider.
        protected IDictionary<string, string> Data { get; set; }

        //
        // Summary:
        //     Returns the list of keys that this provider has.
        //
        // Parameters:
        //   earlierKeys:
        //     The earlier keys that other providers contain.
        //
        //   parentPath:
        //     The path for the parent IConfiguration.
        //
        // Returns:
        //     The list of keys for this provider.
        public virtual IEnumerable<string> GetChildKeys(IEnumerable<string> earlierKeys, string parentPath);
        //
        // Summary:
        //     Returns a Microsoft.Extensions.Primitives.IChangeToken that can be used to listen
        //     when this provider is reloaded.
        //
        // Returns:
        //     The Microsoft.Extensions.Primitives.IChangeToken.
        public IChangeToken GetReloadToken();
        //
        // Summary:
        //     Loads (or reloads) the data for this provider.
        public virtual void Load();
        //
        // Summary:
        //     Sets a value for a given key.
        //
        // Parameters:
        //   key:
        //     The configuration key to set.
        //
        //   value:
        //     The value to set.
        public virtual void Set(string key, string value);
        //
        // Summary:
        //     Generates a string representing this provider name and relevant details.
        //
        // Returns:
        //     The configuration name.
        public override string ToString();
        //
        // Summary:
        //     Attempts to find a value with the given key, returns true if one is found, false
        //     otherwise.
        //
        // Parameters:
        //   key:
        //     The key to lookup.
        //
        //   value:
        //     The value found at key if one is found.
        //
        // Returns:
        //     True if key has a value, false otherwise.
        public virtual bool TryGet(string key, out string value);
        //
        // Summary:
        //     Triggers the reload change token and creates a new one.
        protected void OnReload();
    }
}
```

To register a `ConfigurationProvider` in your application, you need a `ConfigurationSource`:
```csharp
public class CustomConfigurationSource : IConfigurationSource
{
    public CustomConfigurationSource() { }

    public IConfigurationProvider Build(IConfigurationBuilder builder) =>
        new CustomConfigurationProvider();
}
```

To register the `CustomConfigurationSource` in the program.cs file, you need to add the following line:
```csharp
.ConfigureAppConfiguration((_, configuration) =>
{
    configuration.Add(new CustomConfigurationSource());
})
```

The complete code for the program.cs file looks like this:
```csharp
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
        .ConfigureAppConfiguration((_, configuration) =>
        {
            configuration.Add(new CustomConfigurationSource());
        }).ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
}
```

For minimal API's, it looks like this:
```csharp
var builder = WebApplication.CreateBuilder(args);
builder.Host.ConfigureAppConfiguration(configuration => {
    configuration.Add(new CustomConfigurationSource());
});
// register DI Services here -- ie. builder.AddSwaggerGen();

var app = builder.Build();
// setup middleware pipeline here

app.Run();
```

If we run the application and put a break point in the `CustomConfigurationProvider` class at the `TryGet` method, we can see the key and value that is being requested:

![debug TryGet method in custom configuration provider](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/custom-config-provider/debug-custom-config-tryget.png)

Each time a dependent service asks the configuration engine for a key, it will call the `TryGet` method of every provider, including the custom provider. The `TryGet` method will return true if the key is found, and will set the value parameter to the value of the key. It is during this method that we can change the value of the key, and use some code to get the value from another location.

### Add Azure Key Vault code
At the start of this post, I talked about reading data from Azure Key Vault, and having it potentially housing rotating credentials. Let's add the code into our Custom Configuration Provider to read from Azure Key Vault.

For this next section, we will need to bring in 3 Nuget packages: 
 * `Azure.Identity`
 * `Azure.Security.KeyVault.Secrets`
 * `Azure.Core`


 And add these using statements to the `CustomConfigurationProvider.cs` file:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```

We are assuming that you already have created an Azure KeyVault service in your Azure subscription. If not, you can follow the post on **Storing Config Values in Azure KeyVault** to create one.

Next, we will add a private method to our `CustomConfigurationProvider` class:
```csharp
private string GetKeyvaultSecret(string vaultName, string secretKey)
{
    try
    {
        SecretClientOptions options = new SecretClientOptions()
        {
            Retry =
            {
                Delay= TimeSpan.FromSeconds(2),
                MaxDelay = TimeSpan.FromSeconds(16),
                MaxRetries = 5,
                Mode = RetryMode.Exponential
            }
        };

        var client = new SecretClient(
            new Uri($"https://{vaultName}.vault.azure.net/"),
            new DefaultAzureCredential(),
            options);
        KeyVaultSecret secret = client.GetSecret(secretKey);
        string secretValue = secret.Value;
        var expires = secret.Properties.ExpiresOn;
        return secretValue;
    }
    catch (Exception ex)
    {
        // you may want to log the exception message, or handle the error in a different way
        return string.Empty;
    }
}
```
This method encapsulates the logic to read from Azure Key Vault. It takes the name of the vault, and the name of the secret key, and returns the value of the secret key.

We can now modify the TryGet method in our CustomConfigurationProvider class to read from Azure Key Vault:

```csharp
public override bool TryGet(string key, out string value)
{
    if (key == "TimeSecret")
    {
        var secret = GetKeyvaultSecret("kvsample11", key);
        value = secret;
        return !string.IsNullOrEmpty(secret); // return true if the secret was found, false if not.
    }

    if (Data.TryGetValue(key, out value))
    {
        value = "Changed the value";
        return true;
    }
    return false;
}
```

This code works to get the secret from our Key Vault and will make a call to the Key Vault on each request for the key. In my experience, calling the Key Vault on each request is not a good idea (it can take several seconds or longer to return), so I am using a cache to store the secret.

First, I will want to add a private field to the `CustomConfigurationProvider` class as well as a constructor:

```csharp
IMemoryCache _cache;
public CustomConfigurationProvider()
{
    _cache = new MemoryCache(new MemoryCacheOptions
    {
        SizeLimit = 1024  // you can determine an appropriate size limit for your application
    });
}
```

Since Configuration Providers are singletons, the constructor should only be called once when the application bootstraps. The MemoryCache should live within the singleton and live as long as the application lives.

Let's modify the TryGet method to add the code to cache the secret:

```csharp
if (key == "TimeSecret")
{
    if (_cache.TryGetValue(key, out string secretValue)) {
        value = secretValue;
        return true;  // found it in the cache, save ourselves some time!
    } else
    {
        var secret = GetKeyvaultSecret("kvsample11", key);
        if (!string.IsNullOrEmpty(secret))
        {
            value = secret;
            _cache.Set(key, secret, DateTime.UtcNow.AddMinutes(2));
            return true;
        }
        else
        {
            value = "";
            return false;
        }
    }             
}
```

### Summary
In this post, we demonstrated how to build a CustomConfigurationProvider that reads from Azure Key Vault. We also demonstrated how to use the MemoryCache to cache the secret.

Using a Custom Configuration Provider is a good way to store secrets in a secure way, but have the flexibility to change the secret on a regular basis.
