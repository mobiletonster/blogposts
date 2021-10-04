<!-- # ASP.NET Core 5.0 - Authentication/Authorization -->

<iframe width="560" height="315" class="video-frame" src="https://www.youtube.com/embed/BWa7Mu-oMHk" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

## Introduction

Authentication and authorization is used to prevent unidentified and unauthorized users from gaining access to systems they don't have permission to use. In other words, keep the bad people out let the good people in. People often confuse authentication and authorization. The http standards didn't help things either (401 unauthorized, 403 forbidden), but i don't want to discuss that. Instead let's watch two short clips from popular movies that illustrate the differences between authentication and authorization.

### Authentication Clip
First let's see an example of authentication using multi-factors to validate the identity of a user. 

<iframe width="560" height="315" class="video-frame" src="https://www.youtube.com/embed/BWa7Mu-oMHk?start=63&end=90" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Authorization Clip
In this next clip we will see that although the user is a valid user and should have pretty broad authority that authority is not recognized by the gatekeeper and the user is denied passage or is not authorized to pass.

<iframe width="560" height="315" class="video-frame" src="https://www.youtube.com/embed/BWa7Mu-oMHk?start=106&end=148" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

These clips demonstrate that authentication is used for identification and validation of a user and authorization is used for permissions or the authority of a user.

## Time for code
To begin with let's startup Visual Studio 2019.
![Image 1](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/background.jpg "background image with purple gradient")

Once Visual Studio 2019 loads, click `create new project` and we're going to be creating an ASP.NET Core web application. Give it a name of `Authn` as the name of the solution and application.  I put it in a folder called `delete me` so i can remember to get rid of this when i'm done, but you can put it wherever you like.

Select ASP.NET Core 5.0 and create an ASP.NET Core web application MVC (model view controller). Enable Razor runtime compilation by checking the box.

We now have a blank MVC application in ASP.NET Core. To run this application I prefer to switch from `IIS Express` to use the built-in kestrel web server. To do that switch to the name of the project `Authn` and then hit play (F5 shortcut key).

Now that the application is launched we can see that it has a a basic page with a couple of links in the navigation bar menu. If you click on privacy it will take you to a page that has the privacy policy on it.
 okay the next thing we want to do is take a look at our project and see what was built if we look inside the project here we have a few folders and some some files let's take a look inside the controllers folder in the controllers folder we have a home controller if we open that up we'll see there's some code and the code that is in here is pretty simple we have an action for the index which returns the home view because the name is home controller and then we have a privacy view as well and these are the various routes that get clicked on when you click on the menu it navigates to these routes and it returns the appropriate view for these so let's create another route and we will call this route um secured route

and we'll just return a view so we don't have a view created for this yet one way to create a view is just to right click on this and say add view

and we'll just create a an empty one

and we will call the name of our view secured

and just so that we know we have something in here to display we'll just say this is a secured how about we just say this is top secret okay now i'm going to go into another folder in the views called shared and in here we have a layout and this is just some basic nbc razer setup and i won't get into that too much but i'm just going to copy one of these nav links here

and create another one and we'll call this one secured and the policy will be or the name of it is secured there's no policy okay so let's run this and see what we have

so now we have one one new link which i added secured and if i click that it takes me to this route home secured and you can see our message this is top secret all right well wouldn't be much of a secret if you can get to that that easily so let's go back to the home controller and we're going to mark this with an attribute called authorize

and it requires a namespace so i just hit control period to bring that in so now this is will be protected by this authorized attribute if we run this again and i try to click on that secured link it should not let us in so let's take a look and see what happens

okay well it didn't let us in but it also complained and said there was an error here and the error is that no authentication scheme was specified and there was no default challenge scheme found now let's take a moment and talk about a few concepts you will often hear the web described as stateless state is a snapshot in time of data regarding a transaction or a process the web being stateless isn't exactly true but what people mean when they say the web is stateless is that the state is not being stored on the server in this instance by state we mean that the server doesn't remember who you are between requests think of the server as a forgetful doorman whose memory only lasts a few milliseconds in most cases every time you make a request to the server you must authenticate and prove you are who you say you are one way of proving who you are is by providing a secret that only you and the doorman know these are known as credentials often in the form of username and password combination since providing credentials on each request would be cumbersome after you log in you are usually given a ticket to represent you instead then you simply present the ticket each time and the server knows who you are the tickets are temporary and expire after a certain amount of time after which you must revalidate your identity by providing credentials once again in the microsoft world authentication is subdivided into separate processes authentication is really the process of checking a ticket you have already been given this happens on each request in the case of cookie authentication scheme the ticket it is looking for is a cookie the process of validating a user for instance by prompting for credentials is the challenge scheme we will look at other challenge schemes in a moment for now a simple username and password prompt will suffice for our challenge scheme

