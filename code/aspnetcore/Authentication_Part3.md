PART 3


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

now i'm going to give this a name we'll just call it off sample app i could add a logo or whatever but i don't need to do that for now so i'm just going to add a our this is important our

redirect uri now before when we were connecting the google uh when we just we were listening on slash auth for the callback but if we add two of them we can't have them both listening on slash auth so i'm going to be specific here and call this one octa dash auth

so now that we've created our application it's given us a client id and a client secret this is the domain that we have assigned to us with a paid account i believe you can put a custom domain on this and maybe you can't even with the free account i'm not sure but this will be good enough for what we're trying to do there are different grant types that we can use we could do a client credentials grant or an authorization code flow or an uh implicit slash hybrid flow but for what our application is using we're going to use the authorization code flow or that that type of a grant the check box for require consent is checked and here's our callback our login redirect uri octa dash auth that we just set up on the previous screen i'm not going to change anything everything here looks really good but i need to grab the client id the client secret and the octa domain it's time to return to our application and we're going to go back to our off sample application into the startup and before where we had openid connect google register i've changed it to be a lowercase g for google and this will make more sense later why i did that

but we're going to add a new one and i'm going to specifically call it octa with lowercase

and we're going to just set this up with the authority and the authority is is the domain which is this domain right here

so we'll set that up and then we need a client id

i believe this is the client id

and we need the client secret

and that should be the client secret we need a callback path

and we set it up to be octa auth so now that we have two openid connect providers registered google and octa we need to now change the authentication default challenge scheme we're now going to just switch to octa we'll change this in a moment if i don't switch it to octa it's going to automatically redirect to google for sign in let's just run this and see how far we get

so if i click on secured should now redirect me uh oh we have a problem the response type is not supported by the authorization server configured response types code error error uri okay this makes sense

i think one of the most frustrating parts of openid connect is that developers often find that different providers implement different parts of the specification so it makes it it makes it a little bit challenging sometimes to discover what options you need to turn on for which different providers and this is really just there's not a lot you can do about it you just need to understand how each of the different providers are expecting their configuration to be set up so octa requires that we set the response type and we want the response type to be code now there are some constants that you can use but sometimes i think it's harder to remember that but it's open id connect response type dot code and if you hover over that you can see it's a constant that is equivalent to the word code so in this case i know it's probably better to use constants but i can spell code it's four letters hopefully i get that right let's run this and see if that makes things better

okay so this is an interesting error it says open id connect protocol exception message contains error access denied error description is the user is not assigned to the client application all right what does that mean well this is actually kind of an interesting error so if we head back to our octa dashboard let's take a look and see what that means so we've created an application called auth sample app but let's if we look at assignments we'll see that no people are assigned to use this application this is one of the reasons i wanted to demonstrate using an idp such as octa or auth0 or even active directory they behave a little bit differently than google facebook and twitter with google facebook and twitter you don't have any control over the user in the idp but here you do because this is your space your tenant in the octa idp i have the rights to assign people to applications or i can assign groups to applications or whatever so if i go in here and click assign i can assign this application to people and here's bob so now if i assign bob

now bob is assigned to this application if we return to our application and we try to go through the sign in process again click on secured redirects okay so now it made it all the way through and we are logged in but we've hit our denied url so now that he has rights to our application according to octa the next thing we need to figure out is why doesn't he have rights to this particular page now if you recall this authorized attribute over the secured endpoint requires a role of admin and we haven't really given him a way to be an admin let's take off that requirement that you have to be an admin for the time being and let's just mark it with authorize rerun the application and see if our authentication is working the way we think it should be

okay so now we're logged in as bob and because it doesn't require us to be an admin at the for the time being it's allowing us to to get in here and this looks pretty good here's his name identifier and this is the unique id that a user gets assigned in octa so when a new user signs up and adds himself to our space in octa they're automatically given a unique key that represents them his name is bob coder his preferred username is bob at edmonics.com and that's his auth time some pretty basic stuff there and that looks pretty good so if you recall what we had to do with google remember we had to attach ourselves to the on token validated event and we had to look at his claims to see if he had a if his claim his name identifier his unique id in google which was this then we knew it was bob and if it was bob coming in we'd create a new claim of type role and make him an admin and add that to his list of claims that happened during the on token validated event for google now we could go and do the same thing with octa so we'd have to look for this name identifier in the case of octa to know that it's bob coming in but when you're working with an idp like octa or off0 or active directory you have some other options here and that's kind of what we want to look at next is some different ways to identify that bob is an admin or in this case belongs to an admin group

