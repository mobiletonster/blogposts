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

6. Choose a version of .NET to target. We will want to target the latest as of this writing, version 7. We will leave the checkbox unchecked for "Do not use top level statements". The click CREATE.

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

