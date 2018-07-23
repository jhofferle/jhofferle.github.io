---
id: 318
title: Automatic Logoff when Screensaver Activates
date: 2008-03-19T14:18:43+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=318
permalink: /automatic-logoff-when-screensaver-activates/
categories:
  - VBScript
tags:
  - VBScript
  - WMI
---
Many organizations have workstations in common areas for various purposes. If a user walks off without logging out first, that workstation needs to be unlocked by an administrator or hard rebooted before another user can logon. This script is designed to be placed in the startup folder on these common workstations. It runs constantly in the background and checks to see if a screensaver is running. If it finds an active screensaver, it will force a logoff.

<pre class="lang:vbs decode:true">&#039;**************************************************************************
&#039;auto_logoff.vbs
&#039;
&#039;Script that automatically logs off a user when the screensaver activates.
&#039;Intended to contantly run in the background on a scanstation or common
&#039;workstation.
&#039;
&#039;From Microsoft Technet "Hey Scripting Guy!" article.
&#039;http://www.microsoft.com/technet/scriptcenter/resources/qanda/feb07/hey0209.mspx
&#039;
&#039;Jason Hofferle
&#039;02/21/2007
&#039;
&#039;
&#039;**************************************************************************

strComputer = "."

Set objWMIService = GetObject("winmgmts:\\" & strComputer & "\root\cimv2")

Set objEventSource = objWMIService.ExecNotificationQuery _
    ("SELECT * FROM __InstanceOperationEvent WITHIN 10 WHERE TargetInstance ISA &#039;Win32_Process&#039;")

Do While True
    Set objEventObject = objEventSource.NextEvent()
    If Right(objEventObject.TargetInstance.Name, 4) = ".scr" Then
        Set colItems = objWMIService.ExecQuery("Select * from Win32_OperatingSystem")
        For Each objItem in colItems
            objItem.Win32Shutdown(4)
        Next
    End If
Loop
</pre>