returning to our project we need to fix the problem that we ran into where the application was unable to find any authentication scheme is registered let's go into the startup now in the startup there are two main methods configure services and configure so the configure section has houses the pipeline for processing a request and inside of here is what we call middleware we'll talk about middleware more in another video the other method is configure services this is part of microsoft's built-in dependency injection system in here we'll register various services and pieces of code with the configure services method and it will add it to a collection known as the iservice collection and this is where we need to set up our authentication so we'll do that now the way we do that is by saying add authentication okay so if we just did that much that won't really get us too far one things that we need to do is we need to add a particular scheme authentication without a scheme won't be very helpful the scheme we want to use is called ad cookie or cookie authentication all right so now that we have cookie authentication set up let's try and run this and see what happens now now if i click on secured what no authentication scheme was specified and there was no default challenge scheme found hmm so even though we added those something's not quite right let's see if we can fix that well if we look at the add authentication overload there is an overload see there's an overload where they want us to pass in the name of the default scheme it's a string so let's see if i can do this cookie authentic location defaults

i get it right using that was close and the authentication scheme because we're using cookie authentication although we've chained it on here we really need to explicitly tell the authentication system that this is our default authentication mechanism so cookie authentication defaults is part of the asp.net core authentication.cookies and if we were to hover over it what we would see is it's a constant remember it wanted a string so the constant and the name for it is just generically cookies it's easy enough to just use that so let's see what happens

okay we got a whole new error now this time we got a 404 error saying page not found and if we look closely at the url you will see that the application tried to route us to account slash login so that's progress the only problem is we don't have a route for account slash login we'll need to fix that next we need a route for our login page before we add that though let's go take a look inside the startup

i want to take a deeper look at this ad cookies i'm going to format this a little bit differently move this down here and i'm going to show you how to configure some options or cookie authentication using a lambda expression i can now access a number of different options for the cookies the first one that comes up on the list is login path now i'm going to specify a particular route or login path or the cookie authentication by default if i don't put anything in here it's going to come up to the account login page expecting there to be an account controller and a login action in that controller since i don't want to really add a whole nother controller just for the login page i'm going to just keep the route at slash login at the root and then we're going to go and put that in the home controller instead so back in the home controller i'm going to add a public eye action result log in and i'm grateful that it corrects me and we'll just return a view for the login page now if i leave it like this we're going to have a little bit of a problem here because i told the cookie authentication scheme to redirect to the slash login not home slash login right now if i leave it like this by default it will look at it will only bind this to the home slash login route to fix that i'm going to be explicit and i'm going to put an attribute over that http get and login as the route now it will map this to that particular route we don't have a view so let's add a view

another empty one and we'll simply call it login.csht and to make this look like a login page i'm just going to put a h1 in there let's run this and see what happens now when i click on secured it routes us instead of to account slash login you see it went to log in it has a return url on there and now we're actually on the login page so so far so good so let's build a login page back in our cshtml page i'm just going to paste in some code here so we don't have to type it out it isn't it's essentially a form the action is going to be slash login and the method is going to be post we have an input field for a username a password and then a submit button pretty simple login page we need to add another route in our home controller to accept this login post method we already have a login http get method so let's create an http post method to handle when the user logs

in i'll call this validate i'm going to be expecting that we will receive a username and a password

and for now if it's successful i'm just going to return it in ok in our simple example the validate method is where we want to check to see if the username and password is a match to our system normally we would attach this to some sort of a database or some sort of a lookup to look up that particular username and that particular user's password to see if it's a match in this case we don't have any such things so i'm going to make it just very simple i'm going to say if the user's name is bob and the password that they pass in is pizza