returning to our octa dashboard remember we said we could assign groups right now we have two different groups set up i created one earlier called admin or administrator group and then in everyone group what we could have done is rather than individually assign bob access to this application by default he's in the everyone group i could have just assigned the everyone group to this application and then anybody who's in that group would automatically be able to use that but i want to assign the administrator group writes to use this application and i think i'll go ahead and assign the everyone group too while we're at it so now we have two different groups here that are that are assigned to use this application let's go take a look at the admin group itself who's in the admin group well i had already added bob to the admin group now how do we know that he's in the admin group that's the next thing we need to figure out back of the dashboard i'm going to go down to security and i'm going to select api now if you don't have this api option you may need to go to your settings and go to features and check app configuration api access now with that checked i should be able to go to security and select api so now we're looking at authorization servers we have one that's predefined for us this default authorization server let's go in here and what we want to do is add a claim

so let's add a specific claim and i'm going to call this roles and i want this to be in the id token always and we're going to use groups and we can use starts with equals contains but in this case i just want to include a regex pattern dot star and include this in any scope

now once i do this i have this new claim type of roles that matches this regex pattern based on groups that the user belongs to so if i go look at the token preview now this is kind of a cool little tool that they give you so i find my app the auth sample app we're using authorization code and i look for a specific user in this case bob he's the only one we have so and then i choose the open id scope to preview the token and this shows me what the token is going to look like and you'll notice in here there is a new claim in here for roles and since bob belongs to the everyone group and i've added him to the admin group both of those show up as roles this is pretty exciting let's go to our application and run this and see if now when we log in is bob he gets new claims added to his token looks good

okay we've made it all all the way through and we've signed in but i don't see any new claims in here for the roles that we added something's missing sorry to disappoint you it didn't work let's see if we can fix this

we go back and look at the dashboard for a minute one thing that you'll notice is that the issuer uri has changed slightly before when we put in our authority we only put https dev dash whatever octa dot com slash but notice it has oauth 2 slash default default on here now i'm going to copy this and come back to our code and change the authority to this new address which helps our application talk directly to that authorization server let's run this again and see if that fixes things let's try this click secured sign in as bob return to the app and notice magically we have two new claims there there of type role and it has declared him as having the claim for everyone role and admin role that's pretty cool so now this role has showed up we should be able to go back in our application to our home controller and re-add the authorization with roles equals admin and our application should still work and it led us through this time and we still see that all of his information so it is working now even with our authorized route requiring an admin role in just a few minutes we provisioned a new octa account designated a new application in octa implemented the credentials in our app including using group assignments as a custom role claim in our id token right now our application is only using octa as a sign in option even though we have also registered google as an idp with our application the next step will be to allow a user to select which way he wants to log in back in our code let's return to our startup cs file if you recall on line 37 we changed the default challenge scheme from google to octa but what we really want to do here is change this to be cookie authentication defaults authentication scheme and what that's going to do is whenever there's an opportunity for a challenge to happen or we need to prompt the user for credentials rather than automatically going to either google or octa or facebook or twitter or github or whoever whichever providers we add what we're going to do is we're going to say hey the user needs to log in we need to challenge them where should we go we've said cookie you're in charge of that and cookie says oh i'm going to take them to a route login let's go to the home controller and look at that route for login so the route login will return a view for logging let's go look at the view under home login cshtml the logincs html page has a form on it that accepts a username and a password and a submit button we created this form during the first video that we recorded on cookie authentication where we made a custom login screen we need to add some links to this custom login screen giving the user an option to log into google or octa in addition to logging in with the username and password okay i've added these links you can see i have an href google octa with the return url log in with google log in with octa if we look in the home controller i've already written some code here for login google that would accept google's return url in the google action method here and it would call the challenge async google authentication properties configure weight vaults etc etc however i don't really want to use this code this way i mean what i could do is i could make a copy of this and i could paste this and i could have an entry here for octa make all the changes for login octa and so on and so forth but if i were to add a whole bunch of providers this could get pretty repetitive so let's see if there's a better way to do this i'm going to undo these changes here and although i want to follow this particular pattern let's make it a little more generic in here i'm going to put curly brace provider curly brace and i'm just going to change this to be generic login external

