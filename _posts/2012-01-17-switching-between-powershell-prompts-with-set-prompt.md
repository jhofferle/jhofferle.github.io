---
title: Switching Between PowerShell Prompts with Set-Prompt
date: 2012-01-17T09:00:11+00:00
author: Jason Hofferle
permalink: /switching-between-powershell-prompts-with-set-prompt/
categories:
  - PowerShell
tags:
  - PowerShell
---
I'm usually pretty happy with the default PowerShell prompt that displays the current path, however sometimes I just want a generic prompt for screenshots or a demonstration. With an infinite number of custom PowerShell prompts possible, it became apparent that the appropriate prompt depends on the current task. By copying the Set-Prompt function into `$Env:UserProfile\Documents\WindowsPowerShell\profile.ps1`, I have an easy way to quickly switch between these different prompts and I only need to edit my profile when I want to add a new prompt.

If I see a neat custom prompt like a [Domain Controller PowerShell Prompt](http://jdhitsolutions.com/blog/2011/11/domain-controller-powershell-prompt/ "Domain Controller Prompt") or a [Christmas Prompt](http://jdhitsolutions.com/blog/2011/11/friday-fun-a-christmas-prompt/ "A Christmas Prompt"), I just add an additional switch block to Set-Prompt.

![image-center](/assets/img/Set-Prompt-e1326389362668.png)

[Set-Prompt.zip](https://drive.google.com/open?id=15-A18awd1g9tEsxTtjUGgVr7cTCJs7oB)

```powershell
Function Set-Prompt
{
    Param
    (
        [Parameter(Position=0)]
        [ValidateSet("Default","Basic","DC","Xmas")]
        $Action
    )
    
    switch ($Action)
    {
        "Basic"
        {
            Function global:prompt
            {
                $Null
            }
        }
        
        "DC"
        {
            function global:prompt {

            #check and see if logon server is the same as the computername
            if ( $env:logonserver -ne "\\$env:computername" ) {
            #strip off the \\
            $label = ($env:logonserver).Substring(2)
            $color = "Green"
            }
            else {
            $label = "Not Connected"
            $color = "gray"
            }
             
            Write-Host ("[$label]") -ForegroundColor $color -NoNewline
            Write (" PS " + (Get-Location) + "&gt; ")
            }
        }
        
        "Xmas"
        {
            Function global:Prompt {
            $time=([datetime]'12/25/2012'-(get-date)).ToString().Substring(0,11)
            $text="[**Christmas in $($time)**]"
            $text.tocharArray() |foreach {
            if ((Get-Random -min 1 -max 10) -gt 5) {
             $color="RED"
             } 
            else {
             $color="GREEN"
            }
            write-host $_ -nonewline -foregroundcolor $color
            }
            Write (" PS " + (Get-Location) + "&gt; ")
            } #end function
        }
        
        default
        {
            Function global:prompt
            {
                  $(if (test-path variable:/PSDebugContext) { '[DBG]: ' } 
                  else { '' }) + 'PS ' + $(Get-Location) `
                  + $(if ($nestedpromptlevel -ge 1) { '&gt;&gt;' }) + '&gt; '
            }
        }
    }
}
```
