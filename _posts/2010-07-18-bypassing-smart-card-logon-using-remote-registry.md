---
id: 321
title: Bypassing Smart Card Logon using Remote Registry
date: 2010-07-18T14:19:41+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=321
permalink: /bypassing-smart-card-logon-using-remote-registry/
categories:
  - VBScript
tags:
  - registry
  - VBScript
---
This VBscript prompts for a computer name or IP Address, connects to that system’s registry over the network and changes the scforceoption key to allow for immediate logon without a smart card. A PowerShell GUI version of this script can be found <a href="http://www.hofferle.com/?p=347" target="_blank">here</a>, and there is also an <a href="http://www.hofferle.com/?p=875" title="Toggle Smart Card Logon Requirement with Set-ScForceOption" target="_blank">updated version</a> that works like a PowerShell cmdlet.

Many organizations now require CAC cards or another type of smart card to logon to workstations. A common way to enforce this is to use the Interactive logon: Require smart card group policy setting. When there is a problem with smart card authentication, this setting makes it difficult for troubleshooting.

<pre class="lang:vbs decode:true">&#039;******************************************************************************
&#039;cac_bypass.vbs
&#039;
&#039;Changes registry key on remote computer to allow logon without CAC card
&#039;
&#039;Jason Hofferle
&#039;21 June 2007
&#039;
&#039;******************************************************************************
Option Explicit

Const HKEY_LOCAL_MACHINE = &H80000002
Dim objReg, strComputer

strComputer = InputBox("Computer Name or IP Address")

On Error Resume Next
Set objReg=GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\default:StdRegProv")
objReg.SetDwordValue HKEY_LOCAL_MACHINE, "SOFTWARE\Microsoft\Windows\CurrentVersion\policies\system", "scforceoption", 0
If Err &lt;&gt; 0 Then
        WScript.Echo "Error changing registry key on " & strComputer
Else
        WScript.Echo "Registry Key changed on " & strComputer
End if

Set objReg = Nothing
</pre>