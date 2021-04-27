In this post we will learn how to add a sqlite database to an ASP.NET core project using entity framework core, code first.

First, let's add some nuget packages. Start by launching "manage nuget packages" window and add `Microsoft.EntityFrameworkCore.Design`.

![Image 1](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/1-efcoredesign-cropped.jpg#screenshot "manage nuget - add ef core design")


Next, add `Microsoft.EntityFrameworkCore.Sqlite`.

![Image 2](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/2-efcoresqlite-cropped.jpg#screenshot "manage nuget - add ef core sqlite")


Check to be sure that we have actually added those dependencies:

![Image 3](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/3-dependenciesadded-cropped.jpg#screenshot "dependencies appear in solution explorer")

We have .Design and .Sqlite packages added.

Add a new folder to contain all the data items, then add a new class `AppUser.cs`

![Image 4](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/4-datafolder-appuserclass-cropped.jpg#screenshot)

Here is the code for the class:

```csharp
public class AppUser
{
    public int UserId { get; set; }
    public string Provider { get; set; }
    public string NameIdentifier { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public string Email { get; set; }
    public string Firstname { get; set; }
    public string Lastname { get; set; }
    public string Mobile { get; set; }
    public string Roles { get; set; }
    public List<string> RoleList
    {
        get
        {
            return Roles?.Split(',').ToList()??new List<string>();
        }
    }
}
```

This class has some basic fields in it:
UserId, Provider, NameIdentifier, Username, etc. It also contains a string for roles which will hold a comma-delimited list of strings.

There is one more property for RoleList. It will convert a comma-delimited list of `roles` for a user and split them into a list of strings.

Let's add another class. We need to add an AuthDbContext which will inherit from the DbContext in Entity Framework.

```csharp
public class AuthDbContext:DbContext
{
    public DbSet<AppUser> AppUsers { get; set; }
    public AuthDbContext(DbContextOptions<AuthDbContext> options) : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<AppUser>(entity =>
        {
            entity.HasKey(e => e.UserId);
            entity.Property(e => e.UserId);
            entity.Property(e => e.Provider).HasMaxLength(250);
            entity.Property(e => e.NameIdentifier).HasMaxLength(500);
            entity.Property(e => e.Username).HasMaxLength(250);
            entity.Property(e => e.Password).HasMaxLength(250);
            entity.Property(e => e.Email).HasMaxLength(250);
            entity.Property(e => e.Firstname).HasMaxLength(250);
            entity.Property(e => e.Lastname).HasMaxLength(250);
            entity.Property(e => e.Mobile).HasMaxLength(250);
            entity.Property(e => e.Roles).HasMaxLength(1000);

            entity.HasData(new AppUser
            {
                Provider = "Cookies",
                UserId = 1,
                Email = "bob@admonex.com",
                Username = "bob",
                Password = "pizza",
                Firstname = "Bob",
                Lastname = "Tester",
                Mobile = "800-555-1212",
                Roles = "Admin"
            });

        });
    }
}
```

This simple class contains one `DbSet<AppUser>`,  a constructor for`AuthDbContext` and an `OnModelCreating` where the `modelBuilder.Entity` for `AppUser` defines all of the properties and sets attributes such as the primary key, max length and others.  
Finally, with `Entity.HasData` seed information for our data is inserted.

In the seed data, we define "bob tester" user and include the roles of `Admin`.

We need to go to the `appsettings.json` and add a connection string for our database. The connection string definition in our case is the `DefaultConnection` entry as listed below:

```json
"ConnectionStrings": {
    "DefaultConnection": "DataSource=Database\\app.db"
},
``` 
 Constructed this way, Entity Framework will add an `app.db` (sqlite) file in the data folder.
 
![Image 5](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/5-app.db-cropped.jpg#screenshot)

Returning to the `Startup.cs` file

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
