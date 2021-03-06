---
title: Removing Printer Shares with PowerShell and WMI
date: 2012-03-13T09:00:20+00:00
author: Jason Hofferle
permalink: /removing-printer-shares-with-powershell-and-wmi/
categories:
  - PowerShell
tags:
  - PowerShell
  - Remoting
  - WMI
---
![image-left](/assets/img/Printer-Icon.jpg){: .align-left}Every IT Professional runs into situations where there isn't a utility available for handing a particular task. My first large-scale vbscript fixed a failed upgrade of the Novell Netware Client on thousands of workstations that was preventing users from logging in. Automation technologies like PowerShell excel during those times when something needs to be happen _right now_, and there isn't much time for planning. 

In a more recent situation, I was handed a spreadsheet of computers with printer shares that needed to be removed for security reasons. After [getting the list of computer names](/generating-lists-of-computer-names-with-powershell/) from the spreadsheet into PowerShell, I had to figure out how to remove the printer shares. I very often turn to Windows Management Instrumentation (WMI) in situations like this because I'm familiar with what it can do from my vbscript days where WMI was very often the only way to get anything done. PowerShell makes working with WMI much easier than it ever was with vbscript.

```powershell
Get-WmiObject -ComputerName Computer01 -Class Win32_Printer | foreach {$_.Shared = $False; $_.Put()}
```

The [documentation](https://docs.microsoft.com/en-us/windows/desktop/CIMWin32Prov/win32-printer) for the Win32_Printer class says the _shared_ property is Read/write, so it can be modified. The _put_ method is used to update the printer object. So in this example, all the printers on Computer01 are retrieved, the Shared property on each printer is set to false, and the put method is called to save the change.

```powershell
Invoke-Command -ComputerName $Computers -ScriptBlock {Get-WmiObject Win32_Printer | foreach {$_.Shared = $False; $_.Put()}}
```

This can be done very quickly with PowerShell remoting enabled, in which case the Invoke-Command cmdlet is used. The difference here is that instead of specifying a single computer name, the `$Computers` variable contains a list of computer names. This allows the scriptblock to be sent to all the computers in parallel.

```powershell
Get-WmiObject Win32_Printer -ComputerName Computer01 | Set-WmiInstance -Arguments @{Shared=$False}

Invoke-Command -ComputerName $Computers -ScriptBlock {Get-WmiObject Win32_Printer | Set-WmiInstance -Arguments @{Shared=$False}}
```

Another method is to use the Set-WmiInstance cmdlet to set properties on the printer objects.
