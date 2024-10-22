# Creating WPF Web Components
## Why?
These days, web technology is getting the majority of the attention and improvements. While WPF is a solid, mature choice for a desktop application it has stalled in receiving relevant improvements and updates in recent years. Additionally, many developers have little or no experience using WPF, MVVM, XAML and other similar technologies found in UWP, WPF, Silverlight, WinUI 3, etc. Conversely, there are many developers who have exposure to web development, HTML, CSS and Javascript because of the low barrier to entry and the ubiquity of the web.

Specifically, there may be times we can enhance or improve our WPF application by leveraging embedded web technology in our app. If we examine the source code for Microsoft Map Control available in WPF we find that it is a UserControl that wraps a WebView Control. The heavy lifting is done in web technologies (specifically WebGL). The performance of the Map Control is impressive and has help breathe new life into WPF applications.

See [microsoft-ui-xaml/controls/dev/MapControl/MapControl.xaml on GitHub](https://github.com/microsoft/microsoft-ui-xaml/blob/winui3/release/1.5-stable/controls/dev/MapControl/MapControl.xaml)

## How?
Microsoft updated the Edge Browser by retiring the Trident rendering engine and the Chakra JS engine and adopted the Chromium and V8 engines instead. When they made this change, they also created a new WebView control aptly named WebView2 that developers could embed in their windows applications written in UWP, WinUI 3, WinForms, MAUI (win) and WPF. Each mobile platform has their own implementation of a WebView control for use in native apps. This means code developed in HTML, CSS and JS could be shared with mobile applications if engineered properly.

## Let's get our hands dirty
1. Open Visual Studio (or another IDE, like Rider) and create a new WPF project.
2. Click `Create a new project` and open the dialog window. Filter for WPF and C# as the language, then choose the `WPF Application` template option. 
3. Name the project WpfWebComponents. 
4. Choose the current Long Term Support option for the framework, which is currently .NET 8.0.
5. click Run to make sure our app launches correctly. We should get an application with a blank window.

## Let's add a WebView
See [WebView 2 Getting Started in WPF](https://learn.microsoft.com/en-us/microsoft-edge/webview2/get-started/wpf)
1. Using the nuget package manager, let's add the WebView2 dependency. Search for `WebView2` and you should find `Microsoft.Web.WebView2`. Install the latest stable version (currently 1.0.2849.39).

2. Open `MainWindow.xaml` and add an xmlns (xml namespace)
```xml
xmlns:wv="clr-namespace:Microsoft.Web.WebView2.Wpf;assembly=Microsoft.Web.WebView2.Wpf"
```
3. Add a WebView2 control to the main Grid:
```xml
<Grid>
    <wv:WebView2 Source="https://revoltjs.org/home/gravitypoints"></wv:WebView2>
</Grid>
```

4. Run the app and play with the page that loads.

We have now added a WebView control to our application, pointed its source to a fun page on the internet and effectively created a simple browser application. You could add a text input box that allowed for addresses to be added, forward and back buttons for navigation, and build your own web browser, but that isn't in the scope of this exercise.

## WPF Web Component User Control
Let's create a cutsom user control that wraps the WebView2 control. For our first example, let's create a really simple button.

0. Before we get started, let's remove the WebView control from the main Grid.
```xml
<Grid></Grid>
```
We can also remove the xmlns: `xmlns:wv="clr-namespace:Microsoft.Web.WebView2.Wpf;assembly=Microsoft.Web.WebView2.Wpf"` from the MainWindow.
1. Add a folder for our control called SimpleButton.
2. Right click the project and add a New UserControl(WPF).
3. Name the control SimpleButton.xaml.
4. Drag the control into the SimpleButton folder. **We don't want to change the namespace to include the folder SimpleButton because it would be redundant and would clutter things later.**
5. Add the WebView2 xmlns to the UserControl
```xml
xmlns:wv="clr-namespace:Microsoft.Web.WebView2.Wpf;assembly=Microsoft.Web.WebView2.Wpf"
```
6. Add a WebView2 control the main grid:
```xml
<Grid>
    <wv:WebView2 x:Name="WebView" DefaultBackgroundColor="Transparent"></wv:WebView2>
</Grid>
```
We added a value of `Transparent` for the `DefaultBackgroundColor` property which will make more sense later as we add the custom control to our application.

7. Open the code behind SimpleButton.xaml.cs file.
8. Add the following code to the code behind file:
```csharp
private async void InitializeAsync()
{
    WebView.WebMessageReceived += WebView_WebMessageReceived;
    await WebView.EnsureCoreWebView2Async();
    WebView.CoreWebView2.DOMContentLoaded += CoreWebView2_DOMContentLoaded;
}

private void CoreWebView2_DOMContentLoaded(object? sender, CoreWebView2DOMContentLoadedEventArgs e)
{
    
}

private void WebView_WebMessageReceived(object? sender, CoreWebView2WebMessageReceivedEventArgs e)
{
    
}
```
8. Add the `InitializeAsync();` method call to the constructor like this:
```csharp
public SimpleButton()
{
    InitializeComponent();
    InitializeAsync();
}
```

## How to host local content - a primer:
See [WebView2, Working with local content](https://learn.microsoft.com/en-us/microsoft-edge/webview2/concepts/working-with-local-content?tabs=dotnetcsharp)

In addition to loading remote content, content can also be loaded locally into WebView2. There are several approaches that can be used to load local content into a WebView2 control, including:

* Navigating to a file URL.
* Navigating to an HTML string.
* Virtual host name mapping.
* Handling the WebResourceRequested event.

These approaches are described below.

### Selecting an approach
The various ways of loading local content into a WebView2 control support the following scenarios:

<!--
| Scenario | By navigating to a file URL | By navigating to an HTML string | By using virtual host name mapping | By using `WebResourceRequested` |
| --- |:---:|:---:|:---:|:---:|
| Origin-based DOM APIs | ✔️ | ❌ | ✔️ | ✔️ |
| DOM APIs requiring secure context | ❌ | ❌ | ✔️ | ✔️ |
| Dynamic content | ❌ | ✔️ | ❌ | ✔️ |
| Additional web resources | ✔️ | ❌ | ✔️  | ✔️ |
| Additional web resources resolved in WebView2 process | ✔️ | ❌ | ✔️ | ❌ |
-->

![Image 1](https://raw.githubusercontent.com/mobiletonster/blogposts/main/code/wpf/images/ScreenShot-table1.png#screenshot "WebView2 local content support table")

## Navigating to an HTML string
Before deep diving into any of these approaches, we should start simple with navigation to an HTML string.
1. Add a method to our code behind file:
```csharp
private string LoadHtmlString()
{
    string html = @"<html><body><button>Click Me!</button></body></html>";
    return html;
}
```

2. Inside our `InitializeAsync()` method, add the following lines to the bottom of the method. 
```csharp
string html = LoadHtmlString();
WebView.NavigateToString(html);
```
It should look like this:
```csharp
private async void InitializeAsync()
{
    WebView.WebMessageReceived += WebView_WebMessageReceived;
    await WebView.EnsureCoreWebView2Async();
    WebView.CoreWebView2.DOMContentLoaded += CoreWebView2_DOMContentLoaded;
    string html = LoadHtmlString();
    WebView.NavigateToString(html);
}
```

>To test our code, we need to add the SimpleButton control to our main window.

3. Open `MainWindow.xaml` and add `local:SimpleButton` to the main grid:
```xml
<Grid>
    <local:SimpleButton></local:SimpleButton>
</Grid>
```
4. Run the application. 

You should see a button appear in the UI. This button will be small and won't fill the UI, although the WebView2 control does stretch to fill the entire UI. We made the WebView2 in our SimpleButton.xaml have a transparent background. Time to see what that does.

1. In the MainWindow.xaml file, add `Background="LightGreen"` to the main Grid like this:
```xml
<Grid Background="LightGreen">
    <local:SimpleButton></local:SimpleButton>
</Grid>
```

2. Run the application again.

You should observe that the green comletely surrounds the button. We can see the effects of the Transparent background of the WebView in effect.


## Navigate to string, but with a file?
> We are able to `NavigateToString()` pretty succesfully, however, we don't get intellisense help with our HTML. This is not ideal. Let's tweak our application and see if we can extract the html into a seperate file.

1. Right-click the `SimpleButton` folder and `Add New Item` of type `HTML Page`. Name the page `index.html` and click Add.
2. The page should load into the editor automatically. Find the `<body>` tag and add the `<button>Click Me File!</button>` tag between the body tag elements.
3. Hit save. 

We basically have the same thing (although much richer with the template) that our navigate to stream had, except we get richer support for code completion this way.

Now we need to figure out how to get the HTML read in from disk and into memory.

1. Add the following method to the code behind:
```js
private string LoadHtmlFromFile()
{
    string html = string.Empty;
    string? exePath = Path.GetDirectoryName(Assembly.GetExecutingAssembly().Location);
    string filePath = Path.Combine(exePath, "SimpleButton", "index.html");

    if (File.Exists(filePath))
    {
        html = File.ReadAllText(filePath);
    }
    return html;
}
```
2. Change the `InitializeAsync()` method to call `LoadHtmlFromFile();` instead of `LoadHtmlString();`

3. Right click the `index.html` file and select properties. Set the Build Action to `Content` and Copy to Output Directory as `Copy always`.

4. Run the application. Verify that the WebView is loading our on disk version of the index.html file.

Although we read the file from disk, and loaded the HTML into memory, we didn't really use the navigating to file Url approach, but rather cheated our way through the navigate to string method.

## Navigate to file Url
To use the actual navigate to file method, let's change things up a bit.

1. In our InitializeAsync() method of the SimpleButton.xaml.cs file, remove the last two lines and replace them with the below:
```csharp
// REMOVE
string html = LoadHtmlFromFile();
WebView.NavigateToString(html);
```
```csharp
// ADD THESE LINES
WebView.CoreWebView2.SetVirtualHostNameToFolderMapping("SimpleButton", "/SimpleButton/", CoreWebView2HostResourceAccessKind.Allow);
WebView.CoreWebView2.Navigate("http://SimpleButton/index.html");
```

Our method should now look like this:
```csharp
private async void InitializeAsync()
{
    WebView.WebMessageReceived += WebView_WebMessageReceived;
    await WebView.EnsureCoreWebView2Async();
    WebView.CoreWebView2.DOMContentLoaded += CoreWebView2_DOMContentLoaded;
    WebView.CoreWebView2.SetVirtualHostNameToFolderMapping("SimpleButton", "/SimpleButton/", CoreWebView2HostResourceAccessKind.Allow);
    WebView.CoreWebView2.Navigate("http://SimpleButton/index.html");
}
```

2. Run the application and verify that things still work as expected.

It should load the button just as before but without all the heavly lifting of reading in the file.

## Navigate to resource
There are other options available to us. For instance, rather than include the index.html file as Content that gets copied to the output directory, we could embed the index.html file as a resource that gets captured into the .dll itself. This way, the output directory of the shipped product will look much cleaner without a bunch of html, css and js files. This approach works well, but will rely on the WebResourceRequested method to intercept the request for the resource and rather than look for it on disk, resolve it by extracting it from the embedded resource using the ResourceManager class. We may examine this at a later date, but for today, let's continue using the NavigateToFileUrl method instead.

## Size our button
We want to control the size of the Button from the XAML width/height properties. This means we will want the button to fill the entire WebView. Let's make that change in the index.html page.

1. In the index.html page, add a `<style>` tag block in the `<head>` section of our html page. with the following code:

```html
    <style>
        body {
            max-width: 100%;
            overflow-x: hidden;
            min-height: 100vh;
            overflow-y: hidden;
            margin:0;
        }

        #button {
            width: 100%;
            height: 90vh;
        }
    </style>
```
 2. Add an `id` attribute to the `<button>` tag like this:
 ```html
<button id="button">Click Me File!</button>
```

3. Set the Width and Height properties on the SimpleButton usercontrol in the MainWindow.xaml:
```xml
<Grid Background="LightGreen">
    <local:SimpleButton Width="200" Height="100"></local:SimpleButton>
</Grid>
```
 4. Run the application.

 We now have better control of sizing the button from the XAML side.

What if we wanted to set the text to something different than "Click Me File!"? It would be convenient if we could create a dependency property on the Usercontrol.

## Add a Text property for the control
We will add a dependency property that allows us to set the desired text value of the button on our user control. 

1. Add the following code to code behind:

```csharp
public static readonly DependencyProperty ButtonTextProperty = DependencyProperty.Register(
    nameof(ButtonText),
    typeof(string),
    typeof(SimpleButton),
    new PropertyMetadata(string.Empty, ButtonTextValueChanged)
);

private static void ButtonTextValueChanged(DependencyObject d, DependencyPropertyChangedEventArgs e)
{
    if (d is SimpleButton control)
    {
        control.ButtonText = e.NewValue.ToString();
    }
}

private string _buttonText = string.Empty;
public string ButtonText
{
    get => _buttonText;
    set => SetButtonText(_buttonText = value);
}

private async void SetButtonText(string buttonText)
{
    await WebView.EnsureCoreWebView2Async();
    await WebView.CoreWebView2.ExecuteScriptAsync($"setButtonText('{buttonText}')");
}
```

Next, we need to add a function in Javascript for our SetButtonText() to actually apply our dependency property value. 

2. In the index.html file, add the follow code just below the button:
```html
<script type="text/javascript">
    function setButtonText(text) {
        document.getElementById("button").innerHTML = text;
    }
</script>
```

3. Now add a ButtonText property to the SimpleButton instance on the main window so it looks like this:
```xml
    <Grid Background="LightGreen">
        <local:SimpleButton Width="200" Height="100" ButtonText="My Button"></local:SimpleButton>
    </Grid>
```

4. Run the app.

Hmmm... It didn't work. Why not? 

We have a small race condition. The Dependency Property gets set very early and attempts to call the setTextMethod() in the Javascript before the page is loaded and ready to run. To solve this, we should add some code to the DOMContentLoaded event to ensure we get the button text applied at Initialization.

1. Find the CoreWebView2_DomContentLoaded method handler and make it async by adding the async keyword before the void.
2. Add the following code inside the method so it looks like this:
```csharp
private async void CoreWebView2_DOMContentLoaded(object? sender, CoreWebView2DOMContentLoadedEventArgs e)
{
    await WebView.CoreWebView2.ExecuteScriptAsync($"setButtonText('{ButtonText}')");
}
```

3. Run the application again.

This time, immediately after the DOM (document object model) or page is loaded in the WebView, we will apply the proper text.

## Hookup the click event.
We want to hook up the click event in Javascript to the C# side so we can react and do useful things.

1. In our `index.html` page, find the `<script>` tag section and add another function below the first function:
```js
function clicked(elm) {
    if (window.chrome.webview) {
        window.chrome.webview.postMessage('clicked');
    }
}
```
2. Add an onclick="clicked(this)" event to our HTML button:
```html
<button id="button" class="glow-on-hover" role="button" onclick="clicked(this)">Button</button>
```

It should look like this when we are completed:

```html
<body>
    <button id="button" class="glow-on-hover" role="button" onclick="clicked(this)">Button</button>
    <script type="text/javascript">
        function setButtonText(text) {
            document.getElementById("button").innerHTML = text;
        }

        function clicked(elm) {
            if (window.chrome.webview) {
                window.chrome.webview.postMessage('clicked');
            }
        }
    </script>
</body>
```

3. In the code behind of our SimpleButton.xaml.cs file, at the top of the class, and an event like this:
```csharp
public event EventHandler? Clicked;
```

4. Also in the code behind of our SimpleButton.xaml.cs file, find the WebView_WebMessageReceived event handler and change it to look like this:
```csharp
private void WebView_WebMessageReceived(object? sender, Microsoft.Web.WebView2.Core.CoreWebView2WebMessageReceivedEventArgs e)
{
    var clicked = e.TryGetWebMessageAsString();
    if (clicked == "clicked")
    {
        Clicked?.Invoke(this, EventArgs.Empty);
    }
}
```

5. To see it work, head back to the MainWindow.xaml page and add a "Clicked" event handler to the button so it looks like this:
```xml
<local:SimpleButton Width="200" Height="100" ButtonText="My Button" Clicked="SimpleButton_Clicked"></local:SimpleButton>
```

6. In the code behind, add the following method:
```csharp
private void SimpleButton_Clicked(object sender, EventArgs e)
{
    if(sender is SimpleButton button)
        button.ButtonText = "Ouch, that hurt!";
}
```

7. Run the app.

When you click the button, now the C# code behind in the XAMl will get the invocation and be able to respond in various ways. This time, the effect should be that we change the text of the button.

## Summary
In this example, we built a very simple button usercontrol that uses HTML,CSS and Javascript to render the actual UI inside a WebView control, but the user can simply use the XAML based user control as they would any other control in WPF.

### Future considerations
This example is quite simple. More complex controls will require many more dependency properties, many more Javascript methods and a more structured way to control the crossing of the barrier between Javascript and C#.

Typescript may bridge this divide better and help keep the code more robust.

Additionaly, consideration will need to be given for how testing can be implemented for these controls. There are many tools for testing the Web, but understanding how to bridge them in WPF may require some cleverness.

