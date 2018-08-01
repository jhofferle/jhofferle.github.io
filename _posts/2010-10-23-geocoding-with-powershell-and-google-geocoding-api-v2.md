---
title: Geocoding with PowerShell and Google Geocoding API V2
date: 2010-10-23T10:48:34+00:00
author: Jason Hofferle
permalink: /geocoding-with-powershell-and-google-geocoding-api-v2/
ninja_forms_form:
  - "0"
categories:
  - PowerShell
tags:
  - Google
  - PowerShell
---
This function uses the [Google Maps Geocoding API Version 2](https://developers.google.com/maps/articles/geocodingupgrade). It accepts objects with address, city, state and zip parameters. It then uses that information to find the longitude and latitude of the location and adds additional properties to the object. I use it with [Import-CSV](http://technet.microsoft.com/en-us/library/dd347665.aspx) and [Export-CSV](http://technet.microsoft.com/en-us/library/dd347724.aspx) to add information to a spreadsheet.

```powershell
Function Add-Geocode
{
    Begin
    {
        $webClient = New-Object System.Net.WebClient
        $pre = "http://maps.google.com/maps/geo?q="
        $suf = "&output=csv&sensor=false"

        $accHash = @{}
        $accHash.Add(0,"Unknown")
        $accHash.Add(1,"Country")
        $accHash.Add(2,"State")
        $accHash.Add(3,"County")
        $accHash.Add(4,"City")
        $accHash.Add(5,"Postal code")
        $accHash.Add(6,"Street")
        $accHash.Add(7,"Intersection")
        $accHash.Add(8,"Address")
        $accHash.Add(9,"Building")
    }

    Process
    {
        $add = "$($_.address -replace " ","+")+"
        $city = "$($_.city -replace " ","+")+"
        $state = "$($_.state -replace " ","+")+"
        $zip = "$($_.zip -replace " ","+")+"

        $queryString = "$($pre)$($add)$($city)$($state)$($zip)$($suf)"
        Write-Verbose $queryString
        $returnValue = $webClient.DownloadString($queryString).Split(",")
        If ([int]$returnValue[0] -eq 200)
        {
            $_ | Add-Member NoteProperty GEOACCURACY $accHash.Item([int]$returnValue[1])
            $_ | Add-Member NoteProperty LATITUDE $returnValue[2]
            $_ | Add-Member NoteProperty LONGITUDE $returnValue[3]
        }
        Else
        {
            Write-Warning "Could not GeoCode $($_.site). Return value was $($returnValue[0])"
        }

        Write-Output $_
    }
}
```
