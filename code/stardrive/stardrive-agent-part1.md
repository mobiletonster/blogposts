## StarDrive - Building the Client Agent
Let's begin by building the client agent for the remote machine. As of this writing, the current version of .Net Core is .NET 7 and Visual Studio 2022 is the latest IDE version.

### Create the Project

1. Open Visual Studio (2022).

2. Select **"Create a new project"**

![vs2022 start dialog](images/part1/1-vs-create-new-project.png)

3. The templates dialog will open. There are a lot of templates to choose from and it can be overwhelming. You can use the filters and the searchbox to reduce the options. In the searchbox, type `"console"`. Additionally, select `"C#"` as the language from the language drop down box just below the searchbox.

![vs2022 create new project dialog](images/part1/2-console-app.png)

4. Choose the `Console App` for .Net (not the older .Net Framework version)

5. Choose a name for the application. In this case, I chose `StarDrive.Agent`. Make sure the solution name is simply `StarDrive` to distinguish it from the StarDrive.csproj project file. Click **Next** to continue.

![configure project name dialog](images/part1/3-name-console-app.png)

6. Choose a version of .NET to target. We will want to target the latest as of this writing, version 7. We will leave the checkbox unchecked for "Do not use top level statements". Then click CREATE.

![.net version selector](images/part1/4-net-version.png)


7. This will generate our project from the base template and place the project inside of a solution. In this screenshot, notice that the project opened the Program.cs file, which is the starting point for our application.

![initial console project created with defaults](images/part1/5-we-have-liftoff.png)

We now have a project created!

### First Run
Let's run the project and make sure everything happens as expected.  To (build) and run the project, simply press the Play button on the toolbar:

![play button in VS2022](images/part1/6-click-run-button.png)

This will build the app, and run it once it has compiled.

![console running hello world](images/part1/7-console-running.png)

Not too exciting here, but notice that it output the text "Hello, World!" That's good, it means it worked.


### Directory Browsing Code:
One of the key responsiblities for our StarDrive app will be to browse the directories of the remote machine. Specifically, it will be the responsiblity of the `Agent` console application.

There are a couple of ways to browse directories with C# and .Net Core. Using the `System.IO` namespace, we can employ some static methods to do our work. Perhaps you have seen code like this before?

```C#
string path = @"C:\StarDriveData\";

string[] folders = System.IO.Directory.GetDirectories(path);
string[] files = System.IO.Directory.GetFiles(path);

foreach (var folder in folders)
{
    Console.WriteLine(folder);
}

foreach (var file in files)
{
    Console.WriteLine(file);
}
```
The `GetDirectories()` and `GetFiles()` methods return an array of strings with the full path of the directories or files located immediately at the designated path.

If you were to run the code above, you might see output that look like this:

```cmd
C:\StarDriveData\Documents
C:\StarDriveData\Music
C:\StarDriveData\Pictures
C:\StarDriveData\Videos
C:\StarDriveData\SampleText.txt
```

In the above code block, the first four entries are folders or directorys. The last entry is a file `SampleText.txt`.

The trouble with relying on these two methods is that they don't offer more details such as FileSize, LastModifiedDate, etc. To get the information we will need, we can use another class called `DirectoryInfo`. This class provices similar data, except it also includes FileSize, LastModifiedMethod and more. Let's rewrite the above code to use DirectoryInfo:

```C#
string path = @"C:\StarDriveFiles\";

DirectoryInfo di = new DirectoryInfo(path);
if (di.Exists)
{
    FileInfo[] files = di.GetFiles();
    foreach(var f in files)
    {
        Console.WriteLine(f.Name);
    }

    DirectoryInfo[] folders = di.GetDirectories();
    foreach(var f in folders)
    {
        Console.WriteLine(f.Name);
    }
}
```

