---
id: 409
title: ScriptBlock Flexibility in PowerShell
date: 2011-04-27T09:00:23+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=409
permalink: /scriptblock-flexibility-in-powershell/
ninja_forms_form:
  - "0"
categories:
  - PowerShell
tags:
  - PowerShell
  - Scripting Games
---
I decided to start sharing some of my submissions for the 2011 Microsoft Scripting Games that received decent ratings. This is my entry for [Advanced Event 3](http://blogs.technet.com/b/heyscriptingguy/archive/2011/04/06/the-2011-scripting-games-advanced-event-3-use-powershell-to-query-classic-event-and-etl-diagnostic-logs.aspx), where the idea was to create a script for event log gathering. The unique part is how a single scriptblock is invoked two different ways. I don&#8217;t know that I would always integrate this into a single function, but it&#8217;s an interesting example of how flexible scriptblocks can be.

Any code that needs to reach out and touch other computers is going to benefit from PowerShell remoting. The more computers you have to connect to, the more efficient it is to use remoting. I wanted to utilize remoting the way I would in my own production environment, but I also wanted the option to use the built-in remoting capabilities found in the Get-WinEvent cmdlet. Not everyone is lucky enough to have remoting configured everywhere.

I created a scriptblock that was essentially a self-contained script with parameters. All of the actual &#8220;work&#8221; is handled in this scriptblock. The rest of the function is figuring out how to launch that scriptblock.

```powershell
$ScriptBlock = {
        
      Param
      (
        [String]$Name,
        [Int]$NumOfEvents,
        [Int]$NumOfDays,
        [Int]$ID,
        [Int]$Level
      )
        
      $filter = @{LogName=""}
            
      if ($ID)
      {
        $filter.Add("ID",$ID)
      }
        
      if ($Level)
      {
        $filter.Add("Level",$Level)
      }
      
      if (Test-Connection -ComputerName $Name -Count 1 -Quiet)
      {
        $logNames = Get-WinEvent -ComputerName $Name -ListLog * | 
          Where-Object {$_.RecordCount -and $_.IsEnabled}
        
        foreach ($log in $lognames)
        {
          $filter.LogName = $log.LogName
            
          Get-WinEvent -ComputerName $Name `
                       -FilterHashTable $filter `
                       -MaxEvents $NumOfEvents `
                       -ErrorAction SilentlyContinue |
            Where-Object {$_.TimeCreated -ge (Get-Date).AddDays(-$NumOfDays).Date}
        }
      }
    }
```

With the script-within-a-script ready, I just needed to launch it appropriately. The Get-LoggedEvent function has a UseRemoting switch. When this switch is specified, I want to use Invoke-Command to take advantage of PowerShell&#8217;s fan-out remoting capabilities. I pass Invoke-Command the array of names, the scriptblock and the arguments to be passed to the scriptblock. The scriptblock is given &#8216;localhost&#8217; for the computer name, and Invoke-Command takes care of running a copy of the scriptblock on each computer.

```powershell
if ($UseRemoting)
    {
      Invoke-Command -ComputerName $ComputerName `
                     -ScriptBlock $ScriptBlock `
                     -ArgumentList 'localhost', `
                                   $NumberOfEvents, `
                                   $NumberOfDays, `
                                   $EventID, `
                                   $LevelTable[$Severity]  
    }
```

If UseRemoting is _not_ specified, I need to unwrap the array of computer names and call the scriptblock once for each computer name. This time, I need to pass the actual computer name to the scriptblock, because the built-in remoting capabilities of Get-WinEvent will be used instead of letting Invoke-Command run the scriptblock locally on each system.

```powershell
else
    {
        foreach ($Name in $ComputerName)
        {
          &$ScriptBlock -Name $Name `
                        -NumOfEvents $NumberOfEvents `
                        -NumOfDays $NumberOfDays `
                        -ID $EventID `
                        -Level $LevelTable[$Severity]
        }
    }
```

With any script, there are always improvements that can be made. [Bartek Bielawski](http://becomelotr.wordpress.com/) (who ended up winning the 2011 Scripting Games) made a good suggestion that I use splatting for those long lists of parameters instead of backticks. He also pointed out that I don&#8217;t need to specify false as a default value for switch parameters because the value will already default to false.

```powershell


Has the same effect as:

