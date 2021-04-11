In part 3  of our Authentication in ASP.NET Core 5.0 series, we examine the IDaaS (Identity as a Service) specifically Okta and we implement multiple social login providers to let our user decide which provider they would like to use to login to our site.

Authentication in ASP.NET Core 5.0 series:
https://mobiletonster.com/area/code - website with companion articles & code
PART 1 - (Authentication/Authorization & Cookies) - https://youtu.be/BWa7Mu-oMHk
PART 2 - (this video) - https://youtu.be/K0Z7GTOvbMo
PART 3 - (IDaaS & Multiple Providers) - https://youtu.be/Cej_u3fb9rI
PART 3a - (Connect SQLite DB) - https://youtu.be/z-Hll4Xddjs

Chapters
0:00 Intro
0:36 Review Part 1 & 2
0:50 Overview
1:56 Register Okta dev account 
2:55 Create new app at Okta
4:50 Adding Okta to startup.cs
8:00 Testing Okta in our app
8:25 Error - add user to app in Okta dashboard
9:37 Permission denied - missing admin role
12:00 Using Okta groups for role assignment
17:00 Summarizing Okta
17:20 Adding multiple login providers
19:46 Making the login providers generic
22:47 Testing multiple providers
23:40 Problem with logout
24:44 Capture login scheme with events
26:06 Trying the Cookie Auth Events instead
32:08 Adding .AuthScheme as claim
33:06 Fix logout problem
35:41 Adding SignedOutCallbackPath in startup
36:55 Register callback path at Okta dashboard
38:45 Setting Scope properties in OIDC
39:53 Why use IDaaS?
40:38 Using database to capture and register users from social login providers
41:47 Database magically appears in project
42:04 Adding UserService.cs
48:32 Connect UserService in HomeController, TryValidateUser
50:12 Error: Unable to resolve UserService
50:28 Register UserService in DependencyInjection in Startup.cs
51:26 Getting user role from the database
54:52 Create add new user method
56:05 Writing extension method to ease getting claims properties
58:58 Connecting add new user method to Cookie Auth Event
1:01:26 Testing the add user method
1:03:01 Okta and the missing first & last name fields
1:05:35 Final Thoughts

Intro includes music:  
Spend My Time With You (Electro Swiremng Remix) - 11 Acorn Lane
http://www.11acornlane.com
https://www.youtube.com/watch?v=zPMNu8pX2zw