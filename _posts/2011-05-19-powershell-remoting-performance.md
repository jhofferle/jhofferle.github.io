---
title: PowerShell Remoting Performance
date: 2011-05-19T09:00:39+00:00
author: Jason Hofferle
permalink: /powershell-remoting-performance/
categories:
  - PowerShell
tags:
  - PowerShell
  - Remoting
---
Using fan-out PowerShell remoting has some amazing performance benefits, but it can be difficult to find real-world metrics that describe just how efficient it is. It's challenging to communicate how powerful it can be in a demonstration, because typically a speaker at a conference will only have a couple of virtual machines. Fan-out remoting just doesn't seem that big of an improvement over a foreach loop when you only have a handful of systems.

I'm fortunate enough to work in an environment where we have 64-bit Windows 7 clients with remoting enabled through group policy. When I was preparing for my presentation on remoting last week, I wanted to get some numbers to show how well it worked in a true production environment.

Measure-Command was used to compare a foreach loop that might be used without remoting against Invoke-Command. The command is based on a true scenario where I needed to query event logs on Windows 7 clients during our beta to determine how often a particular event was occurring. For this test, I wanted to retrieve the last 20 successful logon events for each computer.

```powershell
Measure-Command {$results = $computers | % {Get-WinEvent -FilterHashtable @{logname="security";id=4624} -MaxEvents 20 -ComputerName $_}}
```

```powershell
Measure-Command {$results = Invoke-Command -ComputerName $computers -ScriptBlock {Get-WinEvent -FilterHashtable @{logname="security";id=4624} -MaxEvents 20} -ThrottleLimit 50}
```

![image-center](/assets/img/RemotingChart01-e1305755995603.png)

I also ran the same remoting command against larger numbers of computers to see how well things scaled.

![image-center](/assets/img/RemotingChart02-e1305758086684.png)

These commands were run from my workstation against clients all over the United States and most were multiple WAN hops away. This method was used in a real production environment to gather statistics for convincing management that a problem had been solved. PowerShell remoting is a truly powerful feature that really starts shining when you throw hundreds or thousands of computers at it.
