PART 3

<iframe width="560" height="315" class="video-frame" src="https://www.youtube.com/embed/Cej_u3fb9rI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Welcome back to part 3 of our series on Authentication and Authorization in ASP.NET Core. In today's post we will learn about IDaaS (Identity as a service) and why you might want to use one. 

We will also demonstrate how to connecte multiple auth challenge or login providers so your users can choose which identity provider they want to use to login to your site. 

There are many IDaaS providers such as Okta, Auth0 and Azure Active Directory as well as self hosted options such as Identity Server or Forgerock. 

We will be connecting to Okta and briefly look at some benefits of doing so.

I have made sample code available on github at  https://github.com/mobiletonster/authn.

IDaaS providers are different than social login providers like Facebook and Google. The IDaaS instances are yours and you can add users to them just as you would if you had a table of users in your own system, but remember that a hosted provider has already taken care of the details such as encrypting the user's password, or giving the user recovery options if they forget their username or password and can provide multifactor authentication tools such as SMS codes or even facial recognition with standards like FIDO. Most hosted providers offer a pretty generous free tier as well.

Let's get started with Okta.

At https://developer.okta.com  sign up and create a new account

Once you have completed the steps to register for a developer account at Okta, go ahead and sign in with the credentials used to create the account.

This should take you to the dashboard. Click on applications and add an application. The developer account allows you to have up to 5 different applications registered before you need to pay.

Click create a new app. There are different types of apps you can create.
 * Web
 * Native
 * Single page
 * Service
 We're going to create a web application using OpenId connect.

 ** POST UNDER CONSTRUCTION **

Be sure to check out the sample code in Github and visit my website at Mobiletonster.com. You can find me on Twitter @mobiletonster and feel free to leave me comments or questions at the website, on twitter or on YouTube.

Once again, thanks for watching!

<iframe width="100%" height="450px"  class="video-frame" src="https://linkto.run/p/SO9LHOAP" title="Part 4 Interest Poll" frameborder="0"></iframe>