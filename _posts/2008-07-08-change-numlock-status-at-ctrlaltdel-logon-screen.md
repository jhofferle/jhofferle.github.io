---
id: 306
title: Change Numlock Status at Ctrl+Alt+Del Logon screen
date: 2008-07-08T14:08:56+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=306
permalink: /change-numlock-status-at-ctrlaltdel-logon-screen/
categories:
  - VBScript
tags:
  - registry
  - VBScript
---
Numlock status on a Windows computer is controlled from three places. The BIOS controls the numlock until Windows is booted. At the logon screen, the numlock status along with settings such as the screensaver are loaded from the .DEFAULT user profile. After a user logs on, the settings are loaded from that particular user’s profile.

This script prompts for a computer name and then changes the InitialKeyboardIndicators value so that the numlock is on at the Ctrl+Alt+Del logon screen.

```vb
&#039;******************************************************************************
&#039;numlock.vbs
&#039;
&#039;Changes registry key on remote computer to set numlock status on boot to on
&#039;
&#039;Jason Hofferle
&#039;6 Aug 2007
&#039;
&#039;******************************************************************************
Option Explicit

Const HKEY_USERS = &H80000003

Dim objReg, strComputer

strComputer = InputBox("Computer Name or IP Address")

On Error Resume Next
Set objReg=GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\default:StdRegProv")
objReg.SetStringValue HKEY_USERS, ".DEFAULT\Control Panel\Keyboard", "InitialKeyboardIndicators", 2
If Err &lt;&gt; 0 Then
        msgbox "Error changing registry key on " & strComputer,0
Else
        msgbox "Registry Key changed on " & strComputer,0
End if

Set objReg = Nothing
```

Additional options for the value of InitialKeyboardIndicators:
       
0 &#8211; all indicators off
       
1 &#8211; Caps Lock on
       
2 &#8211; Num Lock on
       
4 &#8211; Scroll Lock on
       
3 &#8211; Caps Lock and Num Lock on
       
5 &#8211; Caps Lock and Scroll Lock on
       
6 &#8211; Num Lock and Scroll Lock on
       
7 &#8211; Caps Lock, Num Lock, and Scroll Lock on