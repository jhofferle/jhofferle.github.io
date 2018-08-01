---
title: Add Domain User or Domain Group to Local Administrators Group
date: 2008-03-20T14:14:18+00:00
author: Jason Hofferle
permalink: /add-domain-user-or-domain-group-to-local-administrators-group/
categories:
  - VBScript
tags:
  - Active Directory
  - VBScript
---
This script will add a domain user or group to the local administrators group of a remote computer. Make sure to change YourDomain to your WinNT domain name. Also change Administrators if you need to add the user or group to a different local group.

```vb
'**************************************Heading*********************************
'add_admin.vbs
'
'Jason Hofferle
'6/14/2005
'
'
'******************************************************************************

Option Explicit

Dim objArgs, objLocalAdminGroup
Dim strComputer, strUser

'On Error Resume Next

Set objArgs = WScript.Arguments

strComputer = objArgs(0)
strUser = objArgs(1)

Set objLocalAdminGroup = GetObject("WinNT://" & strComputer & "/Administrators")

WScript.Echo strComputer & "  " & strUser

objLocalAdminGroup.Add("WinNT://YourDomain/" & strUser)

Set objArgs = Nothing
Set objLocalAdminGroup = Nothing
```

To use at a command prompt:
~~~
add_admin.vbs [computer] [user]
~~~
To add the domain user user01 to the local administrators group on workstation01:
~~~
C:\>cscript.exe add_admin.vbs workstation01 user01
~~~
To add the domain group DomainIT to the local administrators group on workstation01:
~~~
C:\>cscript.exe add_admin.vbs workstation01 DomainIT
~~~