then we're gonna say that's okay if that isn't a match and uh better fix that make sure that's a double equals there if it's not a match then i'm going to say

bad request for now and we'll take a look at that we'll make some changes here but let's run this and see what happens

now i have a login page let's say that fred was my username and ankle was my password if i put that in i get a 400 error http error 400 which is what we told it to do send back a bad result but if i put bob and i put pizza now i get an ok and lastpass wants to store that our basic behavior is working but that's not exactly the behavior we want to present to the user so we're going to adjust this just a little bit what we really want to do is when the user has made it past the challenge successfully we want to return them to where they were originally heading if you recall when we went to the login page originally we noticed in the url there was this parameter return url equals percent to f home percent to f secured in other words home slash secured that was the route they were originally intended to go but we've redirected them to the login page momentarily once they've logged in logged in we really want to return them back to that url so we need to do a couple of things first on our get for our login page we need to capture that return url i'm going to modify this method by capturing the return url and you'll notice the casing doesn't necessarily match the binding engine in the model view controller will automatically match return url with a capital r with this lowercase return url it knows it's the same thing we need to temporarily stuff

return url into a view data bag and i won't go into a lot of detail about this necessarily but what that allows me to do is pass this information into the view

so if we go to the login cs html view what we really want to do is capture that return url that was passed in and put it inside of this page and i'll show you here what i mean to get access to anything in the view data bag it's pretty simple you just type something like this bar return url equals view data of the key that's the key that's that we put that in return url as a string and then to use it what i want to do is i want to modify this host action to go back to the slash login to again forward the return url

so that's the parameter and to access that return url i put the at symbol return url this is all basic razor syntax which we won't get into too deeply here but suffice it to say this should pass that on now there's one more thing i need to do here the return url is going to have slashes in it and by convention in the web we want to escape those or encode those urls

there i've used the system.net web utility url encoder to encode that return url

okay now if we go back to the home controller the post login method will also need to grab that incoming return url

then when we're done we will now if the user puts in bob and peace pizza or they pass the challenge we will return them we will redirect them back to where they originally were coming from

all right let's see if we can follow this through so far

bob pizza okay so that worked but what happened we're back at the login page again let's open up our developer tools and see if we can follow along and see what happened i'm going to try it again bob pizza that's interesting we follow what happened it redirected us to the login return url page then it went to secured because we told it that was where we were originally coming from and it came back to this login return url page so for whatever reason even though we put in our our credentials and we've said everything's okay it doesn't seem to be really logging us in because by the time we go back to the secured route again it thinks we're not logged in anymore we have our redirect to login working but we don't actually have the mechanism to sign us in working that's because there's a couple more things we need to do back in our login method we actually need to sign the user into the authentication scheme

and the way that we do that is there's an extension method called sign in async that lives on the http context so since it's async we're going to have to make a few wholesale changes here to make this all work well

we'll mark this method as async return a task by action result and then we'll call this sign in async

but it's not happy what's it unhappy about it says there's no over led load that takes zero arguments it requires us to pass in the name of the scheme or one of the other overloads would be a claims principle so we need to create a claims principle

okay now that we have a claims principle let's see if we pass that in

and make it happy okay signing async's happy but now claims principle is unhappy

claims principle is in a namespace system security claims so we'll resolve that now is it happy yeah it's happy but it's not going to work i'm going to save us some time and look at the third overload this overload is expecting an identity to be passed in i'm going to pass in an identity and i happen to know that it's going to be a claims identity is what it wants so i'm going to create a claims identity yes claims identity has 10 overloads but the ones that the one that we want to use it wants a list of claims and then it wants to know the scheme that we're using

so i could just put the string name in cookies and that would work but it's a good idea to use constants

if you recall we use this in startup and the default is the authentication scheme again if we hover over that we'll see that it is a constant equal to cookies all right so the last thing we need is we need some claims

and claims are a list

and i'm going to do it this way

