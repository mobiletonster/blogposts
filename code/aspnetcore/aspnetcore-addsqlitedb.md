In this post we will learn how to add a sqlite database to an ASP.NET core project using entity framework core, code first.

First, let's add some nuget packages. Start by launching "manage nuget packages" window and add `Microsoft.EntityFrameworkCore.Design`.

![Image 1](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/1-efcoredesign-cropped.jpg#screenshot "manage nuget - add ef core design")


Next, add `Microsoft.EntityFrameworkCore.Sqlite`.

![Image 2](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/2-efcoresqlite-cropped.jpg#screenshot "manage nuget - add ef core sqlite")


Check to be sure that we have actually added those dependencies:

![Image 3](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/3-dependenciesadded-cropped.jpg#screenshot "dependencies appear in solution explorer")

We have .Design and .Sqlite packages added.

Add a new folder to contain all the data items, then add a new class `AppUser.cs`

![Image 4](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/4-datafolder-appuserclass-cropped.jpg#screenshot "screenshot of solution explorer showing data folder and AppUser class")

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
 
![Image 5](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/5-app.db-cropped.jpg#screenshot "screenshot of solution explorer showing app.db file")

Returning to the `Startup.cs` file in the `ConfigureServices` method, let's add a services.AddDbContext like this:

```csharp
services.AddDbContext<AuthDbContext>(options => options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
```

Although order doesn't typically matter when registering dependency services, I have placed this line just below `services.AddControllersWithViews()`. Be sure to resolve all references with their appropriate using statements. I like to use the `CTRL + .` shortcut while the cursor is positioned on the unresolved item to invoke the menu. 

Now it is time to use the nuget package manager console. To load the console, go to the menu View -> Other Windows -> Package Manager Console:

![Image 6](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/6-findpackagemanger-cropped.jpg#screenshot "Visual Studio Menu showing how to load Package Manager Console")


![Image 7](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/7-nugetpackagemanger-cropped.jpg#screenshot "Package Manager Console")

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

```console
dotnet ef migrations add Initial -o Data
```

When it completes it will have added a migration file to the Data directory.

```csharp
public partial class Intitial : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "AppUsers",
            columns: table => new
            {
                UserId = table.Column<int>(type: "INTEGER", nullable: false)
                    .Annotation("Sqlite:Autoincrement", true),
                Provider = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                NameIdentifier = table.Column<string>(type: "TEXT", maxLength: 500, nullable: true),
                Username = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Password = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Email = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Firstname = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Lastname = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Mobile = table.Column<string>(type: "TEXT", maxLength: 250, nullable: true),
                Roles = table.Column<string>(type: "TEXT", maxLength: 1000, nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_AppUsers", x => x.UserId);
            });

        migrationBuilder.InsertData(
            table: "AppUsers",
            columns: new[] { "UserId", "Email", "Firstname", "Lastname", "Mobile", "NameIdentifier", "Password", "Provider", "Roles", "Username" },
            values: new object[] { 1, "bob@admonex.com", "Bob", "Tester", "800-555-1212", null, "pizza", "Cookies", "Admin", "bob" });
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(
            name: "AppUsers");
    }
}

```

You will see there are two methods; `Up` and `Down` for creation and a destruction of our database. During the creation an `AppUsers` table will be added to the database with a primary key of `UserId` using an `Autoincrement` value from Sqlite. Notice also that the `migrationBuilder.InserData()` command will insert our `seed` data into our newly created table.

Finally, we want to apply this migration to our database by running the following command in the package manager console:

```console
dotnet ef database update
```

The first time we run this command, it will create the `app.db` file and add it to our `Database` folder. If you want to inspect the contents of this database, you can download a tool called [DB Browser for SQLIte](https://sqlitebrowser.org/)

To use it from Visual Studio, right click the `app.db` file in Solution Explorer and select `Open With`. 

![Image 8](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/8-sqlitedbbrowser-cropped.jpg#screenshot "Use Open With to define tool to open our app.db file")

It will give you a dialog to let you select which program you want to use to open this file. Click the `Add` button then click the `...` button to browser to the location of your `DB Browser for SQLite.exe` file. Mine installed into this location:

    "C:\Program Files\DB Browser for SQLite\DB Browser for SQLite.exe"

Click OK to complete and I would `Set as Default`. Now go ahead and open it with our new tool.

![Image 9](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/aspnetcore/images/addsqlitedb/9-dbbrowser-cropped.jpg#screenshot "DB Browser for SQLite app showing rows of data in app.db")

You will see a couple of different tables that it created, one for `AppUser` and one for `__EFMigrationsHistory`. In our `AppUsers` table you'll see that we already have a record in there from our seed data `bob tester` for our `bob tester user`.

That is all it takes to add and connect a sqlite database to an ASP.NET Core project with EF Core code-first!



