---
id: 1339
title: Generating Lists of Computer Names with PowerShell
date: 2012-03-06T09:00:26+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=1339
permalink: /generating-lists-of-computer-names-with-powershell/
categories:
  - PowerShell
tags:
  - Active Directory
  - PowerShell
---
PowerShell is particularly good for automating tasks that need to be performed on multiple computers, and many cmdlets are designed to allow multiple computer names to be specified. In many cases, the most difficult task is getting together a list of computers to use with some of the cmdlets and scripts available. There are several ways to generate these lists of names, and very often the situation will dictate which method should be used.

<pre class="lang:powershell decode:true">$Computers = "Computer01","Computer02","Computer03"
Get-WmiObject -Class Win32_OperatingSystem -ComputerName $Computers
</pre>

The simplest way is to manually type names of computers to generate an array of text strings in a variable. That variable can then be passed to cmdlets that accept an array of strings for the input of a ComputerName variable.

<pre class="lang:powershell decode:true">$Computers | Out-File computers.txt
</pre>

If this same list of names might be useful in the future, one way to save the list permanently is to save it to a text file.

<pre class="lang:powershell decode:true">$Computers = Get-Content computers.txt
</pre>

When the list is needed again, the contents of the file can be read and stored to a variable.

<pre class="lang:powershell decode:true"># Load the Microsoft Active Directory Module
Import-Module ActiveDirectory

# Get a list of computers that have WIN7 in their name
$Computers = Get-ADComputer -Filter "Name -like &#039;*WIN7*&#039;" | ForEach-Object {$_.Name}

# Get a list of all computer names
$Computers = Get-ADComputer -Filter * | ForEach-Object {$_.Name}

# Get a list of fully qualified host names
$Computers = Get-ADComputer -Filter * | ForEach-Object {$_.DNSHostName}
</pre>

When there&#8217;s more than a few computers to deal with, it&#8217;s much easier to get those names from the computer accounts in Active Directory. In a correctly configured domain environment, the <a href="http://technet.microsoft.com/en-us/library/dd378937(v=ws.10).aspx" title="Active Directory Administration with Windows PowerShell" target="_blank">Microsoft Active Directory cmdlets</a> can be used to generate lists of computers.

<pre class="lang:powershell decode:true"># Load the Quest Active Directory cmdlets
Add-PSSnapin Quest.ActiveRoles.ADManagement

# Get a list of computers running the Windows Server 2008 operating system
$Computers = Get-QADComputer -OSName "Windows Server 2008*" | ForEach-Object {$_.Name}

# Get a list of computers that are members of the Database Servers group
$Computers = Get-QADComputer -MemberOf &#039;Database Servers&#039; | ForEach-Object {$_.Name}

# Get a list of computers that are located in the Domain Controllers OU
$Computers = Get-QADComputer -SearchRoot "testlab.local/Domain Controllers" | ForEach-Object {$_.Name}
</pre>

Not everyone has the infrastructure in place to use the Microsoft cmdlets. Fortunately, Quest has <a href="http://www.quest.com/powershell/activeroles-server.aspx" title="ActiveRoles Management Shell for Active Directory" target="_blank">freely available</a> cmdlets for interacting with Active Directory.

<pre class="lang:powershell decode:true">Get-ADComputer -Filter * | Export-Csv computerList.csv -NoTypeInformation
Get-QADComputer | Export-Csv computerList.csv -NoTypeInformation
</pre>

Sometimes it&#8217;s difficult to filter a list of names using these cmdlets. Another option is to export all the computer account properties to a Comma Separated Values file, and then use Excel to filter the list.

<pre class="lang:powershell decode:true">$Computers = Import-Csv -Path computerList.csv | ForEach-Object {$_.Name}
</pre>

When the spreadsheet has been narrowed down to the required names, it can then be imported back into PowerShell to get the array of computer names.

The capability of PowerShell to import Csv files is also useful when a list of computers is provided in an Excel spreadsheet from another IT department. It&#8217;s common for reporting software to generate spreadsheets of computer names along with all kinds of other data. By saving these Excel spreadsheets as Csv files, they can easily be imported directly into PowerShell.

[<img src="/assets/img/Excel_ComputerList.png" alt="Excel Computer List" title="Excel_ComputerList" width="640" height="551" class="alignnone size-full wp-image-1344" srcset="https://www.hofferle.com/wp-content/uploads/2012/03/Excel_ComputerList.png 640w, https://www.hofferle.com/wp-content/uploads/2012/03/Excel_ComputerList-150x129.png 150w, https://www.hofferle.com/wp-content/uploads/2012/03/Excel_ComputerList-300x258.png 300w, https://www.hofferle.com/wp-content/uploads/2012/03/Excel_ComputerList-557x480.png 557w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/Excel_ComputerList.png)

[<img src="/assets/img/Excel_SaveAsCsv.png" alt="Excel Save as Csv" title="Excel_SaveAsCsv" width="640" height="472" class="alignnone size-full wp-image-1345" srcset="https://www.hofferle.com/wp-content/uploads/2012/03/Excel_SaveAsCsv.png 640w, https://www.hofferle.com/wp-content/uploads/2012/03/Excel_SaveAsCsv-150x110.png 150w, https://www.hofferle.com/wp-content/uploads/2012/03/Excel_SaveAsCsv-300x221.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/Excel_SaveAsCsv.png)

<pre class="lang:powershell decode:true">$Computers = Import-Csv -Path Computers.csv | Foreach-Object {$_.NetBIOSName.Trim()}
</pre>

Sometimes spreadsheets generated by reporting software will include spaces around the cell data. If computer names have spaces around them, cmdlets like Invoke-Command will not accept those names as input. One way to clean up the computer names is to use string methods like Trim.

<pre class="lang:powershell decode:true">$Computers = 1..254 | ForEach-Object {"192.168.1.$_"}
</pre>

In certain situations, IP addresses can be used instead of computer names. This is helpful when actions need to be run on all computers on a subnet, regardless of their names.

Just like anything in PowerShell, there are lots of different ways to accomplish a task. These are just a few ways to generate a list of computers, which can then be used with any number of scripts and functions.