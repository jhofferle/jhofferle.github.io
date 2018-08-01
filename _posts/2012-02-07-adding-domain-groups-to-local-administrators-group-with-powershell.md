---
title: Adding Domain Groups to Local Administrators Group with PowerShell
date: 2012-02-07T09:00:41+00:00
author: Jason Hofferle
permalink: /adding-domain-groups-to-local-administrators-group-with-powershell/
categories:
  - PowerShell
tags:
  - Active Directory
  - PowerShell
---
A common way to add domain groups to the local administrators group on a computer is with the `net` command. This worked well for me until I ran into groups with names longer than 20 characters. That's right, [the NET.EXE /ADD command does not support names longer than 20 characters](https://support.microsoft.com/en-us/help/324639/net-exe-add-command-does-not-support-names-longer-than-20-characters). If `net localgroup /add` is being used in a computer startup script, the groups with long names just won't be added.

So the traditional batch file startup script was replaced with a PowerShell startup script, and this is how I now add domain groups to the local administrators group on computers.

```powershell
([adsi]"WinNT://./Administrators,group").Add("WinNT://DOMAIN/My Extremely Long Group Name with Spaces,group")
```
