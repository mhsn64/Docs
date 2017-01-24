---
title: "Katana Samples | Microsoft Docs"
author: microsoft
description: ""
ms.author: aspnetcontent
manager: wpickett
ms.date: 01/17/2014
ms.topic: article
ms.assetid: 
ms.technology: 
ms.prod: .net-framework
msc.legacyurl: /aspnet/overview/owin-and-katana/katana-samples
msc.type: authoredcontent
---
Katana Samples
====================
by [Microsoft](https://github.com/microsoft)

## Katana Samples

![code sample](katana-samples/_static/image1.png)

**ASP.NET Routes Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/AspNetRoutes/ReadMe.txt)  
In some applications you will want to hook up OWIN components in the Asp.Net route table side by side with non-OWIN components. This sample shows how to use the RouteCollection extension methods MapOwinPath and MapOwinRoute provided by Microsoft.Owin.Host.SystemWeb.


![code sample](katana-samples/_static/image2.png)

**Branching Pipelines Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/BranchingPipelines/ReadMe.txt)  
OWIN request processing pipelines do not need to be linear, they can be branched to process requests in different ways. This sample shows how to construct a branching pipeline based on request paths or other request data such as headers. These components are available in the Microsoft.Owin.Mapping nuget package.


![code sample](katana-samples/_static/image3.png)

**Custom Server Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/CustomServer/MyCustomServer/CustomServer.cs)   
Shows how to use a custom OWIN server when self-hosting OWIN.


![code sample](katana-samples/_static/image4.png)

**Embedded Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/Embedded/ReadMe.txt)  
Some OWIN servers can be run inside of your own process (&quot;self-hosted&quot;). This sample shows how to start an OWIN application using the tools provided by the Microsoft.Owin.Hosting nuget package.


![code sample](katana-samples/_static/image5.png)

**HelloWorld Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/HelloWorld/ReadMe.txt)  
OWIN is a HTTP server API abstraction that enables application portability across various servers. This sample demonstrates how to write a Hello World application using some **simple wrappers** around the raw OWIN abstraction and run it on a web server like ASP.NET.


![code sample](katana-samples/_static/image6.png)

**Hello World Raw OWIN Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/HelloWorldRawOwin/ReadMe.txt)  
This sample demonstrates how to write a Hello World application using the **raw** OWIN abstraction and run it on a web server like Asp.Net.


![code sample](katana-samples/_static/image7.png)

**SignalR Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/SignalR/Program.cs)  
Shows how to self-host SignalR using OWIN / Katana. For more info about self-hosting SignalR, see [Tutorial: SignalR Self-Host](../../../signalr/overview/deployment/tutorial-signalr-self-host.md).


![code sample](katana-samples/_static/image8.png)

**Static Files Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/StaticFilesSample/Startup.cs)   
Shows how to support HTTP requests for static files using OWIN / Katana.


![code sample](katana-samples/_static/image9.png)

**Web API** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/WebApi/ReadMe.txt)   
This sample shows how to host OWIN in IIS and add Web API to the OWIN pipeline.


![code sample](katana-samples/_static/image10.png)

**Web Socket Sample** | [Source Code](http://aspnet.codeplex.com/sourcecontrol/latest#Samples/Katana/WebSocketSample/WebSocketServer/Startup.cs)   
Shows how to support Web Sockets in OWIN by using the [System.Net.WebSockets.WebSocket](https://msdn.microsoft.com/en-us/library/system.net.websockets.websocket(v=vs.110).aspx) class.