and then the string that comes in for the return url is going to come from the query string but we will also get the provider from the route now technically if i wanted to be explicit i could add things like this comes from the route

and this comes from the query and that would be okay to do but the model binder in asp.net core is really pretty good it will look to find a match and it just works anyway i'm going to leave that in there for now just to make it clear where these are coming from i'll leave this in here that says if if we already have a user that's logged in then there's no point of taking them to a login just take them home here's the return url string is null or empty return url in other words if we don't have a return url passed to us then designate the slash which is represented the home page as the return url otherwise take the return url and then we need these authentication properties where we pass the redirect uri in is the return url finally in our previous video we used the await http context challenge async passing in the name of the scheme google and the authentication properties to perform the challenge and to redirect to google's login page but obviously in this case we don't want to pass google we want to pass the provider that way if it's octa it will call the challenge async portion from octave versus the challenge async method for google now there's something else we can do here i wanted to show using the hdb contents challenge async last time but i think this time i want to show something a little bit different there is an alternative we can return a new challenge result and that challenge result has six overloads one that takes authentication properties ones that takes the names of the scheme one that takes a list of schemes or authentication scheme comma properties and that's the one i want to use provider authentication properties now notice there's no wait or anything like that it just it should just work except we need to change this to be return i don't think it needs to be async anymore i action result

so we don't need to use this

i think that should work i think i want to run this let's see what happens

so now we have our login page we can use a username and password and sign in or we can log in with google or octa so let's start with google

it redirected me to google's login page and i'm going to log in as bob tester and now i'm logged in so let's log out again

and this time i'm going to click secured and i'm going to select login with octa now it redirects me to the octa login page and i'm going to sign in as bob coder and sure enough this time we're bob coder not bob tester and we're logged in and we have our admin and everyone rolls so that's pretty cool that was pretty easy to connect now one thing i noticed is when i clicked log out it took me to the google logout page even when i was logged into octa we should probably fix that so let's take a look at our logout code currently this is the logout code we have we call await httpcontext.sign out async and return redirect to this google page and it has to be this way unfortunately because google uses a unique way to log out if you actually want to log out of their app engine you have to call this redirect so i guess the big problem here is that even though we can call logout we really don't know at this point what challenge scheme was used by the user or which identity provider they selected when they logged in we really need to know that so that when they log out we choose the correct logout method i looked around a little bit to see if there was an easy way to discover which challenge scheme was used when a user logged in and there really isn't let's just take matters into our own hands so let's go back to startup and we'll look at our authentication setup here so we know we have events in openid connect that we can hook into we're already connecting into the on token validated event for google so let's put a break point in here and just see what other information we have available at this point in time

i'm going to choose google so that we land in that

so if we inspect the context and if we scroll down there is a scheme here and if we inspect the scheme it gives us the name of the scheme that's being used which is google which would make sense because we're inside the openid connect google handler so that's one way we could do it we can check to see if the scheme is google and if it is we can add a claim that says scheme and the name of the scheme google and add that to as a property for the user who's signed in then we know whether he sided with google or octa or if he just signed in with cookie authentication so that would work and we would have to add that for every openid connect provider we attached but i'm wondering if we could take advantage of the second phase of events inside of the ad cookie options so let's take a look in here

do events equals new cookie authentication events and the event that i'm going to be caring about i think is on signing in during the signing in process it's a two-step process remember once add open id connect is completed it will call and fire the on signing in event of the cookie authentication handler

so we'll demonstrate this

and we'll put a breakpoint in here and say scheme equals context dot

scheme and we'll see what we get here

okay we've hit the first break point i'm going to continue on the second break point inspect the scheme this time it said cookies so scheme probably isn't going to help us inside of cookies during the unsigning in event however i do know that there's this thing called authentication properties and if we look in the items instead of authentication properties you'll notice that this authentication scheme it says google right there i think we can use that to our advantage so let's write a little bit of code here that fixes this so we want to be in context.properties which is authentication properties and there's a list of items and we want the item where the key is equal to and if you were paying attention i believe it starts with dot auth scheme

and we'll just say the first or default

okay i'm going to take this breakpoint off and we'll run it through

okay we hit the second breakpoint and this time we got the key value pair back dot oscheme and the value is google now let's test that if we were to go ahead and log out of google this time click this again and let's sign in with

octa