```powershell


All of my entries for the 2011 Scripting Games can be found at [PoshCode](http://2011sg.poshcode.org/Scripts/By/114.html).

Complete Script:

```powershell
# -----------------------------------------------------------------------------
# Script: Get-LoggedEvent.ps1
# Author: Jason Hofferle
# Date: 04/06/2011
# Version: 1.0.0
# Comments: This script is based around the Get-LoggedEvent function. This 
#  function was designed with the intent that gathering a list of computer 
#  names and formatting output would be done by other cmdlets in the pipeline. 
#  These type of tasks gain so much speed through PowerShell remoting that an 
#  optional parameter was added that will use fan-out remoting instead of 
#  querying each computer individually.
# -----------------------------------------------------------------------------

Function Get-LoggedEvent
{
  [CmdletBinding()]
  Param
  (
    [String[]]
    $ComputerName = $Env:ComputerName,
       
    [Int]
    $NumberOfEvents = 1,
       
    [Int]
    $NumberOfDays = 0,
        
    [Int]
    $EventID,

    [ValidateSet("Critical","Error","Warning","Informational")]
    [String]
    $Severity,
        
    [Switch]
    $UseRemoting = $false
  )

  Begin
  {
    # For converting human-friendly text to numbers used in query        
    $LevelTable = @{
      Critical=1
      Error=2
      Warning=3
      Informational=4}
    
    # Specify a scriptblock that can be run locally, or passed to
    # Invoke-Command when using PowerShell remoting.        
    $ScriptBlock = {
        
      Param
      (
        [String]$Name,
        [Int]$NumOfEvents,
        [Int]$NumOfDays,
        [Int]$ID,
        [Int]$Level
      )
        
      $filter = @{LogName=""}
            
      if ($ID)
      {
        $filter.Add("ID",$ID)
      }
        
      if ($Level)
      {
        $filter.Add("Level",$Level)
      }
      
      if (Test-Connection -ComputerName $Name -Count 1 -Quiet)
      {
        $logNames = Get-WinEvent -ComputerName $Name -ListLog * | 
          Where-Object {$_.RecordCount -and $_.IsEnabled}
        
        foreach ($log in $lognames)
        {
          $filter.LogName = $log.LogName
            
          Get-WinEvent -ComputerName $Name `
                       -FilterHashTable $filter `
                       -MaxEvents $NumOfEvents `
                       -ErrorAction SilentlyContinue |
            Where-Object {$_.TimeCreated -ge (Get-Date).AddDays(-$NumOfDays).Date}
        }
      }
    }
  }

  Process
  {
    if ($UseRemoting)
    {
      # Pass the entire array of computer names to Invoke-Command
      Invoke-Command -ComputerName $ComputerName `
                     -ScriptBlock $ScriptBlock `
                     -ArgumentList 'localhost', `
                                   $NumberOfEvents, `
                                   $NumberOfDays, `
                                   $EventID, `
                                   $LevelTable[$Severity]  
    }
    else
    {
        # Unwrap $ComputerName array and invoke the scriptblock for each computer
        foreach ($Name in $ComputerName)
        {
          &$ScriptBlock -Name $Name `
                        -NumOfEvents $NumberOfEvents `
                        -NumOfDays $NumberOfDays `
                        -ID $EventID `
                        -Level $LevelTable[$Severity]
        }
    }
  }
  &lt;#
      .Synopsis
      Returns recent events from local and remote computers.
      
      .Description
      The Get-LoggedEvent function queries event logs and event trace logs and
      returns the most recent events from each log.
           
      .parameter ComputerName
      Gets the event information on the specified computers.
      The default is the local computer name.
        
      .parameter NumberOfEvents
      Specifies the number of events to return from each log.
      The default is to return only the latest event.
        
      .parameter NumberOfDays
      Specifies the number of past days to query for events.
      The default is 0, which only returns events logged today.
        
      .parameter EventID
      Gets only the events with the specified event ID.
      The default is all events.
      
      .parameter Severity
      Gets only the events with the specified Level. Valid values are Critical,
      Error, Warning and Informational.
      The default is all types.
      
      .parameter UseRemoting
      Instead of querying each computer individually, use PowerShell remoting
      to connect to the remote systems specified in the ComputerName property.
      The default is to not use remoting.
        
      .Example
      Get-LoggedEvent
        
      Description
      -----------
      This command gets the latest event from each event log on the local
      computer, if there have been events logged today.
      
      .Example
      Get-LoggedEvent -NumberOfEvents 10 -NumberOfDays 3 -Severity Warning
        
      Description
      -----------
      This command gets the latest 10 Warning events from each event log that
      have been logged in the last three days.
      
      .Example
      Get-LoggedEvent -ComputerName DC1,DC2,WIN7 -UseRemoting
        
      Description
      -----------
      This command gets event information from the three computers specified
      using PowerShell remoting.
  #&gt;
}
```