---
id: 324
title: Creating AD computer accounts from a CSV
date: 2008-03-19T14:20:44+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=324
permalink: /creating-ad-computer-accounts-from-a-csv/
categories:
  - VBScript
tags:
  - Active Directory
  - VBScript
  - WMI
---
This is an updated script that creates computer accounts in Active Directory. This script uses a comma separated values file as an input instead of two text files.

```vb
&#039;**************************************Heading*********************************
&#039;create.vbs
&#039;
&#039;Jason Hofferle
&#039;04/12/2007
&#039;
&#039;Script to create AD computer accounts from file
&#039;******************************************************************************
option explicit

&#039;**************************************************************************
&#039;Variable Declarations
&#039;**************************************************************************
Const ADS_UF_WORKSTATION_TRUST_ACCOUNT = &h1000
Const ForReading = 1

dim strInputFile, strOU
dim arrTemp
dim objFSO, objInputFile, objRootDSE, objContainer, objComputer
&#039;**************************************************************************


&#039;**************************************************************************
&#039;Configuration
&#039;**************************************************************************
strInputFile = "input.csv" &#039;specifies name of csv input file
strOU = "OU=YourOU,OU=YourOU,OU=YourOU," &#039;The root will be appended
&#039;**************************************************************************


&#039;**************************************************************************
&#039;Global Object Initialization
&#039;**************************************************************************
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objInputFile = objFSO.OpenTextFile(strInputFile, ForReading)
Set objRootDSE = GetObject("LDAP://rootDSE")
Set objContainer = GetObject("LDAP://" & strOU & objRootDSE.Get("defaultNamingContext"))
&#039;**************************************************************************


&#039;**************************************************************************
&#039;Main Script Execution
&#039;**************************************************************************
Do Until objInputFile.AtEndOfStream
        on error resume next
        err.Clear
    arrTemp = Split(objInputFile.Readline, ",", -1, 1)
    if err &lt;&gt; 0 then
                wscript.echo "Error reading line from input file."
                err.Clear
        end if
    set objComputer = objContainer.Create("Computer", "cn=" & arrTemp(0))
    if err &lt;&gt; 0 then
                wscript.echo "Error creating computer account " & arrTemp(0)
                err.Clear
        end if
    objComputer.Put "sAMAccountName", arrTemp(0) & "$"
    objComputer.Put "description", arrTemp(1)
    objComputer.Put "userAccountControl", ADS_UF_WORKSTATION_TRUST_ACCOUNT
    objComputer.SetInfo
    if err &lt;&gt; 0 then
                wscript.echo "Error creating computer account " & arrTemp(0)
                err.Clear
        end if
    set objComputer = nothing
    on error goto 0
Loop

objInputFile.Close

set objFSO = nothing
set objInputFile = nothing
set objRootDSE = nothing
&#039;**************************************************************************
```