now if i check the scheme it says dot o scheme as octa so we have one more we want to check

and our log out still isn't correct because it's taking us back to google but now i'm going to use

so it's bob not blob let's try this again it's bob and pizza okay so we're signing in and this time the scheme is null null interesting so if we look under properties what do we see two items and issued and expires okay so we could look for null null and know that that was the built-in authentication i think we can fix this by going into the home controller and going to our validate event where we validate the username and password and there's an overload for signing async that accepts something called an authentication properties so we're going to add something there let's call var properties equals new authentication properties

and we'll put properties here

now authentication properties has a couple of overloads see what they are

dictionary of string string for items so we'll want to create a new items equals new dictionary of string comma string

and then we'll say i don't oh did i say item i want items items dot add and then we'll do the key being auth scheme dot ah scheme and the value will be cookie authentication defaults dot authentication scheme

put that in there so we'll pass items into our new authentication properties and we'll pass properties into sign in async so a few extra steps there but basically what this is going to do is it's going to add an entry for dot auth scheme and the entry will be this constant which is cookies and we'll put it inside of the authentication properties and that should properly pass that all the way through from our challenge this is kind of our challenge local challenge scheme and when it calls sign in async it should land inside of our startup for cookies so we'll watch this in action and now we have a scheme there because if we look in the items there's three items and we've added this dot dot auth scheme and cookies so that should take care of our three different scenarios google octa and cookies now that we have our auth scheme passing all the way through we need to add that as a claim so that it is part of that user's identity and steal some of this code here and replicate it up here and what i'm going to do is we're not going to add this as a type we're going to call this scheme dot key comma scheme dot value

and then we'll add it to the claim and that should add it to our identity so let's see if that works so that works we now have an additional claim property for a auth scheme and the value is octa

i believe we have everything in place now to go and fix the logout process so let's go to home controllers and let's go to our logout code currently this is what it looks like it's a simple sign out but let's see if we can fix this up so the first thing i want to do is i want to know what scheme was used so let's see if we can figure that out i think the easiest way to do that is go to the user

user.claims.first or default where the claim or the claim type is going to be equal to dot auth scheme that's what we're looking for and we'll get the value out from there value value let's try that again okay we got that let's run it to this point and see if we get that value back this time i'm going to use google

now when i press the log out button did we pick up the scheme and it says the scheme is google so i think we're good there we now know the scheme that was used to sign in

because google is a little bit different i'm going to check to see if our scheme was google if it is then this is the way we want to perform the the log out

and google's a little bit different because you have to redirect to this location if you want to log out of your google account and not just out of your application but for everything else we're going to do it this other way you sign out result there it is and the sign out result takes a number of different overloads but the one that i'm interested in is the list of authentication schemes so i'm going to try and do this inline we'll just do a new array and the two schemes are cookie authentication defaults dot authentication scheme and the other one is the scheme that was passed through or the scheme that the user signed in with it during the challenge step we want to sign out of both of those so that we sign out of our application and out of the identity provider and we just want to return that and i think that should take care of it there are a couple more things we need to do if we go back to our startup we need to add a couple of things here so we have an option for the callback path and you'll notice there's two callback paths there's also one for the signed out callback path and i want to make this one octus sign out it can be anything we want by default it is something like sign out callback oidc or something like that but since we have multiple providers it's best if we are specific about our callback and the other thing we're going to need and i was debated whether to put this in and just run it and find the error but let's just go ahead and put this in save ourselves sometimes we will need to save the token

the reason that is is during the logout process octa needs to sign out using the token so we will actually be sending the token back to octa to sign out now we won't actually have to do any of this the framework will take care of all of this for us but that is a requirement that they send the token back and it's kind of all built into openid connect in their protocol so

finally we need to go back to the octa dashboard go back to applications click on our off sample app and in the general settings section here you'll notice that we have a login redirect uri but we didn't specif specify a logout redirect uri or the location to return to after octa is done doing the logout process so i'm going to click edit and i'm going to add a logout uri and this needs to be

our octa-sign out that we registered with the middleware or with the handler i should say this this means when octa is done signing out it will return to this location to complete the process

now let's run it and see if all of this works

log in with octa

sign in now click log out redirected all the way back and we are logged out again

so let's take a peek and see if we can see what happened i'm on the network tab here i'm going to go and log out we'll watch all the steps