i'm going to add a new claim and claim accepts a value i'm going to do username or a type and a value so the type is a string and it's it can be anything we want it's a key value pair and we can create that now there is a few standard key pair types and we can get claim types dot name identifier as one and i'm going to put username in there as well okay i missed a parenthesis up here so we're happy now all right so there's a lot going on in here and we probably need to talk about it what are claims claims are properties that describe a user they're the things that the user is or is not not necessarily the things that the user can or cannot do in the system so for instance possible claims might be the user's first name last name their email address maybe their birth date or their age those are all properties that describe a user looking at the code that we've added in here we created a list of claims we populated it with two values so far a claim with a key value of username a claim with the name identifier which if you look closely at that i'm not sure you can see that but it's a long constant that is a standard for the name identifier that would be like a user's id or username and then we created something called the claims identity which is the identity of the user with their claims attached to them in the scheme that that is bound to from there we created a claims principle which internally is something like an authentication ticket and we pass that into the context sign in async so what's going to happen during sign in is inside of this magic black box known as the cookie authentication scheme that information is going to be taken in that principle that we passed in a ticket is going to be made out of that and that ticket is going to be stored inside of a cookie and we'll see this happen here in a moment then on each subsequent request from the user back to the server that cookie will go will write for free on that request the server will receive it recognize the cookie as the cookie it created open it up to check it for a few things make sure it hasn't expired make sure it's valid and hasn't been tampered with and then it'll re-extract that information so it can remember who you are on every single request there's one more thing we need to do we need to go to the startup and we need to add a piece of configuration to the middleware

now we could have run this but we would have been frustrated because it wouldn't have worked there's one more thing we need to add in front of use authorization we need to use authentication and this one's important if we don't do this the authentication middleware won't actually execute and the cookie won't actually get created now with everything in place it's time to try it out let's give it a shot

i'm going to click on secured i'm going to put in the correct username and password bob and pizza and ta-da we're into the top secret page now let us through this time let's take a peek into the developer tools and we'll look under the application tab and you'll see here under storage there's cookies and the cookie that's bound to this particular domain name localhost 5001 is this generic name.aspnetcore.cookies and if we look inside the cookie here we can see the contents although the contents are not really useful to us that's because the values in here have been encrypted if we could decrypt them what we would see are the claims for the user that we've added all is well so far but what happens if a user puts the wrong username and password combination in right now we just throw up an ugly error page but let's make this a little bit better i'm going to add this return url in here

and you may be wondering why and that's because i'm going to change this from return bad request to return view and the view i'm going to return is the login view now i have to specify that by name because by default it's going to look for a view that matches the name of the method which would be validate but what i really want to do is if the user puts in a bad username and password combination i want to return them to the login view and then i want i need to pass the return url in because as you can see up in this method just before i call the view it has to set that return url because inside the view it's expecting that so i have to satisfy that condition now if i go to login view oh there's one oh so the other thing i need to do is i need to let the user know why we came back to the login page to do that i'm going to use another

data bag if you will or it's a dictionary of objects and i'm going to use the word error and i'm going to say

error username or password is invalid

okay now we'll go to the view and we'll get access to that variable

and remember it's in temp data this time not view data

okay then what we'll do is i'm going to do a if not string dot is no or empty error

i'm going to output an h2 and it's going to be at error so it's going to just write the message there and i'm going to make this a class of alert danger and add a little bit of padding and let's do padding i don't know 20 pixels and should we run that and see how that works

this time i'm going to put the incorrect credentials in

and it came back and it says error username or password is invalid

and it's not going to be happy until i give it correct credentials okay so now we have that taken care of

next i think what we want to do is work on log out we want to be able to sign the user out so let's take a look at that now so i think we go to the home controller first and let's create a log out method okay so let's let's create a public i action result and we'll call it log out

and this may not make sense but we should annotate that with an authorized attribute and the reason that is is you need to be logged in in order to log out

so to log out all we need to do in fact we'll need to change this but

sign out async

so that means this needs to be an async

task result so we'll need to sign out and then once we're signed out let's return them

just to the home page all right so i think that should take care of it for sign out now there's some overloads let's take a peek and see what's in those overloads there's nothing there is a specific authentication properties i could pass the name of the scheme in and properties so

let's try it without and see what happens

that would be nice if we had a way to log out i'm back in the layout cs html page and i want to have a log out button in fact i'm just going to copy this

