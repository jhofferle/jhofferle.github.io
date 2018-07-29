---
id: 579
title: Fan-Out Performance Without Remoting
date: 2011-06-02T09:00:31+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=579
permalink: /fan-out-performance-without-remoting/
categories:
  - PowerShell
tags:
  - PowerShell
  - Remoting
---
After my last post on PowerShell Remoting performance, I received an email asking if I had tried [Split-Job](http://poshcode.org/). It works by creating multiple runspaces so that a single command can be run against multiple computers in parallel without remoting enabled. The MaxPipelines parameter controls the number of simultaneous runspaces. I gave the function a try against a group of 100 computers similar to those used in my earlier tests. While it doesn&#8217;t quite have the speed or efficiency of Invoke-Command, it&#8217;s a great alternative if remoting isn&#8217;t an option in your environment. [Tome Tanasovski](http://powertoe.wordpress.com/) tackled the problem by using PowerShell jobs in his [Start-ComputerJobs](http://poshcode.org/) function, but I haven&#8217;t had the opportunity to try it out.

```powershell
$computers | Split-Job -MaxPipelines 20 { % {Get-WinEvent -FilterHashtable @{logname="security";id=4624} -MaxEvents 20 -ComputerName $_} }
```

![image-left](/assets/img/Split-Job_Chart-e1306973532969.png){: .align-left}