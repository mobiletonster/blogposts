<!-- # ASP.NET Core 6.0 - Authentication/Authorization -->

## Introduction

Authentication and authorization is used to prevent unidentified and unauthorized users from gaining access to systems they don't have permission to use. In other words, keep the bad people out let the good people in. People often confuse authentication and authorization. The http standards didn't help things either (401 unauthorized, 403 forbidden), but i don't want to discuss that. Instead let's watch two short clips from popular movies that illustrate the differences between authentication and authorization.

hello everybody and welcome to this video dot net 6 is out asp.net core is shipped and there's been quite a few changes that have left a lot of people confused like who moved my cheese where's startup.cs this video i hope we'll take a dive into that and see where it went things haven't fundamentally changed in the middleware for asp.net core but some of the project structure has changed and where you put things has changed a little bit so let's deep dive into that and figure out what's going on so let's get started all right so the first thing i'm going to do is create a new project a console project

we'll call it old to new

and i'm going to have it be a.net core 3.1 and we're going to upgrade it to a 6.0 and see what the differences are if you've been around.net for a while you'll recognize this old project structure we're going to go ahead and change this over to a.net 6 project piece by piece and explain what has happened so some of the changes they made in net six was to simplify and remove cruft from our application one of the first features they introduced was something called file scope namespaces i mean if you've been around.net for a while seriously how often do we put more than one namespace in a file like never so you can remove these curly braces put a semicolon here and have filescope namespaces now it's going to complain at me because this is a 3.1 project so before we go too far maybe we had to edit our project file and change this from a net core 3 app to a net 6.0 app

okay if we do that what changes is this namespace now is happy and the other thing that we can get rid of is a using statement if we add implicit using so let's go back to the edit project file and add implicit usings type enable if we enable implicit using statements most of the common using statements that exist will be included in the sdk by default and you no longer need to include them in your file for everything to work the next feature they added was something called top level programs they wanted to remove the craft that was basically unnecessary but present in every console application or asp.net core application for that matter so they said hey we can get rid of this

and we don't really need this namespace anymore and so what we're left with is just the stuff inside and this actually still works so if we run this you'll see hello world okay very cool

currently this is just a plain console application but i want to turn it into a currently this is just a console application but i want to turn it into an asp.net core application but before we do that let's take a look under this dependencies and frameworks currently you see microsoft.netcore.app that is the sdk that includes all the required packages to create a console application so watch what happens when i go back to the cs proj file and i change the sdk to a web project

notice what happened over here the framework brought in microsoft.asp.net

as the framework besides the framework bringing in all the necessary packages for an asp.net core app the other thing that changed was the global using statements now let's get rid of this console

and let's type this

we are now just constructing a web application where does this web application come from it comes from microsoft.asp.netcore.builder.web application

let's type var app equals builder dot build

whoops i can type typing's hard and then we can do an app.run now if we do this it'll be kind of boring because there are no endpoints so under the minimal api model which is what this supports out of the box

now we have a fully functional minimal api web application in asp.net core wants us to reload the projects we should have done that before but let's run this as an old to new

now when we launch the default path it returns hello world so we have a fully functioning web application minimal apis are a great way to quickly build a web api project the middleware flow is similar to that of the full web apn mvc projects and shares much of the same implementations in previous asp.net core projects you were given a startup.cs class with two methods in it configure services and configure in configure services you would register services for dependency injection in configure you outline your middleware pipeline order and structure but what does this look like in our new project where do you put dependency injection the answer is between this first builder statement and this app.builder.build so for instance if i wanted to add authentication

i would simply do that but in order to use it in my middleware pipeline before i start my map get i would have to say app dot use authentication

and app.use authorization so in a nutshell everything on this first line where we construct the builder and before we do builder.build is where our dependency injection registration is accomplished everything after builder.build is the middleware pipeline from here all the way down to the bottom okay the code here is not quite complete for actually adding authentication while we're here let's go ahead and do it the way that it needs to be we also need to add services dot add authorization and in order for this to take effect you need to annotate the endpoint that you want protected with the authorize attribute which means the using statement of microsoft asp.net core dot authorization it still isn't quite complete but let's run it to this point and see what happens