so as part of the log out process it's kind of long here but the most important part of this url is if you look at the id token underscore hint equals and then the token gets embedded in that url and sent back to octa that is what they need to be able to sign the user out of octa and then return back to us

one final piece i wanted to show are a couple of settings that we didn't explicitly set but you can and that is

the ability to add specific scopes now we didn't add these because

the handler actually by default includes both open id and profile as scopes to send off to octa or to every openid provider since this is a generic open id connect handler by default there are two scopes that are automatically added and we don't need to specifically add those and send those but in this particular case but if you do want to add other scopes for instance with octa you can ask for a refresh token by sending the scope of offline access i believe

by adding that scope they will also return a refresh token to you as well as the id token and an access token

we've just scratched the surface of what is possible with a hosted identity provider like octa when would you use a hosted provider like octa or azure directory well you can use it in many ways for example if you were building a system for a small company and they wanted to manage their accounts for hr purposes like when a new person is hired or another person leaves employment hosted solutions are great you don't have to build it yourself but you still have control over the attributes of the user's identity once you have your environment set up you can use tools like terraform to make it easy to transform your development environment to look the same for other developer accounts and even preload it with test users etc maybe your organization already has a terraform script to accomplish just this for the next section of this tutorial i wanted to move away from the hosted identity concept and build out the application to really support social identity providers like google facebook twitter etc and demonstrate how we can allow new users to register with our application so we can remember them on subsequent visits i've made a couple of changes i've taken out the hard-coded authority client id secret etc and i've moved those to my secrets.json file to keep them protected i did this so i could add it to source control on github to make the project available to you you will find the project on github at github.com mobiletonester authen

next i added a sqlite database file locally in the project to simulate a hosted database like sql azure without having to set one up it makes it easier for you to use when you download the project from github to learn more about how i added the sqlite database using entity framework code first you can click the link to go watch my codehack moment video showing the setup you'll notice we have a data folder and inside of our data folder we have an app db app users an authdb context and some migration code for the snapshot and the initial now that we have an app database created for our application i want to add some services so that we can write common functions to interact with it i'm going to add a new folder so i can place all my services in here and let's go ahead and add a class i'm going to call this the user service

okay and let's go ahead and create a constructor for it and let's add a private read only i'm going to need access to the database context and i'll just call it context since we only have one

and we'll need to get that handed to us through the dependency injection system

and then we'll assign

the variable the incoming that looks pretty good so the first thing we want to do is be able to look up a user by either their id or by their external provider so we could do something like this and maybe we'll make these internal so internal app user get user by external provider

and then pass in the name of the provider be it octa or google and then i'm going to have the name identifier passed in or their unique id from each of those providers then i'll look in our context oops

and in our app users and i'm going to look for where an app user has a provider equal to the incoming provider that we're asking about and then we'll have where the name identifier equals the name identifier and then we'll take the first or default and then we'll return app user so we could do that all in one line but i may want to add some other things later so keep it separate for now and then within our system we have a our own surrogate key or an id that's just an integer base from one to infinity well infinity is a little high one to some really large number and i may want to look up the user by that internal id that we assign them once they get added into our system and for this i'm just going to use a find so this will work for external providers like google facebook octa twitter etc get user by id will let us look up users that are internal that have been already registered or joined our system and then finally if someone created their own custom username and password with our local system we need a way to do the validate event if you look in our home controller right now in our validate event we're literally just doing a hard code and we're only looking for one user named bob so we want to change this to be driven from the database instead so for this i'm going to return a boolean i'm going to call it a try validate user so we're going to create a try pattern that returns a boolean and well just follow me along here this will make sense so we'll pass the username and password in and then we'll pass back a list of claims our out variable is going to be just a list of claims if we find a match so the first thing i'm going to do is initialize claims oops first thing i'm going to do is initialize claims next thing i'm going to do is try to find the a matching app user there's a couple different ways we could do this but i think this one will make sense and i think you'll be able to follow along here and follow my crazy train of thought so i'm adding two where clauses to this link statement basically we're going to look in our list of users and if we have a list if we have a user that matches the username and password that we're looking for we'll grab it as first or default now if there's no match for instance if the user puts the wrong username and password combination in there app user should come back no so if app user is null then we're just going to return false and we're not going to do anything else but if we do get a match that means they put the right username and password combination in and we have an app user back then we're going to add a bunch of claims and this could take a minute so maybe i'll just paste these in so we're adding a name identifier a username a claim for given name surname email mobile phone

