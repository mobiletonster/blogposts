today on code hack moment we will learn how to add a sqlite database to an asp.net core project using entity framework core code first

to get going we're going to add a sqlite database as a local file i'm going to start by going to manage nuget packages and add a couple of important packages i'm going to need the entity framework core design

as well as the entity framework core dot sqlite

okay and we'll just check and make sure that we have actually added those dependencies and it looks like we've got core design and core sequel light looks like we're in good shape

next thing i want to do is add a new folder to put all my data in then i want to add a new class

this is going to be our app user dot cs file

okay i pasted this class in it has a few fields in it user id provider name identifier username password email first name last name mobile and a string for roles then i want to add one more property for roll list and all this will do is take the comma delimited list of roles that we could add to a user and split them out into a list of strings that define the role list and we'll use that later let's add another class

i'm going to add an auth db context

and this will inherit from db context in entity framework

in the interest of time i've just copy and pasted this we have one db set of type app user we have our constructor off db context and we have our on model creating where we have the model builder dot entity of app user where we defined all all of the properties that matter and we can set things like which is the key and what the max length is and finally we have this entity.has data which is seed information we can precede our data with a bob tester user and we've included the roles of admin next we want to go to the app settings json and we want to add a connection string for our database the connection string i've put in here for the default connection is data source data slash app.db we'll see that this should put an app.db file in the data folder when we're all done next we want to go to our startup

and it appear above authentication order doesn't really matter but this is where i want to put it let's add a services dot add db context or auth db context

and we'll need to solve that

some options here and we'll use options.use sqlite solve that as well

all right too i guess then we'll do configuration dot get connection string and we want the default connection

semicolon there and i think that will be all we need in the startup now it's time to go to the package manager console and in the package manager console we want to run a few different commands so the first one is dotnet tool install i like to install this globally you don't have to but i think you'll use it enough that it makes sense to do it it says i already have it installed so that's great if you need to you can update the latest by running dotnet tool update dash dash global

dot net dash ef

okay cool so now we have we've made sure we have the latest now we want to what i want to do is run a net ef migrations add initial and i want that to be put into the data folder

it said no project was found so let's look in this directory we're in the solutions directory so let's go down a directory

now let's run that command again it's done and you'll see that we have a migration file that's been added here and we can look at this migration file and there's two methods there's an up method and then a down method so creation and a destruction and during the creation what it's going to do is create an app users table in the database because this is the primary key and it's going to insert this data so that's all based on the the off db context that we created on the on model creating this tool the migration tool looked at that and realized it needed to make sure that that table existed and to do the insert data as part of that so the last thing to do here is to run.net ef database update

we now have an app.db table here so if we want to take a look at this and see what's in it we can open this with a tool that i haven't registered yet so i should add that tool

okay and it's called db browser for sql lite and i'm just going to go ahead and select that

now we'll just click open and now you can see here that there's a couple of different tables that it created one for app users one for ef migrations history and then the sqlite sequence if we look in our app users table you'll see that we already have a record in here and that was from our seed data bob tester for our bob tester user well that looks pretty good and that is all it takes to add and connect sqlite database to an asp.net core project with ef core code first click the card at the end of this video to see the rest of this project in action thanks for watching

you