---
title: Query Active Directory with PowerShell
date: 2010-06-28T14:31:03+00:00
author: Jason Hofferle
permalink: /query-active-directory-with-powershell/
categories:
  - PowerShell
tags:
  - Active Directory
  - PowerShell
---
This function queries Active Directory for users matching the specified parameter and generates a PSObject with certain properties from the returned objects. The collection of returned objects can be piped to additional PowerShell cmdlets for sorting, formatting or exporting.

```powershell
Function QueryAD
  {
    Param
      (
        [string]$field,
        [string]$value
      )

    Function GetUserInfo
        {
            param
                (
                    [string]$last,
                    [string]$first,
                    [string]$initials,
                    [string]$company,
                    [string]$office,
                    [string]$address,
                    [string]$city,
                    [string]$state,
                    [string]$zip,
                    [string]$country,
                    [string]$phone,
                    [string]$title,
                    [string]$department
                )

            $output = New-Object PSObject
            $output | Add-Member -type NoteProperty -name Last -value $last
            $output | Add-Member -type NoteProperty -name First -value $first
            $output | Add-Member -type NoteProperty -name Initials -value $initials
            $output | Add-Member -type NoteProperty -name Company -value $company
            $output | Add-Member -type NoteProperty -name Office -value $office
            $output | Add-Member -type NoteProperty -name Address -value $address
            $output | Add-Member -type NoteProperty -name City -value $city
            $output | Add-Member -type NoteProperty -name State -value $state
            $output | Add-Member -type NoteProperty -name Zip -value $zip
            $output | Add-Member -type NoteProperty -name Country -value $country
            $output | Add-Member -type NoteProperty -name Phone -value $phone
            $output | Add-Member -type NoteProperty -name Title -value $title
            $output | Add-Member -type NoteProperty -name Department -value $department
            $output
        }


    $searcher = New-Object DirectoryServices.DirectorySearcher
    $searcher.Filter = "(&(objectcategory=person)(objectclass=user)($field=$value))"
    $results = $searcher.FindAll() | Sort-Object @{Expression={$_.Properties.sn}}

    ForEach ($user in $results)
        {
            GetUserInfo `
                $user.properties.sn `
                $user.properties.givenname `
                $user.properties.initials `
                $user.properties.company `
                $user.properties.physicaldeliveryofficename `
                $user.properties.streetaddress `
                $user.properties.l `
                $user.properties.st `
                $user.properties.postalcode `
                $user.properties.c `
                $user.properties.telephonenumber `
                $user.properties.title `
                $user.properties.department
        }
  }
```

The script can be called, the function placed in a profile or the script can be dot-sourced, to allow the ExportGAL function to be called like another cmdlet:
~~~
PS C:\> . .\QueryAD.ps1
~~~
The function accepts two parameters, a field and a value. These are used to build the search query. For example, the following will search for anyone with the last name of Smith:
~~~
PS C:\> QueryAD sn Smith
~~~
Building a collection of everyone in the SouthEast Marketing department could be done with this:
~~~
PS C:\> QueryAD department "SouthEast Marketing"
~~~
The collection of objects can be manipulated just like any other object:
~~~
PS C:\> QueryAD department "SouthEast Marketing" | Select-Object last,phone | Format-Table -AutoSize
PS C:\> QueryAD department "SouthEast Marketing" | ConvertTo-Html | Out-File c:\marketing.html
PS C:\> QueryAD department "SouthEast Marketing" | Export-Csv c:\marketing.csv
~~~
