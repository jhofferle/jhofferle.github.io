---
id: 219
title: Updated Get-MonitorInfo
date: 2011-02-21T23:41:24+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=219
permalink: /updated-get-monitorinfo/
categories:
  - PowerShell
tags:
  - PowerShell
  - WMI
---
This is an updated function for gathering monitor information. It uses the technique specified in this [Don Jones post](http://www.windowsitpro.com/blogs/PowerShellwithaPurpose/tabid/2248/entryid/12903/Default.aspx) to accept input from the pipeline or as a parameter.

<pre class="lang:powershell decode:true">Function Get-MonitorInfo
{
    [CmdletBinding()]
    Param
    (
        [Parameter(
        Position=0,
        ValueFromPipeLine=$true,
        ValueFromPipeLineByPropertyName=$true)]
        [alias("CN","MachineName","Name","Computer")]
        [string[]]$ComputerName = $ENV:ComputerName
    )

    Begin {
        $pipelineInput = -not $PSBoundParameters.ContainsKey('ComputerName')
    }

    Process
    {
        Function DoWork([string]$ComputerName) {
            $ActiveMonitors = Get-WmiObject -Namespace root\wmi -Class wmiMonitorID -ComputerName $ComputerName
            $monitorInfo = @()

            foreach ($monitor in $ActiveMonitors)
            {
                $mon = $null

                $mon = New-Object PSObject -Property @{
                ManufacturerName=($monitor.ManufacturerName | % {[char]$_}) -join ''
                ProductCodeID=($monitor.ProductCodeID | % {[char]$_}) -join ''
                SerialNumberID=($monitor.SerialNumberID | % {[char]$_}) -join ''
                UserFriendlyName=($monitor.UserFriendlyName | % {[char]$_}) -join ''
                ComputerName=$ComputerName
                WeekOfManufacture=$monitor.WeekOfManufacture
                YearOfManufacture=$monitor.YearOfManufacture}

                $monitorInfo += $mon
            }
            Write-Output $monitorInfo
        }

        if ($pipelineInput) {
            DoWork($ComputerName)
        } else {
            foreach ($item in $ComputerName) {
                DoWork($item)
            }
        }
    }
}
</pre>