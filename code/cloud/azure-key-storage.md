## Storing Config Values in Azure
In this article, we will demonstrate 2 primary ways of storing secrets data or configuration information for application deployed to Microsoft Azure:
1) Azure App Service Configuration
2) Azure Key Vault

In a previous post, we learned about keeping user secrets out of our code and out of config files that should not be checked into a code repository, particularly during development. In that post, we demonstrated how to use environment variables to set values on a server so that our code could get the config values it needed when it was deployed.

In this post, we learn about alternative ways to store those values in Azure (if you deploy there) to support App Services, Functions, etc. using App Service Configuration and Azure Key Vault.

### Azure App Service Configuration
