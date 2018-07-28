---
id: 327
title: Creating computer accounts in Active Directory
date: 2008-07-30T14:22:02+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=327
permalink: /creating-computer-accounts-in-active-directory/
categories:
  - VBScript
tags:
  - Active Directory
  - VBScript
---
At my organization, users have one computer they log into. To help keep track of who was on what computer, we put our user’s names in the description of each computer account. This becomes a little labor intensive when we get a shipment of new computers to be deployed, so of course we automate computer account creation.

The new computers are already assigned to users in a spreadsheet. I just created two text files one with a list of users and the other with the computers. This script reads a line from each file and then creates a computer account in AD with the name specified in the computers.txt file and with a description specified in the corresponding line of the users.txt file. This description could be anything. If you just need to create computer accounts, comment out the necessary lines. If you want each account to have the same description, set strUser to whatever you want and comment out the other lines.

If using two separate text files is too clunkly, there is an updated script that uses a csv file as input.

```vb
strTextFile = "computers.txt" &#039;specifies name of txt file with computer names
strUserFile = "users.txt"
Const ADS_UF_PASSWD_NOTREQD            = &h0020
Const ADS_UF_WORKSTATION_TRUST_ACCOUNT = &h1000
Const ForReading = 1

&#039;strUser = "Generic Description for all Computer Accounts"
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objTextFile = objFSO.OpenTextFile(strTextFile, ForReading)
Set objUserFile = objFSO.OpenTextFile(strUserFile, ForReading)
Do Until objTextFile.AtEndOfStream
    strComputer = objTextFile.Readline &#039;reads line from text file
    strUser = objUserFile.Readline

    iretvalue = Create(strComputer) &#039;calls shutdown function with strComputer variable
Loop


Function Create(strComputer)
  Set objRootDSE = GetObject("LDAP://rootDSE")
  Set objContainer = GetObject("LDAP://OU=yourcontainer,OU=anothercontainer," & _
                               objRootDSE.Get("defaultNamingContext"))
  Set objComputer = objContainer.Create("Computer", "cn=" & strComputer)
  objComputer.Put "sAMAccountName", strComputer & "$"
  objComputer.Put "description", strUser
  objComputer.Put "userAccountControl", _
                  ADS_UF_PASSWD_NOTREQD Or ADS_UF_WORKSTATION_TRUST_ACCOUNT
  objComputer.SetInfo
End function
```