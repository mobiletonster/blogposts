## StarDrive - Building the Client Agent
Let's begin by building the client agent for the remote machine. As of this writing, the current version of .Net Core is .NET 7 and Visual Studio 2022 is the latest IDE version.

1. Open Visual Studio (2022).

2. Select **"Create a new project"**

![vs2022 start dialog](images/part1/1-vs-create-new-project.png)

3. The templates dialog will open. There are a lot of templates to choose from and it can be overwhelming. You can use the filters and the searchbox to reduce the options. In the searchbox, type `"console"`. Additionally, select `"C#"` as the language from the language drop down box just below the searchbox.

![vs2022 create new project dialog](images/part1/2-console-app.png)

4. Choose the `Console App` for .Net (not the older .Net Framework version)

5. Choose a name for the application. In this case, I chose `StarDrive.Agent`. Make sure the solution name is simply `StarDrive` to distinguish it from the StarDrive.csproj project file. Click **Next** to continue.

![configure project name dialog](images/part1/3-name-console-app.png)

6. 