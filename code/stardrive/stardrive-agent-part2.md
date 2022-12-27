## StarDrive Building the Client Agent - Part 2

### Directory Browsing Code:
One of the key responsiblities for our StarDrive app will be to browse the directories of the remote machine. Specifically, it will be the responsiblity of the `Agent` console application.

There are a couple of ways to browse directories with C# and .Net Core. Using the `System.IO` namespace, we can employ some static methods to do our work. See  Perhaps you have seen code like this before?

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
The `GetDirectories()` and `GetFiles()` methods return an array of strings ( string[] ) with the full path of the directories or files located immediately at the designated path.

If you were to run the code above, you might see output that look like this:

```cmd
C:\StarDriveData\Documents
C:\StarDriveData\Music
C:\StarDriveData\Pictures
C:\StarDriveData\Videos
C:\StarDriveData\SampleText.txt
```

In the above code block, the first four entries are folders or directories. The last entry is a file `SampleText.txt`.

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

>To learn more read [how to get information about files, folder and drives in C#](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/file-system/how-to-get-information-about-files-folders-and-drives) on the Microsoft Learn website.

[-> To Building StarDrive Agent Part 3](stardrive-agent-part3.md
)