The return types of `FileInfo[]` and `DirectoryInfo[]` give us more than just a string of the path, but rather several useful file/directory attributes. It is a good idea to wrap the functions `GetFiles` and `GetDirectories` in a check to ensure that the path exists first as noted by this line from the code block above:

```C#
if (di.Exists) { 
    ...
}
```
We will do more with this code later on, but for now if we run this it should work and provide us access to attributes for directories and files in our target path.

### Create a simple DTO (Data Transfer Object)
The goal of our remote app is to send data back to the web server so we can browse the file system. We could try to send the `FileInfo` class over the network to the web server, but it is not an insignificant class.

If we examine the `FileInfo` class in our locals window while debugging, we can see we have a pretty big object on our hands.

![FileInfo class](images/part1/8-file-info-class.png)

So rather than send such a heavy object to the server, let's create a simpler version. 

Add a class called `DirectoryItem.cs` to the solution explorer (in the root should be sufficient for now). The class will be a simple `POCO` class, or Plain Old C# Object.

```C#
internal class DirectoryItem
{
    public bool IsDirectory { get; set; }
    public string? Name { get; set; }
    public string? Path { get; set; }
    public DateTime LastModified { get; set; }
    public long Size { get; set; }
}
```

This will provide us with a nice simple structure that can be serialized to Json or another serializer to send across the wire more efficiently. This is known as a DTO, or Data Transfer Object. It is optimized for serializing and transport.

Next, let's modify the directory browsing code to not just write to the console, but populate a list of `DirectoryItems` for transport:

```C#
List<DirectoryItem> directoryItems= new();

DirectoryInfo di = new DirectoryInfo(path);
if (di.Exists)
{
    FileInfo[] files = di.GetFiles();
    foreach(var f in files)
    {
        var directoryItem = new DirectoryItem() { Name = f.Name, IsDirectory = false, Path = f.FullName, LastModified = f.LastWriteTime, Size=f.Length };
        directoryItems.Add(directoryItem);
    }

    DirectoryInfo[] folders = di.GetDirectories();
    foreach(var f in folders)
    {
        var directoryItem = new DirectoryItem() { Name = f.Name, IsDirectory=true, Path=f.FullName, LastModified=f.LastWriteTime };
        directoryItems.Add(directoryItem);
    }
}
```
Notice the primary difference between `FileInfo[]` and `DirectoryInfo[]` is that files have a `length` or size to them (in bytes) and the `IsDirectory` property is set accordingly.

If you want to see the effect, you can add this code below the above code and run it:

```C#
foreach (var d in directoryItems)
{
    Console.WriteLine($"{d.Name} - {d.IsDirectory}");
}
```

### Compare Serialized Objects
To demonstrate the value of the DTO, we can quickly add a little code to examine how an object may look when it is serialized to JSON. Leveraging the `System.Text.Json` serializer, let's serialize both a `FileInfo` object and our simpler `DirectoryInfo` object and compare.


```C#
var jsonDirectoryItem = JsonSerializer.Serialize(directoryItems[0]);
Console.WriteLine(jsonDirectoryItem);

var oneFile = di.GetFiles()[0];
var jsonFileInfo = JsonSerializer.Serialize(oneFile);
Console.WriteLine(jsonFileInfo);
```

The singular DirectoryItem (our DTO object) serializes nicely to this:

```json
{
    "IsDirectory":false,
    "Name":"SampleText.txt",
    "Path":"C:\\StarDriveData\\SampleText.txt","LastModified":"2022-12-23T23:55:38.8980915-07:00",
    "Size":64
}
```

However, we get an error trying to serialize the `FileInfo` class.

![serializer chokes on the FileInfo class](images/part1/9-serialize-error.png)

We could attempt to try some of the suggestions given in the exception message, but I think we can be satisfied that our DTO is going to be an improvement over the `FileInfo` class.




1) Make the console app a Windows Service friendly app
2) Add SignalR code to connect to a server, however, we don't have a server and we would have build that now....which is a lot

