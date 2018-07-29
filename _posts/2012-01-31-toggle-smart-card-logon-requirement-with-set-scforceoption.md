---
id: 875
title: Toggle Smart Card Logon Requirement with Set-ScForceOption
date: 2012-01-31T09:00:48+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=875
permalink: /toggle-smart-card-logon-requirement-with-set-scforceoption/
categories:
  - PowerShell
tags:
  - PowerShell
  - registry
---
Two-factor authentication with smart cards is becoming more common, but it can be a real pain when the computer is broken and Windows is refusing to allow a local account to logon for troubleshooting. The security setting `Interactive logon: Require smart card` may prevent console logons, but if the registry can still be accessed over the network, this requirement can be toggled.

I&#8217;ve covered this same method in the past with a <a href="http://www.hofferle.com/?p=321" title="Bypassing Smart Card Logon using Remote Registry" target="_blank">vbscript</a>, and a <a href="http://www.hofferle.com/?p=347" title="Bypass Smart Card Logon using Remote Registry in PowerShell" target="_blank">PowerShell GUI</a>, but this function is designed to work more like a traditional PowerShell cmdlet. The problem of bypassing a smart card requirement also comes up often enough for me that I decided it warranted an update.

<pre>PS>Set-ScForceOption -ComputerName Computer01 -Status

ComputerName                                                Status
------------                                                ------
Computer01                                                  Enabled


PS>Set-ScForceOption -ComputerName Computer01 -Disable
PS>Set-ScForceOption -ComputerName Computer01 -Status

ComputerName                                                Status
------------                                                ------
Computer01                                                  Disabled


PS>
```

[Set-ScForceOption.zip](https://drive.google.com/open?id=1pMWdKAUll1U5fNSUcCLdyvfcmUbfOmFq)

```powershell
Function Set-ScForceOption
{
    [CmdletBinding(
    SupportsShouldProcess=$true,
    ConfirmImpact="Medium",
    DefaultParameterSetname='Status'
    )]

    Param
    (
        [parameter(Mandatory=$true,
        Position=0,
        ValueFromPipeline=$true,
        ValueFromPipelineByPropertyName=$true)]
        [string[]]$ComputerName,

        [Parameter(
        ParameterSetName='Status')]
        [Switch]
        $Status,

        [Parameter(
        ParameterSetName='Enable')]
        [Switch]
        $Enable,
        
        [Parameter(
        ParameterSetName='Disable')]
        [Switch]
        $Disable
    )

    Process
    {
        ForEach ($Computer in $ComputerName)
        {
            If (-NOT (Test-Connection -ComputerName $Computer -Count 1 -Quiet) )
            {
                Write-Warning "Unable to connect to $Computer"
                Continue
            }
            
            Switch ($PsCmdlet.ParameterSetName)
            {
                'Status'
                {
                    If ( $PSCmdlet.ShouldProcess($Computer, "Get status of ScForceOption") )
                    {
                        $reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Computer)
                        $regKey = $reg.OpenSubKey("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\system")
                        If ($regKey.GetValue("scforceoption") -eq 1)
                          {
                            Write-Output (New-Object -TypeName PSObject -Property @{
                                                                         ComputerName=$Computer
                                                                         Status='Enabled'})
                          }
                        Else
                          {
                            Write-Output (New-Object -TypeName PSObject -Property @{
                                                                         ComputerName=$Computer
                                                                         Status='Disabled'})
                          }
                    }
                }
                
                'Enable'
                {
                    If ( $PSCmdlet.ShouldProcess($Computer, "Enable ScForceOption") )
                    {
                        $reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Computer)
                        $regKey = $reg.OpenSubKey("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\system", $true)
                        $regKey.SetValue("scforceoption", 1)
                    }
                }
                
                'Disable'
                {
                    If ( $PSCmdlet.ShouldProcess($Computer, "Disable ScForceOption") )
                    {
                        $reg = [Microsoft.Win32.RegistryKey]::OpenRemoteBaseKey('LocalMachine', $Computer)
                        $regKey = $reg.OpenSubKey("SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\policies\\system", $true)
                        $regKey.SetValue("scforceoption", 0)
                    }
                }
            }
        }
    }
}
```