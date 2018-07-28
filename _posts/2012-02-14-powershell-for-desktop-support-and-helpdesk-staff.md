---
id: 800
title: PowerShell for Desktop Support and Helpdesk Staff
date: 2012-02-14T09:00:17+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=800
permalink: /powershell-for-desktop-support-and-helpdesk-staff/
categories:
  - PowerShell
tags:
  - PowerShell
---
[<img src="/assets/img/HelpDesk.jpg" alt="Helpdesk Support Staff" title="HelpDesk" width="347" height="346" class="alignleft size-full wp-image-1045" srcset="https://www.hofferle.com/wp-content/uploads/2012/02/HelpDesk.jpg 347w, https://www.hofferle.com/wp-content/uploads/2012/02/HelpDesk-150x150.jpg 150w, https://www.hofferle.com/wp-content/uploads/2012/02/HelpDesk-300x300.jpg 300w" sizes="(max-width: 347px) 100vw, 347px" />](/assets/img/HelpDesk.jpg)Many of the resources and discussions about PowerShell are geared towards enterprise IT staff responsible for supporting servers. With entire books written about using PowerShell to manage Exchange, vSphere, and other enterprise technologies, it&#8217;s easy for end-user support personnel to get the impression that PowerShell isn&#8217;t something they need to know. I&#8217;m here to tell you that desktop support and helpdesk staff have just as many reasons to learn PowerShell as a server administrator.

I come from a different background than many heavy PowerShell users because I&#8217;ve spent the majority of my IT career on the client side of things. I&#8217;ve assisted end users in person and over the phone, I&#8217;ve performed several desktop deployments, and I&#8217;m working on virtual desktop infrastructure. Even though I&#8217;m not responsible for managing Active Directory or SQL Servers, PowerShell has become a daily tool for me, and I thought I would throw out some reasons why someone that rarely touches a server should make PowerShell a priority.

## PowerShell is Faster at Certain Tasks

Remote desktop and similar technologies are extremely helpful for supporting distant users, but there are some simple tasks that can be accomplished must faster with a PowerShell command or script. Copying a new version of a configuration file, restarting a service, or unlocking an account can all be done much faster from the command line.

<pre class="lang:powershell decode:true">Invoke-Command -ComputerName Computer01 -ScriptBlock {Regsvr32.exe /s c:\Windows\SysWOW64\capicom.dll}
</pre>

It would take minutes to make a remote desktop connection, authenticate and register a dll on a remote workstation. It takes seconds to use PowerShell remoting to perform that same task, and it doesn&#8217;t even disrupt the end user.

## PowerShell Will Save Time

&#8220;I don&#8217;t have time to learn something else&#8221; is one of the most common excuses I hear for not learning PowerShell. While it does initially take more time to learn how to perform a task without pointing and clicking, the first time that task needs to be performed on multiple computers, that time investment will be repaid tenfold. Any time the same task needs to be performed on more than one computer, it&#8217;s a candidate for automation. I automate network printer installations by using the PrintBrm utility to export printerExport files, and import them on other computers.

<pre class="lang:powershell decode:true">Get-EventLog -ComputerName Computer01 -LogName Security -Newest 10
</pre>

If you&#8217;ve ever used Event Viewer to look through the logs on a remote system, you know it can involve some waiting. Knowing how to collect information from workstations can save a massive amount of time, whether you&#8217;re checking for a certain event or generating a report of free disk space.

## PowerShell Provides Automation Capabilities

Many organizations utilize some sort of enterprise solution for deploying software, making changes to client systems and other automated tasks. One of the problems I still run into today is that while we have all these tools, I don&#8217;t have access to utilize them. Something as simple as creating a desktop shortcut for your local users gets put on the backburner by enterprise staff. Since most local IT admins have administrative access to the workstations they&#8217;re responsible for, something like creating shortcuts is an easy task for PowerShell.

<pre class="lang:powershell decode:true"># CreateShortcut.ps1
$Shell = New-Object -ComObject WScript.Shell
$Shortcut = $Shell.CreateShortcut("$Env:Public\Desktop\hofferle.com.lnk")
$Shortcut.TargetPath = "http://www.hofferle.com"
$Shortcut.IconLocation = "shell32.dll,43"
$Shortcut.Save()
</pre>

<pre class="lang:powershell decode:true">Invoke-Command -ComputerName Computer01,Computer02,Computer03 -FilePath C:\CreateShortcut.ps1
</pre>

Many companies still brute force some IT tasks, where it&#8217;s easier to have desktop support touch desktops that develop an enterprise fix. PowerShell allows front-line support to develop automated fixes to save themselves time without relying on an enterprise solution. 

## PowerShell Makes it Easy to Share Resources

Not everyone needs to be a PowerShell expert because modules are an easy way to distribute more complex scripts written by advanced PowerShell users. In my vbscript days, it could be an ordeal to write out documentation on how to use a certain script or write in-line help for it. With comment based help and script modules in PowerShell v2, it&#8217;s incredibly easy to share automated fixes with others. I maintain a module for my organization that packages some complex tasks into easy-to-use functions with built-in help. This allows beginners just getting started to become immediately effective, but those beginners still need to understand the essentials. If you become proficient enough with PowerShell to build these tools for others, you get credit for saving everyone&#8217;s time in addition to your own.

## PowerShell Will Help Your IT Career

This is probably the most important reason to develop PowerShell skills, and it&#8217;s a valid reason for everyone interested in career advancement. It&#8217;s a rare IT shop that doesn&#8217;t use Microsoft products, and Microsoft products are managed with PowerShell. With the consistency of PowerShell, it&#8217;s very easy to apply the basic concepts and patterns to anything. Learn PowerShell now, and when you get a position as a Directory Administrator or Exchange Admin, the PowerShell skills learned now will be immediately useful.