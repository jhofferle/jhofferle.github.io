---
title: Disable Power Management on Network Adapters and USB Ports with VBScript
date: 2010-06-19T14:04:34+00:00
author: Jason Hofferle
permalink: /disable-power-management-on-network-adapters-and-usb-ports-with-vbscript/
categories:
  - VBScript
tags:
  - registry
  - VBScript
---
This VBscript automates a registry hack to turn off the power management on Network Cards and USB ports. Network Adapters and USB ports have a Power Management Tab under their properties in Windows Device Manager.

Information on the registry changes can be found at [Information about power management setting on a network adapter](https://support.microsoft.com/en-us/help/2740020/information-about-power-management-setting-on-a-network-adapter).

```vb
'******************************************************************************
'  power_mgmt.vbs
'******************************************************************************
'
'  Script to disable power management on network adapters and usb ports
'
'  Jason Hofferle
'  April 15, 2008
'
'  Note that when using this script, the system needs to be rebooted for the new hardware
'  settings to be loaded.  Checkboxes in device manager will not change immediately.
'******************************************************************************
Option Explicit

Const HKEY_LOCAL_MACHINE = &H80000002

Dim objReg
Dim strComputer, strKeyPath, arrSubKeys, arrValueTypes, arrValueNames
Dim subkey, i

on error resume next

'Prompt for computer name
'Comment out this line and use strComputer = "." to skip prompting and run the script
'against the local computer only
strComputer = InputBox("Computer Name or IP Address")
'strComputer = "."

'Connect to registry provider
Set objReg = GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & "\root\default:StdRegProv")

'Set strKeyPath to the USB controller GUID
strKeyPath = "SYSTEM\CurrentControlSet\Control\Class\{36FC9E60-C465-11CF-8056-444553540000}"

'Build an array of the USB controllers
objReg.EnumKey HKEY_LOCAL_MACHINE, strKeyPath, arrSubKeys

'Loop through each USB controller looking for the HCDISABLESELECTIVESUSPEND entry and setting it to 1 if it exists
For Each subkey In arrSubKeys
        'wscript.echo subkey
        objReg.EnumValues HKEY_LOCAL_MACHINE, strKeyPath & "\" & subkey, arrValueNames, arrValueTypes
        for i=0 to ubound(arrValuenames)
         'wscript.echo "Value name: " & arrValueNames(i)
         if arrValueNames(i) = "HcDisableSelectiveSuspend" then
                objReg.SetDWORDValue HKEY_LOCAL_MACHINE, strKeyPath & "\" & subkey, "HcDisableSelectiveSuspend", 1
         end if
        next
Next

'Set strKeyPath to the network adapter GUID
strKeyPath = "SYSTEM\CurrentControlSet\Control\Class\{4D36E972-E325-11CE-BFC1-08002bE10318}"

'Build an array of the network adapters
objReg.EnumKey HKEY_LOCAL_MACHINE, strKeyPath, arrSubKeys

'Loop through each network adapter looking for the PNPCAPABILITIES entry and setting it to demical value 56
For Each subkey In arrSubKeys
        'wscript.echo subkey
        objReg.EnumValues HKEY_LOCAL_MACHINE, strKeyPath & "\" & subkey, arrValueNames, arrValueTypes
        for i=0 to ubound(arrValuenames)
                'wscript.echo "Value Name: " & arrValueNames(i)
                if arrValueNames(i) = "PnPCapabilities" then
                        objReg.SetDWORDValue HKEY_LOCAL_MACHINE, strKeyPath & "\" & subkey, "PnPCapabilities", 56
                End If
        next
Next

Set objReg = nothing
```
