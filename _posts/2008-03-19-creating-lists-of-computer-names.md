---
id: 336
title: Creating Lists of Computer Names
date: 2008-03-19T14:27:49+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=336
permalink: /creating-lists-of-computer-names/
categories:
  - VBScript
tags:
  - Active Directory
  - VBScript
---
Very often you will want to run a script against a list of computers. I’ve found that the most flexible way is to grab computer names from a text file. When your scripts pull input information from a text file, there are many options available for how you create that text file. This vbscript will create a text file of all computer names in a specified Active Directory container.

I have also written about using PowerShell to [generate lists of computer names](http://www.hofferle.com/generating-lists-of-computer-names-with-powershell/ "Generating Lists of Computer Names with PowerShell").

```vb
Const ADS_SCOPE_SUBTREE = 2
Const ForAppending = 8
Const ForWriting = 2

Set objConnection = CreateObject("ADODB.Connection")
Set objCommand =   CreateObject("ADODB.Command")
objConnection.Provider = "ADsDSOObject"
objConnection.Open "Active Directory Provider"
Set objCommand.ActiveConnection = objConnection
objCommand.CommandText = _
    "Select Name, Location from &#039;LDAP://OU=yourcontainer,DC=yoursubdomain,DC=yourdomain,DC=com&#039; " _
        & "where objectClass=&#039;computer&#039;"
objCommand.Properties("Page Size") = 1000
objCommand.Properties("Timeout") = 30
objCommand.Properties("Searchscope") = ADS_SCOPE_SUBTREE
objCommand.Properties("Cache Results") = False
Set objRecordSet = objCommand.Execute
objRecordSet.MoveFirst

Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objTextFile = objFSO.OpenTextFile _
    ("computer_list.txt", ForWriting, True)

Do Until objRecordSet.EOF
                objTextFile.WriteLine(objRecordSet.Fields("Name").Value) &#039;write name to file
                objRecordSet.MoveNext
Loop
objTextFile.Close
WScript.Echo "Script Completed."
```