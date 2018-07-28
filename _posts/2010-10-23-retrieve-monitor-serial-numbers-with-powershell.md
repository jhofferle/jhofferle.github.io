---
id: 45
title: Retrieve Monitor Serial Numbers with PowerShell
date: 2010-10-23T10:46:40+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=45
permalink: /retrieve-monitor-serial-numbers-with-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
---
This function gathers monitor [EDID](http://en.wikipedia.org/wiki/Extended_display_identification_data) data using the [WmiMonitorID](http://msdn.microsoft.com/en-us/library/aa394542%28VS.85%29.aspx) WMI class. This class was introduced in Windows Vista, so this function will not work against XP systems. Some of the information can be very useful, such as the serial number, but not all manufacturers correctly encode this information into the EDID. This information can still be gathered from XP systems, but the EDID data needs to be extracted from the registry.

An update to this function that allows for more flexible input can be found [here](http://www.hofferle.com/archives/219).

```powershell
Function Get-MonitorInfo
{
    [CmdletBinding()]
    Param
    (
        [Parameter(
        Position=0,
        ValueFromPipeLine=$true,
        ValueFromPipeLineByPropertyName=$true)]
        [string]$name = &#039;.&#039;
    )

    Process
    {

        $ActiveMonitors = Get-WmiObject -Namespace rootwmi -Class wmiMonitorID -ComputerName $name
        $monitorInfo = @()

        foreach ($monitor in $ActiveMonitors)
        {
            $mon = New-Object PSObject
            $manufacturer = $null
            $product = $null
            $serial = $null
            $name = $null
            $week = $null
            $year = $null

            $monitor.ManufacturerName | foreach {$manufacturer += [char]$_}
            $monitor.ProductCodeID | foreach {$product += [char]$_}
            $monitor.SerialNumberID | foreach {$serial += [char]$_}
            $monitor.UserFriendlyName | foreach {$name += [char]$_}

            $mon | Add-Member NoteProperty Manufacturer $manufacturer
            $mon | Add-Member NoteProperty ProductCode $product
            $mon | Add-Member NoteProperty SerialNumber $serial
            $mon | Add-Member NoteProperty Name $name
            $mon | Add-Member NoteProperty Week $monitor.WeekOfManufacture
            $mon | Add-Member NoteProperty Year $monitor.YearOfManufacture

            $monitorInfo += $mon
        }
        $monitorInfo
    }
}
```