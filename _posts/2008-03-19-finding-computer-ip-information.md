---
id: 330
title: Finding computer IP information
date: 2008-03-19T14:23:00+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=330
permalink: /finding-computer-ip-information/
categories:
  - VBScript
tags:
  - VBScript
  - WMI
---
When we went through a subnet change, we found there were many systems with an incorrect default gateway manually entered into their IP information.

This a script I wrote to scan a subnet and dump the IP information to a .csv file.

```vb
&#039;**************************************Heading*********************************
&#039;find_gateway.vbs
&#039;
&#039;Jason Hofferle
&#039;9/16/2004
&#039;
&#039;
&#039;******************************************************************************
Const ForAppending = 8 &#039;doesn&#039;t delete info from the text file, just adds to it
Const ForWriting = 2 &#039;deletes all current data in the text file

Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objTextFile = objFSO.OpenTextFile("gateway_list.csv", ForWriting, True)


strIPSubnet = "192.168.1."

Set objShell = CreateObject("WScript.Shell")

On Error Resume Next
For strIPNode = 1 To 254

        strComputer = strIPSubnet & strIPNode

        Set objScriptExec = objShell.Exec("ping -n 2 -w 1000 " & strComputer)
        strPingStdOut = LCase(objScriptExec.StdOut.ReadAll)

        If InStr(strPingStdOut, "reply from " & strComputer) Then
    Set objWMIService = GetObject("winmgmts:\\"& strComputer & "\root\cimv2")
          If Err.Number &lt;&gt; 0 Then
                  objTextFile.WriteLine(strComputer & "," & Err.Description)
                  Err.Clear
          Else
                Set colAdapters = objWMIService.ExecQuery _
        ("SELECT * FROM Win32_NetworkAdapterConfiguration WHERE IPEnabled = True")
      n = 1
      For Each objAdapter in colAdapters
      strLine = strComputer & "," & objAdapter.MACAddress & "," & objAdapter.DNSHostName
      If Not IsNull(objAdapter.IPAddress) Then
        For i = 0 To UBound(objAdapter.IPAddress)
          strLine = strLine & "," & objAdapter.IPAddress(i)
        Next
      End If
      If Not IsNull(objAdapter.IPSubnet) Then
        For i = 0 To UBound(objAdapter.IPSubnet)
           strLine = strLine & "," & objAdapter.IPSubnet(i)
        Next
      End If
      If Not IsNull(objAdapter.DefaultIPGateway) Then
        For i = 0 To UBound(objAdapter.DefaultIPGateway)
           strLine = strLine & "," & objAdapter.DefaultIPGateway(i)
        Next
      End If
      objTextFile.WriteLine(strLine)
      n = n + 1
      Next
                End If
        Else
                objTextFile.WriteLine(strComputer & "," & "Host unreachable")
        End If
Next
```