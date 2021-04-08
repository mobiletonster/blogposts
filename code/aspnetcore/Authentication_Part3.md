PART 3

Welcome back to part 3 of our series on Authentication and Authorization in ASP.NET Core. In today's video we will learn about IDaaS (Identity as a service) and why you might want to use one. 

We will also demonstrate how to connecte multiple auth challenge or login providers so your users can choose which identity provider they want to use to login to your site. So stick around!

[Play the intro video]

In part one of our series we learned about cookie-based authentication in asp.net core and discussed the difference between authentication, authorization and challenge. 

In part two, we learned more about the challenge part of the authentication process and connected to a third party OpenId provider, Google.

There are many IDaaS providers such as Okta, Auth0 and Azure Active Directory as well as self hosted options such as Identity Server. 

We will be connecting to Okta and briefly look at some benefits of doing so.

I have made sample code available on github at  https://github.com/mobiletonster/authn.

Additionally, I will be posting code and articles on my blog at https://mobiletonster.com.

IDaaS providers are different than social login providers like Facebook and Google. The IDaaS instances are yours and you can add users to them just as you would if you had a table of users in your own system, but remember that a hosted provider has already taken care of the details such as encrypting the user's password, or giving the user recovery options if they forget their username or password and can provide multifactor authentication tools such as SMS codes or even facial recognition with standards like FIDO. Most hosted providers offer a pretty generous free tier as well.

Let's get started with Okta.




