## Deploy to Azure App Service - Easy
So you have created your ASP.NET Core app and you want to deploy it somewhere. You decide you want to deploy it to Azure App Service because you have heard good things about it. 

If you are going to be deploying a production grade application, you should really use CI/CD (Continuous Integration/Continuous Delivery), however, if you are just experimenting and needing to deploy very quickly, you can do so directly from Visual Studio if you have a subscription to Azure.

### Signup for Azure Account
Make sure you have signed up for an Azure account. Head over to https://azure.microsoft.com and signup for a free account trial.

![azure free signup](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/1-azure-free-signup.jpg#screenshot)

### Load Your App in Visual Studio
Open Visual Studio and load your project that contains your previously created web application.

In my case, I have a sample application called "MyWebApp". Right click the project in the solution explorer window and select the "Publish" menu item.

![publish menu item in solution explorer](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/2-publish-menu.jpg#screenshot)


### Publishing your app to Azure
In the publish (target) dialog window, select "Azure" and then click "Next".

![publish dialog window](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/3-publish-dialog.jpg#screenshot)

In the next frame  (specific target) of the dialog window, you have options for:
1) Azure App Service (Windows)
2) Azure App Service (Linux)
3) Azure App Service Container
4) Azure Container Registry
5) Azure Virtual Machine

![publish dialog continued](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/4-publish-dialog2.jpg#screenshot)

For the purpose of this post, we will choose to deploy to either option 1 or 2, Azure App Service for Windows or Linux. I am going to select option 1 and click the "Next" button.

When the next frame loads (app service) we can select the subscription, view type (resource group or services) are given a search box and can pick from a list of instances to deploy to.

Before we move on, lets talk about resource groups. Resource groups are a "collection" of different azure services, grouped together. For instance, your app may require a web host (app service), and a database, some blob storage (or a place to store persisted files such as images that aren't deployed with your web app) and an azure function. Resource groups let you keep all of these related serves "bundled" together so you can track them more easily.

First, lets create a new app service instance since this is the first time we are deploying. Once created, we won't be prompted for this during future deployments.

Click the green '+' symbol just above the list of "App Service Instances" to create a new app service.

![app service azure deploy dialog](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/5-app-service-dialog.jpg#screenshot)

A new dialog for the "App Service - Create New" should appear.

We will give it a "Name" for our application. The name you choose will become part of the public url you will be assigned and will need to be unique (no other person has used that name). For instance, if no one else has registered **"mysecretwebapp"** as a name for their application, I can set that as the name. After we deploy, the app will be reachable at "https://mysecretwebapp.azurewebsites.net".

I will assume that we need to create a new resource group for the purpose of this deployment as this is the first time we are deploying it. Click "New..." next to the Resource group field.

![create new appservice](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/6-create-new-appservice.jpg#screenshot)

In the new resource group dialog, give it a name. This needs only be unique within your subscription and is not global like the app service name, so you won't be competing with others for a cool name here. I named mine "SecretSampleGroup".

![Create new resource group](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/7-new-resource-group.jpg#screenshot)

Next, click the "New..." next to the Hosting Plan field. This will let us choose a plan for hosting and a data center location. It ranges anywhere from free to rather large computing resources (and can get expensive. Definitely for those who need such power). 

First, name your plan. I named mine "mysecretwebappPlan".

Next select a location. I chose West US, but you can choose any data center location you would like.

Finally choose the size. I chose "Free" because I don't need performance and scalability for this sample, test application.

![create new hosting plan](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/8-new-hosting-plan.jpg#screenshot)

Once this dialog is completed as in the screenshot below, click the next button to continue on.

![publish new web app dialog completed](images\azure-ezdeploy\9-publish-new-webapp.jpg#screenshot)

You will see a new "Public" page in your Visual Studio project. This is the page you will see when you publish from now on. 

![publish page in visual studio](images\azure-ezdeploy\10-publish-dialog.jpg#screenshot)

This is saved in your solution as an XML file:
![pubxml file for web deploy settings](images\azure-ezdeploy\11-webdeploy-xml.jpg#screenshot)

Back to the new publish page, when we are ready to deploy, simply click the "Publish" button on this page and your app will begin to deploy.

![publish page in visual studio](images\azure-ezdeploy\10-publish-dialog.jpg#screenshot)

On the first attempt, it will check to make sure that the deployment will be successful and is looking for things like .net 5.0 version support among other things.

![publishing checks](images\azure-ezdeploy\12-publishing-checks.jpg#screenshot)

After the checks complete, it will begin deployment (assuming the checks were successful). On success you should see a screen like this:

![publish suceeded screen](images\azure-ezdeploy\13-publish-suceeded.jpg#screenshot)

The publish step will even launch your default browser and load the page so you can test it.

![running app in microsoft edge](images\azure-ezdeploy\14-app-running.jpg#screenshot)


### Editing publish settings
You may want to adjust some of the settings later. To do this, click the "Show all settings" link as circled in the image below:

![publish page in visual studio with all settings link circled](images\azure-ezdeploy\15-show-all-settings.jpg#screenshot)

This will load a new dialog box with two tab sections "Connection" and "Settings".

![connections tab of edit settings dialog](images\azure-ezdeploy\16-connection.jpg#screenshot)
 
 In the "Connection" tab, you will see the publish method, server, site name, user name, password and the destination URL. All of this will get checked into source control EXCEPT the saved password which is only on your local machine. Keep this in mind if you are working with another person or on another machine.

 ![settings tab of edit settings dialog](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/17-settings.jpg#screenshot)

 The settings page include configuration, target framework, deployment mode, target runtime. Usually, I leave these alone as the defaults are fine in most cases.

 Under File Publish Options, there is a checkbox to "Remove additional files at destination". This can sometimes be an important setting. If you made a change that needs to remove a file, this check box will be necessary to keep your project and the publish site in sync.

 Additionally, you can specify a connection string for databases from your secrets json, etc. To see how this works, I have added a couple of values to my secrets.json file:

 ```json
 {
  "Passkey": "secrets.json",
  "ConnectionStrings": {
    "AzureDeployDatabase": "secret connection string"
  }
}
```

In the publish dialog, under Database you will see that the dialog discovered two keys: 
* PassKey
* AzureDeployDatabase (under ConnectionStrings)

![checkbox for AzureDeployDatabase key](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/18-connection-string.jpg#screenshot)

If you place a checkbox next to "Use this connection string at runtime" and choose "secret connection string", this value will be deployed with your application although it isn't checked into source control.

This is a nice way to keep your secrets secret, but still deployed.

Let's see what happens when we deploy this.

![output window for publish](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/19-publish-output.jpg#screenshot)

If you look in the output window for the build/publish, you will se that it creates or updates an `appsettings.production.json` file even though one doesn't exist in our solution explorer. This file is dynamically created and published to azure.

![appsettings in kudu console](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/20-appsettings-prod.jpg#screenshot)

This screenshot is from the Azure portal in the Kudu tools which allow you to explore the file structure of your deployed app in Azure. Here we see that the new appsettings.production.json file exists on the server!

![appsettings prod content](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/cloud/images/azure-ezdeploy/21-appsetting-contents.jpg#screenshot)

In the deployemnt, you will see our ConnectionStrings key with the AzureDeployDatabase specified.

### Summary
In this post, we have examined the power of the **"Publish"** button in Visual Studio and how easy it is to deploy an application directly to Azure. This is great for quickly deploying sample applications or test apps, however, if you are deploying a production class application, you should really setup CI/CD (Continuous Integration/build and Continuous Deployment). 

In a future post, we will examing other ways to  configure secrets for Azure deployed applications.

