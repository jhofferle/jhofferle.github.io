---
title: Geocoding with PowerShell
date: 2010-10-23T20:06:12+00:00
author: Jason Hofferle
permalink: /geocoding-with-powershell/
categories:
  - PowerShell
tags:
  - Google
  - PowerShell
---
Version 3 of Google's Geocoding API does not return information in CSV format. This updated code processes the XML returned by the [new API](http://code.google.com/apis/maps/documentation/geocoding/).

```powershell
Function Add-Geocode
{
    param(
    [Parameter(ValueFromPipeline=$True)]
    [PSObject]$InputObject
    )

    Begin
    {
        $webClient = New-Object System.Net.WebClient
        $pre = "http://maps.googleapis.com/maps/api/geocode/xml?address="
        $suf = "&sensor=false"
    }

    Process
    {
        $queryString = @(
                        $pre
                        $InputObject.address
                        $InputObject.city
                        $InputObject.state
                        $InputObject.zip
                        $suf)

        $queryString = $queryString -replace ' ','+' -join '+'

        [xml]$returnValue = $webClient.DownloadString($queryString)

        If ($returnValue.GeocodeResponse.status -eq "OK")
        {
            $InputObject | Add-Member NoteProperty LATITUDE `
                $returnValue.GeocodeResponse.result.geometry.location.lat
            $InputObject | Add-Member NoteProperty LONGITUDE `
                $returnValue.GeocodeResponse.result.geometry.location.lng
            $InputObject | Add-Member NoteProperty LOCATIONTYPE `
                $returnValue.GeocodeResponse.result.geometry.location_type
        }

        Write-Output $InputObject
    }
}
```
