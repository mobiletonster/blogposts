## What is StarDrive?
I wanted to create a sample application that could demonstrate how to use Asp.Net Core and SignalR to `remotely control connected computers`. There once was a feature in Microsoft's OneDrive application that would allow you to browse your connected computer's files that were not being explicitly tracked by OneDrive itself.

Let's backup. First, in case you aren't familiar with OneDrive, it is Microsoft's cloud drive option, similar to DropBox or Google Drive. It allows you to install a "client agent" on any of your computers and sync them with the cloud while tracking a set of specific folders on your computer. This is an excellent way to "automatically" backup your most important files to the cloud and sync them to other machines. You can always have access to those synced files whether you are at home, school or at work.

The feature that I am interested in replicating from OneDrive is the remote browsing feature which was deprecated a few years ago. Sometimes you are away from your home machine and there may be a file that you want access to but is not being tracked in the set of synced files in OneDrive. When the feature was active, you could select from a list of your connected PC's by computer name and begin to browse the file system as if you were in the file explorer on that machine using just an ordinary web browser.

In this series, we will demonstrate how to build the components for this technology. Although it is centered around browsing the file system of a remote computer, this same technology and idea can be used to perform many different remote operations, such as remotely checking the temperature of your home while you are away, or checking a web camera, or the status of your garage door, lights, etc. It is up to your imagination to connect the components and build something useful with it!

## The Component Architecture
We need only a few pieces to make this work. 
1. A client agent program
2. A web server
3. Network and internet 

### Client Agent Program
First, we will need a "client" or "agent" program running on the remote machine. In our case, we will create a `console` application that must remain running. We will start by building a regular .Net Core console application, but later convert it slightly so it can be run as a `Windows Service`. If you want to run this application on a different OS, such as Linux or Mac, you can skip the step of turning it into a Windows Service compatible agent. There are other ways to make your console application "run as a service" on Linux/Mac which is out of scope for this demonstration.

### A Web Server
The second piece that we need is a web server reachable through the internet. In our case, we will build an Asp.Net Core web server and host it in Azure, but you can host it just about anywhere as long as it is accessible through the internet. The Asp.Net Core web server will use a SignalR Hub to allow remote agents to make a connection to it and setup a bi-directional websocket for two way communication. This is the key!

There are many different web technologies for displaying HTML in a browser. In this first demonstration, I plan to keep it simple and use a tried and true framework of Asp.Net Core MVC, or Model-View-Controller. This is a server side, dynamically generated HTML that is sent to the browser on a request. It is not a SPA or Single Page Application, although we could certainly make one if that was the desire. I hope you will see how simple it is to use something like MVC or an even Razor Pages to build something like StarDrive without having to bring in a heavy javascript framework like React or Angular, etc.

### Network and Internet
The third piece we need is a network connection to the internet. Normally, if you wanted to expose data from a remote machine, it would require setting up a webserver and opening and forwarding ports to expose it to the outside world. Using SignalR, you will not need to open or forward any ports for the client agent to establish a connection with our web server. This is what makes SignalR and websockets so powerful.

## Enough Overview, Quick Demo
TODO: embed a series of images demonstrating what we will be building, or a video snippet.

## Prerequisites
We will be using Visual Studio 2022 which is the latest release as of this writing. Any edition should do, including the free Community Edition found here:
[VisualStudio.com][def1]

Although this could work on the MAC as well using the Visual Studio for MAC edition, this demonstration is primarily geared toward a PC and will require some adaptation to work with the MAC file system and is out of scope for this tutorial. I may update it in the future with specific steps for MAC, but no promises.

Additionally, if you prefer Rider from Jet Brains, the steps may not align perfectly, but everything should work just fine.

We will also need to have .Net 7 installed, but this should get installed with Visual Studio 2022. In particular, make sure you install the `Web Workloads` when installing VS2022.

## Let's Begin
[-> To Building StarDrive Agent Part 1][def2]

[def1]: https://visualstudio.microsoft.com/
[def2]: stardrive-agent-part1.md
