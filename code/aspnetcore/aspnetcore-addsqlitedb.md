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

Finally, `Entity.HasData` provides 'seed' data, or an initial record in our database.

In the seed data, we define "bob tester" user and assign the role of `Admin`.

We need to go to the `appsettings.json` and add a connection string for our database. The connection string definition in our case is the `DefaultConnection` entry as listed below:

```json
"ConnectionStrings": {
    "DefaultConnection": "DataSource=Database\\app.db"
},
``` 
 Constructed this way, Entity Framework will add an `app.db` (sqlite) file in the Database folder when we execute the migration code later.
 
![Image 5](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/5-app.db-cropped.jpg#screenshot)

Returning to the `Startup.cs` file in the `ConfigureServices` method, let's add a services.AddDbContext like this:

```csharp
services.AddDbContext<AuthDbContext>(options => options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
```

Although order doesn't typically matter when registering dependency services, I have placed this line just below `services.AddControllersWithViews()`. Be sure to resolve all references with their appropriate using statements. I like to use the `CTRL + .` shortcut while the cursor is positioned on the unresolved item to invoke the menu. 

Now it is time to use the nuget package manager console. To load the console, go to the menu View -> Other Windows -> Package Manager Console:

![Image 6](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/6-findpackagemanager-cropped.jpg#screenshot)


![Image 7](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/7-nugetpackagemanager-cropped.jpg#screenshot)

First, we want to install a tool by typing the following command:

```console
dotnet too instal --global dotnet ef
```

You may already have it installed, but just want to update the tool. If so, run this command:

```console
dotnet tool update --global dotnet-ef
```

We are ready to run some EF commands. Ensure that you are targeting the correct "Default project" in the Package Manager Console and make sure we are in the directory of the project. In this case, I am in the `Authn` project and I need to make sure I'm in the directory containing the `Authn` project itself. I can run a `dir` command to see the working directory of the console, and in my case I need to run `cd Authn` to change my working directory to the project. 

Now lets create an initial migration entry by running this command:


okay cool so now we have we've made sure we have the latest now we want to what i want to do is run a net ef migrations add initial and i want that to be put into the data folder

it said no project was found so let's look in this directory we're in the solutions directory so let's go down a directory

now let's run that command again it's done and you'll see that we have a migration file that's been added here and we can look at this migration file and there's two methods there's an up method and then a down method so creation and a destruction and during the creation what it's going to do is create an app users table in the database because this is the primary key and it's going to insert this data so that's all based on the the off db context that we created on the on model creating this tool the migration tool looked at that and realized it needed to make sure that that table existed and to do the insert data as part of that so the last thing to do here is to run.net ef database update

we now have an app.db table here so if we want to take a look at this and see what's in it we can open this with a tool that i haven't registered yet so i should add that tool

okay and it's called db browser for sql lite and i'm just going to go ahead and select that

now we'll just click open and now you can see here that there's a couple of different tables that it created one for app users one for ef migrations history and then the sqlite sequence if we look in our app users table you'll see that we already have a record in here and that was from our seed data bob tester for our bob tester user well that looks pretty good and that is all it takes to add and connect sqlite database to an asp.net core project with ef core code first click the card at the end of this video to see the rest of this project in action thanks for watching

you
