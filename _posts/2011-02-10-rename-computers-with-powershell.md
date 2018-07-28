---
id: 138
title: Rename Computers with PowerShell
date: 2011-02-10T15:24:01+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=138
permalink: /rename-computers-with-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
---
Some documentation refers to the Rename-Computer cmdlet, but according to [this](http://www.leeholmes.com/blog/WhereIsRenameComputer.aspx) post, providing an invalid computer name could cause unexpected results. Since local and remote computers can already be renamed using WMI and netdom.exe, the cmdlet was removed. The MSDN documentation [indicates](http://msdn.microsoft.com/en-us/library/aa393056(v=vs.85).aspx) the Rename method will not work on domain members, but I have tested this technique on Windows 7 and XP systems in a domain. 

<pre class="lang:powershell decode:true">Get-WmiObject Win32_ComputerSystem -ComputerName OLDNAME -Authentication 6 |
ForEach-Object {$_.Rename("NEWNAME","PASSWORD","USERNAME")}
</pre>

The Get-Credential cmdlet can be used to mask the password.

<pre class="lang:powershell decode:true">$credential = Get-Credential
Get-WmiObject Win32_ComputerSystem -ComputerName OLDNAME -Authentication 6 |
ForEach-Object {$_.Rename("NEWNAME",$credential.GetNetworkCredential().Password,$credential.Username)}
</pre>

Remember to reboot the computer for the changes to take effect.

<pre class="lang:powershell decode:true">Get-WmiObject Win32_OperatingSystem -ComputerName OLDNAME |
ForEach-Object {$_.Win32Shutdown(6)}
</pre>