and the log out let's go back here it's at home slash log out okay if i left that on there though this is always going to be visible even when you're not logged in so i really only want this to be visible when i'm logged in well we can get access to something called a user object and we can look at the identity and ask if it's authenticated if the user is authenticated then we will display a log out button so that leads me to the next thing let's go to the secured page and let's output the information we know about the user so i'm going to say welcome at and we'll say user dot identity dot name but we don't have a name we never populated it

but that's okay we'll just populate it like that and then let's do a let's do a bullet list

now you'll notice that there's a property off of user called claims which is a list of claims what i want to do is loop through each one of those and output them so we can see what we know about the user so a claim is a type key value pair so i'm just going to say c dot type

do a colon and then do a c dot value so that we can see the name of that the claim and the actual value of the claim and the list item let's run this and see

okay welcome and notice it doesn't output his name because we don't have a key value pair for that we have username bob this name identifier which is bob but we don't have his name so let's go add that be nice if he had a name wouldn't it so normally this would be a look up from a database and we might have more information about him from the database that we would populate into our claims i'm just going to do it by hand and i'm going to use the claim types dot name and the name i'm going to put his full name bob edward jones

now if we rerun this

okay so it came up click secured bob pizza welcome bob edward jones username bob name identifier is bob the name the claims name is bob edward jones so we added a claim here okay that's really good

oh and while we're there so we're logged in let's pull up the developer tools and let's look at our cookies what happens when i click log out

it erases the cookie and the user is no longer logged in when i click on secured it wants to prompt me again all right

so we have login and logout working we're able to store the cookie for the session uh we're able to identify the user add some claims to him put him in and display his claims back out on our secured page so we've made good progress we want to look at one more thing in startup

i want to take a look at events

so you remember when we talked about the black box that is the cookie authentication scheme inside of there there's a lot of things that are happening when you are in the controller and you send that sign in async method and pass the claims principle in there inside of the engine there's a lot of things happening we saw the result of it it created a cookie it put created a ticket and put those claims inside of the ticket and embedded that inside of the cookie and encrypted that well what if we wanted to jump in midway through some of that process in the black box how do we do that the way we would do that is through events the cookie authentication handler exposes events for us during that through that process that would allow us to jump in and do some other things and we're going to take a look at that now so i've added some code here options that events equals new cookie authentication events and now if i just hit the space bar you'll see that i have access to a number of different events in here on redirected to login on redirected to logout unsigned in on signing in on signing out on validate principle you know what are all these different things i think some of them are obvious on redirect to login and on redirect to log out will happen during those those times but let's look at a few of these others let's look at on signing in and the way that this works

and then let's also look at on where is it unsigned in

and then let's look at on validate principle

okay so now

it's saying it's not happy with me why because these are async

now it's not happy because we're not using any async methods in here so for now i'm just going to say await task dot completed task and we'll put that in each one of those and that'll satisfy it so the compiler is happy and then what i want to do is i want to put some break points in here and then we'll run it and we'll watch when these break points get hit

okay so i'm signing in right now and it's hit the signing in event if i look at this there's a context and in the context there's a bunch of different things notice there's a principle and inside the principle there's an identity or a bunch of identities and if i look inside the identity there are three claims and the authentication type is cookie and it says there's three claims username bob name identifier bob and name bob edward jones so at this point in time i have access to the claims principle as it's been passed in now you say well we had access to that already in the controller and that's true but you'll see where this might make a difference here in a minute if i let it continue on now it's to this on signed in context it's completed that process and again if i look in here you'll see that there's a principle and there's a bunch of claims if i look at the results view you'll see the same the same three claims again and then when we hit validate principle we'll go back into the context here and you'll see again the principles there and you're wondering what hits all this stuff here it doesn't make any sense well these all fire at different times okay so now that we've hit the page i'm going to go back to the authen page or to the main home page and notice that validate principle is firing again okay now if i go back to the secured page this on validate principle is once again it's firing so that means on every single authentication request there is an event that fires that says it's validating that principle and remember we created a ticket and we embedded it in a cookie so each time the request is made that cookie goes back then the engine has to validate that that cookie is still active and not expired and that it's legitimate and hasn't been tampered with and it's during that time that this on validate principle event is fired and it gives us a chance to do more we could peek in there at the same time and look at some other factors if we wanted and perform an action