okay we've got an unhandled exception we expect this because we've never specified an authentication scheme let's go fix that now now there are a lot of different types of authentication schemes we could choose but in a web api it's typical that we protect the application with something like a bearer token or a jot token so let's go ahead and add jwt bearer to this so we're going to go up to the add authentication and for now we're going to just say add jwt bearer we'll have to resolve that and it doesn't know what that is that's because we need to bring in a nuget package for that so let's do that

there it is asp.net core authentication jwt bearer let's install that

okay so add jwt bearer has been added and it recognizes it should we run it to this point and see what happens

hmm still no authentication scheme was specified and there was no default challenge scheme found all right let's go see if we can fix that well even though we've cascaded this and chained that on we really need to tell the authentication system what our default is

jwt bearer bearer defaults dot authentication scheme and it doesn't know what it is so let's see if we can resolve that there we go

and if we hover over this authentication scheme we'll see that it's just a constant for string with the word bearer in it okay so with all of that in place now let's see what happens

hmm still not happy with this no authentication scheme was specified all right this actually happens quite a bit to me and probably to others out there i went through and i put default authentication scheme and that's sort of the wrong thing we really want default scheme default scheme is for all the schemes specified and if you remember from some of my other videos if you haven't watched them go watch them there's two separate schemes that we need to configure most of times an authentication scheme and a challenge scheme or if i just say default scheme that means that we want jwt bare defaults to be the authentication scheme and the challenge scheme as well this should change things let's try and see what happens

okay and this is exactly what we're looking for a 401 unauthorized because we haven't supplied it with any kind of token

i think this is good enough to demonstrate where to register dependent services and where pipeline middleware gets configured we haven't really set up all the necessary parameters to get jwt bearer authentication working completely but that is out of the scope of what i'm trying to demonstrate today instead what i want to look at next is how do we bring back something like a startup cs file as you can see with these minimal api structures it can get kind of busy trying to put everything all in one file putting all your dependency registration your pipeline middleware configuration and all of your endpoints so there must be a way that we can construct this a little bit in a little bit more organized fashion so let's try to do that now one quick thing before we jump into the startup cs earlier i put this authorize attribute here and while that does work and it is kind of an interesting looking pattern there's another way to make this endpoint protected so let's go ahead and put that on and i'll show you what that looks like there's a fluent way

and it looks like this dot require authorization i think this looks a little more natural in the newer minimal api structure so i'd highly recommend using the extension method approach as opposed to the attribute approach now let's move on and add a startup.cs file

in the interest of time remember we don't need this namespace anymore so let's just paste this in okay so this is actually the same startup structure from the older style templates in the constructor it passes an i configuration and there's a configure services in a configure section and notice that the configure services takes an iservice collection and configure takes an i application builder and an iweb host environment let's see if we can just make that work so let's go here and let's just say startup equals new startup now remember it requires a configuration so will this work well lo and behold off the builder object is a configuration object that seems to be happy passing in as an i configuration to startup so so far so good now what if we do startup dot configure services and remember it requires an iservices collection so let's see if builder.services can be passed in and if it will be happy with that looks like it would be happy with that and if that's the case then i should be able to take all of this code out and we'll switch around a little bit put it in here and instead we'll say services

and also here services so okay that looks like it should work now everything after here is the pipeline right so if i say startup dot configure which is where the pipeline goes it wants an app well looks like i have an app and then an environment let's see if we can get an environment well we certainly can now in theory i should be able to take all of this in fact all of this down to here and put it in startup

hmm it didn't like that part how come it didn't like app eye application builder does not contain a diff definition for map get okay so something's not quite the same let's go back and take a look and see what's different well in here app is a web application but configures expecting an eye application builder what if we just change this around a little bit and said well we're going to pass you a web application would you be happy with that looks like it's happy with that all right it's time to run this and see what it does and see if it works

