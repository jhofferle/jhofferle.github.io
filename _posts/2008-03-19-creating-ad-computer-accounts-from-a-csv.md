---
title: Creating AD computer accounts from a CSV
date: 2008-03-19T14:20:44+00:00
author: Jason Hofferle
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
'**************************************Heading*********************************
'create.vbs
'
'Jason Hofferle
'04/12/2007
'
'Script to create AD computer accounts from file
'******************************************************************************
option explicit

'**************************************************************************
'Variable Declarations
'**************************************************************************
Const ADS_UF_WORKSTATION_TRUST_ACCOUNT = &h1000
Const ForReading = 1

dim strInputFile, strOU
dim arrTemp
dim objFSO, objInputFile, objRootDSE, objContainer, objComputer
'**************************************************************************


'**************************************************************************
'Configuration
'**************************************************************************
strInputFile = "input.csv" 'specifies name of csv input file
strOU = "OU=YourOU,OU=YourOU,OU=YourOU," 'The root will be appended
'**************************************************************************


'**************************************************************************
'Global Object Initialization
'**************************************************************************
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objInputFile = objFSO.OpenTextFile(strInputFile, ForReading)
Set objRootDSE = GetObject("LDAP://rootDSE")
Set objContainer = GetObject("LDAP://" & strOU & objRootDSE.Get("defaultNamingContext"))
'**************************************************************************


'**************************************************************************
'Main Script Execution
'**************************************************************************
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
'**************************************************************************
```