okay and then i also want to add this section in here to loop through a roll list and add a role claim for each role in the user's role list now let's look at this real quickly the role list is a gap method that returns this from the string roles which is a common delimited list of string values for roles or names and we split those and turn them into a list of strings instead from the database we can add a comma delimited list of roles pull them back out and split those into a list of string and what that allows us to do is in here to loop through each one of those and add them as an individual claim role type for the user then finally we need to return true now that we have our try validate user in the user service ready to go we need to go connect that in the home controller if we look in the home controller under the validate event we still have this hard-coded username and password check and we really want to get away from that and use our new service method to look it up in the database instead before we can do that though we need to add a reference to the service

and we're going to use dependency injection to put this in

i don't have time to go into details about the dependency injection system and how it works but we will just use it and maybe in a future video we'll do a deep dive into that okay so now i've connected the user service and i can use it down below

so user service and we'll call our new tri validate user and what does it need use a username and a password and it's if it's successful if it's true then i'm going to get back a list of claim and we'll call that claims so out of this we can now eliminate all of this stuff in here because that's all encapsulated in our validate but we will need this other part

so we'll paste that up in there and get rid of this whole big block and we can have an ounce block and in the else block we can return our error letting the user know that they something didn't go right with the username or password okay so this should work let's take a look and see if that's the case

and this is the error we get unable to resolve service for user service while attempting to activate the home controller and it says right here dependency injection you didn't set this up and it's right we didn't so let's go do that if we go to the startup we go to the configure services section where do we want to put this about right here and i'm going to add this as scoped and we'll go user service

okay so now that we've registered it with the di system it should be able to successfully hand that to us when we launch the home controller let's try it again

bob and pizza in and voila it took at this time and this time it was driven from the database and the user comes back as bob tester and as a we have a phone number here for mobile phone email address and we've picked up his role of admin and this was performed through the authentication scheme called cookies so this looks great we're doing fantastic we need to turn our attention to our social providers if we go into the startup if we examine google we have a hard-coded entry to make bob an admin instead we want to get that from the database so let's remove our hard-coded entry

we won't be needing these events so let's get rid of it

because we'll want to use the on signing in event instead we need to get the user's name identifier his name id from the claims identity as it's constructed so far and pairing that with the auth scheme used check the database to see if we have a user entry this means we will need access to the user service from the dependency injection system

to get it we can use a method on the context so i'm going to have a var user service variable call the context http context

request services get required service and it wants a type so we'll say type of user service and we'll cast this as a user service

we also need to grab the name identifier from the claims identity

i probably should put some checks in here if user service not equal to null

name identifier it's not equal to null

okay and then what we want to do is say var let's see if we can get an app user from the database using our user service and using our method get user by external provider and for the provider we'll use the scheme and the name identifier and we'll see if we get a value back

so the scheme and the value

the name identifier and see if we get an app user back i say we put a breakpoint here and we run it to this point and see what we get

actually let's put a breakpoint here and we'll step through start with google

okay now we'll come back here and let's see we get we have this object for scheme that shows that the value is google

we get our claims identity and we should get access to the user service which we do

name identifier that's the name identifier for bob so we have both the user service and the name identifier if we call to the database our app user is null because we don't have an entry for him in the database but if we did it would have found him and that would be good so we're on our way let's create an add new user method in our user service so we can add them to our database the first time they log in with our system let's go to user service

and continue on internal and once we add them we'll return them as an app user

call it add new user and we will need

first we'll need to know the provider and then let's bring in all of their claims

so we'll create an app user

and now the mundane effort of mapping all the properties so let's start with name identifier we have to go to our list of claims

the first or default let you see this

time and then we assign it

this could get tedious i want to write an extension method to make this a little bit better now an extension method needs to be in a public static class and i'll just call this extensions and put it here locally not best practice but this should demonstrate what i'm trying to accomplish and we'll public static string get claim from this is this secret keyword

and the name of the property we'll just say return claims dot first or default

type equals equals name question mark dot value

so now if i come up here and i say app user dot username for instance i can say claims dot get claim and the name is going to be username

and that's much shorter than what this is up here so let's redo this one

and we can get rid of this

that's a little cleaner and i'm going to go ahead and do the rest of these

okay so i've finished typing these out there's one more property i need to assign and i'm going to put that at the top because we want to make sure we pass the provider in and assign that

