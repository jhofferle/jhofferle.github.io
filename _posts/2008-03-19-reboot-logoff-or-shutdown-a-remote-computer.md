---
title: Reboot, Logoff or Shutdown a Remote Computer
date: 2008-03-19T14:17:13+00:00
author: Jason Hofferle
permalink: /reboot-logoff-or-shutdown-a-remote-computer/
categories:
  - VBScript
tags:
  - VBScript
  - WMI
---
This script is a command-line utility for rebooting, logging off or shutting down remote computers. If no option parameter is specified, a force reboot is default.

```vb
'**************************************************************************
'reboot.vbs
'
'Script for remotely rebooting a computer
'
'
'Jason Hofferle
'04/09/2007
'
'
'**************************************************************************

set objArgs = Wscript.Arguments
strComputer = objArgs(0)

strAction = "Forced Reboot"
iAction = 6

if wscript.Arguments.Count &gt; 1 then
        strAction = objArgs(1)
        select case strAction
                case "logoff"
                        iAction = 0
                case "flogoff"
                        iAction = 4
                        strAction = "Forced Log Off"
                case "shutdown"
                        iAction = 1
                case "fshutdown"
                        iAction = 5
                        strAction = "Forced Shutdown"
                case "reboot"
                        iAction = 2
                case "poweroff"
                        iAction = 8
                case "fpoweroff"
                        iAction = 12
                        strAction = "Forced Power Off"
                case else
                        strAction = "Forced Reboot"
                        iAction = 6
        end select
end if

Wscript.echo Shutdown(strComputer) 'calls shutdown function with strComputer variable

Function Shutdown(strComputer)
        On Error Resume Next
        Set objWMIService = GetObject("winmgmts:" _
            & "{impersonationLevel=impersonate,(Shutdown)}!\\" & strComputer & "\root\cimv2")
        Set colOperatingSystems = objWMIService.ExecQuery _
            ("Select * from Win32_OperatingSystem")
        For Each objOperatingSystem in colOperatingSystems
            ObjOperatingSystem.Win32Shutdown(iAction) 'Change the number in parenthesis to what you need
        If Err.Number &lt;&gt; 0 Then
          WScript.Echo strcomputer & Err.Description
        Else
          WScript.Echo strcomputer & " is performing a " & strAction
        End If
        Err.Clear
        On Error goto 0
        Next
End Function

' Valid auguments for Win32Shutdown method:
'
' 0 Log Off
' 0 + 4 Forced Log Off
' 1 Shutdown
' 1 + 4 Forced Shutdown
' 2 Reboot
' 2 + 4 Forced Reboot
' 8 Power Off
' 8 + 4 Forced Power Off
```

At a command prompt:
~~~
reboot.vbs [computer] [option]
~~~
To force the computer workstation01 to reboot:
~~~
C:\>cscript.exe reboot.vbs workstation01
~~~
To force the computer workstation01 to power off:
~~~
C:\>cscript.exe reboot.vbs workstation01 fpoweroff
~~~
Complete list of options:
  
* logoff
* flogoff
* shutdown
* fshutdown
* reboot
* freboot
* poweroff
* fpoweroff