well we got our 401 so that's a good sign isn't it looks like it's still working and serving so we haven't done too much different the only thing we did was we put a lot of our configuration stuff in the older style startup.configureservices and startup.configure alright so this seems to work and it's pretty cool it allows me to plug in something that i'm comfortable with which is the old startup.cs file and it does clean this up a little bit i guess but i think we can do better i've always hated in the startup file that the words configure services and configure are so close to each other plus you're passing in a configuration that it kind of gets confusing about what goes where why don't we take the opportunity to make this better and i'm going to try and figure that out right now so let's see if we can figure it out i don't know what i'm doing i'm making this part up as we go along this is fun um for one thing i don't like configure services what if we called it register service dependencies or re register dependent services i'm going to call it that and instead of shoving both the service registration and the middleware configuration in the same file what if we created two new files to register those things instead this may be a stupid idea but let's just try it let's go ahead and add a new class and i think i'm going to call this register dependent services not cs naming it's the hardest thing to do and then we'll add another one

we can always rename these later if we don't like this but let's call this one about setup middle where pipeline i'm going to go back here let's make this static and i'm going to make a method public what the heck should it be

i want to take in let's see let's go to here i'd really like to chain it off the end of create builder right so create builder returns a builder so let's say that we take something that

returns a builder

wait what is that builder it's a web application builder it's right this takes a web applica will return a web application builder but it takes a builder so and i'm just going to call this regis register services method and it's going to take a this web application builder and we'll just call it builder okay so that makes sense then at the end of it we'll just say return builder

let's put some code in here to see what that would look like let's add this put it inside of here the difference is we need it to since services doesn't exist we'll just say builder dot services and we'll say builder dot services oops

naming and typing two very hard things so in here let's just put a note to ourselves

register your dependencies

so that's what we'll register them so let's let's actually wire this up so far i'm going to i'm going to change this a little bit we'll start up here let's comment all this out for for now i don't know what i'm doing this is but i'm just going to call this bar i'm going to show you in my mind i kind of have this idea that i want it to look something like this create builder pass in args and then i want to do register services

and add use old to new register services and i'm going to put in here no that's it just register services should be all i need right and then we'll call build

so let's let's wrap this around so it looks a little bit better here and when we do build we'll get app okay and then we can say app.setup

middleware pipeline

and then we'll say app.run i mean that could work let's go see what that would look like so let's make this static i left the namespace and then we'll make a public static this thing will return a web application i guess and we'll call it setup

middleware don't need to pass in anything other than the this web application

app makes sense now let's go into startup.cs and let's pull all of this out i want this stuff to go in here

um configure the pipeline

then we'll just say return app

and then we get over here really all we want to do is say run

i don't know something like that is that better

that should still work should we try and see if it works maybe i broke everything

yeah still works so it's happy with that i don't hate that what do you guys think so if we look back at startup we've successfully taken the configure services and the configure methods rename them to something which i think is a little bit more representative what it does register dependent services and set up middleware pipeline the only thing we haven't accounted for in startup is this constructor that accepted a configuration object that was oftentimes passed in and used down below what can we do about that the whole point of bringing this in was to have the configuration object available when you're setting up your services or sometimes even when you're setting up your middleware well how do we get access to that well remember

builder has the configuration available so we have that already and obviously that's not happy because i'm not doing anything with it but there's configuration and app also has actual access to the configuration values so there's really no point in passing in iconfiguration since both app web application and web application builder have access to the configuration objects at this at this point in time during setup so i think we're good so there you go there's some different ways to to do this i kind of like that because you can put all your register services in place and your setup middleware into place and still see the general flow of creating a web application setting up the middleware building it and running it well we did it we found our cheese and i think we might even have better options now i hope this video helps you and if you found it helpful let me know in the comments below pass it along and share it with someone else too thanks for watching