okay now that we have all of our properties filled up we need to go to the context dot app users and then we're going to just do an add this new app user and that ad will return an entity so let's grab the entity that comes back

then we'll need to call context dot save changes and then we want to return the entity dot entity all right so that should be our add new user we'll at will construct the user fill his properties out add him to the app users context save those changes which will commit them to the database and then we will get back a new version of the entity that will have the user id surrogate value populated because we're not sending one in the database will automatically increment that and assign that and we'll have that back and we'll return it and use it

let's go back to startup and let's connect this

so if our app user is null let's call our user service i'm going to add a new user and let's pass in our provider which is located in our scheme dot value and then our claims identity dot claims we'll pass those in and what we'll get out of here is a new app user so let's just reassign app user equal to that

it's an innumerable so let's just make it

list so the last thing we want to do in here is

grab the roll list from the user and i recognize that if we're just adding a new user he won't have any roles added to his account we're going to take the claims identity as it's built to this point and we're going to use the add claim method to add a new claim claim types.roll

and add r to that list so then basically what this is saying is we'll go in here and try to find the user in the database if he exists then we'll just drop down to this section and pull out his claims specifically pull out his roles look through those rules and add a claim for his role if it's a new if the user doesn't exist we'll add him first to the database he won't have any roles so we won't have anything to loop over but the next time he logs in if someone's gone in and assigned him some roles then we'll have that data and before we run that let's take a look in our database so i'm using sqlite browser to look at this apparently there's a new version so if we look at our app users table we have one app user so far this is the bob pizza user and the provider is cookies and the user id is one he doesn't have a name identifier because he's a local account and not an external account so when we run this we'll watch and see what happens to our database

click secure this time we'll log in with google again

and we had an error

so we'll do roles split to list and if it's null then we'll return a new list of string

it didn't account for that situation

he should get a permission denied because he doesn't have any roles so if we go back to our database and if i hit refresh you'll see we now have a second provider in here and it's google with his name identifier and his name first name is bob last name is tester and he has no roles so if we were to go in here and give him an admin role

and then we have to write those changes to the database come back to our application log out

and log back in

and it lets us through so if we were to repeat the process with octa

he should get an access denied go back to our database hit refresh now we have an entry for octa it didn't pick up his first name and last name for some reason we'll have to take a look at that and see why octa is not sending this but i'll write those changes we'll come back we'll have to log out log back in log in with octa

so this time we got the app user back but for some reason the app user has no first name or last name so let's look at the claims that are coming in for him

they have a name field for bob coder and they don't specify a first name and last name so they only support a name field which kind of stinks let's go back and fix that under add new user first name last name if app user dot first name if string dot is null or empty first name then let's go to this claim actually so let's just do it this way so then we'll just say app user.first name equals this

so we'll just shove it in the first name field for now we could do more to clean that up but i think that's good enough so what i'm going to do then is let's delete this record write those changes come back in we'll run it again

log in with octa

and if we go back to our database hit refresh now we have an entry from octa and we just shoved bob coder into the first name field now there are other ways we could probably do this with octa we could go and get a separate first name and last name by going and tweaking his token as we showed earlier but for now that's good i'm going to type in admin and then i'm going to write those changes now if i log out and i log back in with octa and voila we are in and he has a couple of claims he's in the every one role and now he has the admin role because i added it 

We have covered a lot in this video. I hope you found it useful.

* We connected to a cloud hosted Identity as a Service Provider in Okta. 
* We learned about how our Okta Tenant is ours to manage and configure with users and groups. 
* We learned how to connect multiple providers as challenge schemes and give the user a choice of different social login providers.
* We learned how to connect to a local SQLite database (which you should probably replace with something like SQLAzure or CosmosDB when you are ready to go to production) to add our users as they login to a social identity provider.

There is still much to do to make our application fully functional, but we have covered the basic mechanics. If you are planning to provide an in-application login, you should be sure to one-way encrypt the user's password to protect it.

I might add a Part 4 video if people are interested in how to host and protect a single page application in ASP.NET Core, such as an Angular or React application or even a Blazor app. Leave me a comment if you are interested.

Be sure to check out the sample code in Github and visit my website at Mobiletonster.com. You can find me on Twitter @mobiletonster and feel free to leave me comments or questions at the website, on twitter or on YouTube.

Once again, thanks for watching!