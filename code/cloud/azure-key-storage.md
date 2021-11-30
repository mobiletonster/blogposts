## Storing Config Values in Azure
In this article, we will demonstrate 2 primary ways of storing secrets data or configuration information for application deployed to Microsoft Azure:
1) Azure App Service Configuration
2) Azure Key Vault

In a previous post, we learned about keeping user secrets out of our code and out of config files that should not be checked into a code repository, particularly during development. In that post, we demonstrated how to use environment variables to set values on a server so that our code could get the config values it needed when it was deployed.

In this post, we will learn about alternative ways to store those values in Azure (if you deploy there) to support App Services, Functions, etc. using App Service Configuration and Azure Key Vault.

## Azure App Service Configuration
First, we are assuming that you have already created an Azure App Service instance and likely have deployed an application and are now awaiting a way to set environment variables or other secrets in the deployed configuration. If not, you can check out the blog post on "easy deploy to Azure App Service".

Navigate to the Azure Portal at https://portal.azure.com and login to your subscription.

![](images/azure-key-storage/1-azure-services.jpg#screenshot)

Locate the app service on the portal. Mine is labeled "mysecretwebapp". Click the name to open the app service portal page.

![](images/azure-key-storage/2-app-service-portal.jpg#screenshot)

Here you will see the left side navigation bar with several options. Under the settings section, locate and click "Configuration".

![](images/azure-key-storage/3-configuration-panel.jpg#screenshot#screenshot)

The configuration panel is divided into 4 tabs:
* Application settings
* General settings
* Default documents
* Path mappings

We are interested in the first tab for this exercise, "Application settings".

### Add New Key/Value Pair

On this tab, there are two sections "Application settings" and "Connection strings". We will add a custom key/value pair to the first section. To do this, click the "+ New application setting" link.

![](images/azure-key-storage/4-add-app-config.jpg#screenshot)

The Add/Edit panel will appear. I added a key of "Passkey" with a value of "From Azure Config!" as depicted above. Then click the 'OK' button. Because we are using a free tier, we don't have access to deployment slots, so you can ignore the "Deployment slot setting" checkbox. If you had a deployment slot, such as a "staging" instance, you could assign the key to a specific slot.

![](images/azure-key-storage/5-save-new-key.jpg#screenshot)

Once you have added your new key/value pair, it will appear in the configuration panel, HOWEVER, you must click the save button at the top of the panel to apply the key or it will not be saved.

Once you do this, you will receive a warning:

![](images/azure-key-storage/6-save-warning.jpg#screenshot)

The save changes dialog warns that these changes will cause your application to restart. Once your application restarts, it will have access to the new configuration value.

### Nested key values
In our appsettings.json and secrets.json files, we can have nested settings for organizational purposes. This is usually represented like this:

```json
{
    "MyConfigValues":{
        "Passkey":"secret value"
    }
}
```
In code, we access this value using the `Configuration` object like this:

```c#
string Passkey = Configuration["MyConfigValues:Passkey"];
```

The nesting is accessible using the ':' colon character as a delimiter.

In contrast to the `json` configuration nesting, the Azure portal is a bit different. To store the value as a nested value, use two `__` underscore characters to seperate the parent and child.

![nested key using double underscore in azure config panel 'Nested__Secret'](images/azure-key-storage/7-nested-secret.jpg#screenshot)

To retrieve the value in code, no changes will need to be made as the code in C# still uses a ':' colon character to delimit the parent and the child.

```c#
string Passkey = Configuration["Nested:Secret"];
```

## Azure Key Vault
While using configuration settings in Azure App Service is a convenient way to store secrets, another way is to use Azure Key Vault. The key vault can be shared across mutliple app service instances, function apps and other services. It is a general purpose, secure way to store secrets.

To create a key vault from the Azure portal, search for Key Vault and click the Create button to begin.

![Key Vault creation screen on Azure portal](images/azure-key-storage/8-create-key-vault.jpg#screenshot)

Next you will be present with the 'create key vault' screen where you select the subscription, resource group and assign a name, region and a pricing tier. For this example, I named my keyvault `kvsample11`, selected the `South Central US` region and opted for the `Standard` pricing tier. Additionally I enabled soft-delete and allowed for 7 days to retain soft delete values. I also disabled purge protection so that a key could be purged even before the 7 days elapsed.

![key vault setup screen](images/azure-key-storage/9-key-vault-setup.jpg#screenshot)

For the next step, setting up access policies, we will leave the default as `Vault access policy` and click `next`. Later, we will return to this screen and configure access policies for our application.

![key vault access policy screen](images/azure-key-storage/10-access-policy.jpg#screenshot)

On the `Networking` tab or step, you can set the connectivity method to match whatever your application needs. In this case, I just set it to a Public endpoing for now.

![key vault networking tab/screen](images/azure-key-storage/11-networking.jpg#screenshot)

You can add tags to describe the resource or help you find it easier.

![key vault tag screen](images/azure-key-storage/12-tags.jpg#screenshot)

Finally, review your settings and when you are ready, click the blue `Create` button below.

![key vault final review screen](images/azure-key-storage/13-review-screen.jpg#screenshot)

In a few minutes, your new azure key vault will be provisioned and ready.

### Setting Access
Before we tackle the next step, we need to talk about how Key Vault is secured and different ways that applications/services can talk to key vault to get secrets from it.

The idea of key vault is to keep secrets out of your code, especially out of your code repository. But how do you access the secrets in the vault? 

There are several ways to do this, including setting up (ironically) a username/password combination that you can use to communicate with the key vault. However, this seems to lead back to the same problem we started with, avoiding the storing of secrets with/in your application code.

A better approach is to use an Azure `managed identity` to access the key vault. This way, Azure can explicitly allow an application (hosted in an app service, for example) to talk to the key vault without needing to pass any secrets, but the application must be approved to communicate with it.

There are several ways to accomplish this, including setting it up through the identity tab in the app service itself, but for the purposes of this tutorial, we will use the Access policies panel of the Key Vault itself.


![Access policies tab](images/azure-key-storage/14-access-management.jpg#screenshot)

From the Access policies panel, click the `Add Access Policy` button.

![Add access policy screen](images/azure-key-storage/15-add-role-assignment.jpg#screenshot)

On the add access policy screen, choose `Get` and `List` from the drop down box next to `Secret permissions`.

![select principal for add access policy screen](images/azure-key-storage/16-members.jpg#screenshot)

Now, click the `None selected` link next to `Select principal` to open the Principal dialog. Then search for the name of your app service (in this case mysecretwebapp), then select it and click the `select` button.

![Add access policy screen continued](images/azure-key-storage/17-select-member.jpg#screenshot)

now click the `Add` button to complete the add policy process.

![Access policy screen, save changes](images/azure-key-storage/18-select-app-service.jpg#screenshot)


### Add a Secret
Now that we have our access rights assigned to the app service, it is time to populate our key vault with a secret. To do this, go to the `Secrets` tab in the vertical navigation panel for the key vault and select it.

![add secret](images/azure-key-storage/24-add-secret.jpg#screenshot)

Then click the `+ Generate/Import` button at the top.

![create secret manually](images/azure-key-storage/25-create-secret-manually.jpg#screenshot)

On the `Create a secret` screen, select Manual. Then, give the secret a name, in this case `MyAppSecret` and a value, like `fluFFyC@t35!`, or something fun. Then click `Create`.

This will add a secret that we can later pull into our application when deployed to our Azure App service.

### Install the packages
Return to our sample app, 
From the terminal window, install the Azure Key Vault secret client library for .NET and Azure Identity client library packages:

```console
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

Additionally, you can use the `Manage nuget packages` GUI if you prefer by right clicking the project and choose `Manage nuget packages`


### Update The Code

Find and open the Startup.cs file in your webapp project. 

Add these lines to the header:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
using Azure.Core;
```



```csharp
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
var client = new SecretClient(new Uri("https://<your-unique-key-vault-name>.vault.azure.net/"), new DefaultAzureCredential(),options);

KeyVaultSecret secret = client.GetSecret("<mySecret>");

string secretValue = secret.Value;
```