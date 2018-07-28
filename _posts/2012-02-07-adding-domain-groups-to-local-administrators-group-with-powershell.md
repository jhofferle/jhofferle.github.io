---
id: 892
title: Adding Domain Groups to Local Administrators Group with PowerShell
date: 2012-02-07T09:00:41+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=892
permalink: /adding-domain-groups-to-local-administrators-group-with-powershell/
categories:
  - PowerShell
tags:
  - Active Directory
  - PowerShell
---
A common way to add domain groups to the local administrators group on a computer is with the `net` command. This worked well for me until I ran into groups with names longer than 20 characters. That&#8217;s right, <a href="http://support.microsoft.com/kb/324639" title="NET.EXE /ADD command does not support names longer than 20 characters" target="_blank">the NET.EXE /ADD command does not support names longer than 20 characters</a>. If `net localgroup /add` is being used in a computer startup script, the groups with long names just won&#8217;t be added.

So the traditional batch file startup script was replaced with a PowerShell startup script, and this is how I now add domain groups to the local administrators group on computers.

<pre class="lang:powershell decode:true">([adsi]"WinNT://./Administrators,group").Add("WinNT://DOMAIN/My Extremely Long Group Name with Spaces,group")
</pre>