okay so those are events and as you saw there's more events that happen but notice the order in which they fired on signing in fires first after that's completed unsigned in com fires and then finally the unvalidate principle fires on every single request so now i want to switch gears just a little bit and let's talk about authorization so far we've used the authorized attribute in the home controller on the secured page and we've simply used authorize which basically means that the only authority that you need to have to use this page is you must be a user that is logged in but what if we wanted to assign different permissions to different users how would we do that well let's take a look

if i go into this authorized attribute and i put a parenthesis in here you'll notice that there's some options i could set a policy or i can set some rules and for now i want to just use roles and it's a string list comma separated and i want to say if the user who's logging in is an admin or they have the admin role then let them go here if they don't then send them to a forbidden page now this is going to be a little bit different because if they're authenticated then they won't get a 401 error and unauthenticated or they won't be redirected to the login page because they're already logged in but if they don't have the the proper permissions or rights they'll get redirected to a different era so let's try this right now with bob we know bob's not an admin because we really haven't given him those permissions or those rights we'll let those break points continue okay so bob's logged in

if i click secure what happens it redirects us to an account access denied page so bob's authenticated he's logged in but he doesn't have rights to go to that page anymore because we restricted it to only administrators so let's do a couple of things here so first off i don't necessarily love that particular path so there's another path in here an access denied path and you can and we'll call this denied so we'll go back to our home controller and let's create a page i don't know where we would put it let's create a page and we'll call it the public i action result denied page and because i put it at the root

and we'll return a view

and let's add a view

and i'll just say h1

permission denied

and then we'll just put a small paragraph that says here you do not have rights to the page you were attempting to access okay let's try that again this time we should get a nice pretty page instead of a 404 not found

and we've landed on the permission denied page you do not have rights to the page you're attempting to access well bob is our only user so likely he's the administrator let's see if we can give him rights well one thing we could do is we could go back to the home controller and right here we could just simply add a claim

and we could make him an admin and that would be all we needed to do to give him that particular role however i want to show a different way to do this i don't want to necessarily do this here and it'll make more sense when you're using other types of schemes like a token bearer scheme like a jwt token bearer scheme or some other schemes where you may not have access to this information or when you're using an outside authentication scheme challenge scheme such as open id or oauth

so i'm going to move this for a moment and let's go back to startup and let's try to put this in the on signing in event so the first thing i want to do is get the principle that has been created to this point

and i want to inspect it if principal has claim

and c dot type equals equals claim types.name identifier so we checked first to see if he has that claim then we also check if the principal dot claims that first or default of type and this is a little messy i know um

type equals equals claim types.name identifier

dot value equals equals bob in other words oh we're trying to figure out here hey is this bob okay so if it's bob then what we want to do is get his current claims identity

from this principle

identity an identity is a i identity and i have to cast this then i can take the claims identity and i can add a new claim to it going to add a role and add his role of admin so what we did is we're in the middle of the process of signing on and during this moment in time the cookie authentication scheme handler is allowing me to jump in here and make modifications before it finishes making his authentication ticket so now's the time to get in here and add any additional claims and that's what i'm doing i'm grabbing his identity out casting it as a claims identity and then adding a claim of role admin to it so that he'll be able to get through in the future all right i think this should work let's give it a try click on secured we're going to log in as bob and when we do that notice that we've we've already started the login process if we were to go back here and set a breakpoint we would be right here at this point in time the sign in async is now firing this code in here this on signing in so we're going to get an access to the principal check to see if he has a claim of name identifier a username basically or an id if he does and his name identifier value is equal to bob which it is then we'll grab his current claims identity and if we were to look at it we'll see that it has three claims and we're going to add a fourth claim to it now if we look at the claims identity there are four claims the fourth one being the admin role so now if we continue on now it's unsigned in it's validating the principal and it led us through this time it didn't redirect us to the access denied page and you'll notice he has these four claims available now wow we have covered a lot in this video we've created our own login screen and handled the credentials ourselves we let the cookie authentication handler create a cookie for us to represent the user in our next video we will look at using a remote authentication handler such as oauth or openid connect and discuss why it is a good idea to delegate login responsibilities to a dedicated identity provider for now thanks for watching

you