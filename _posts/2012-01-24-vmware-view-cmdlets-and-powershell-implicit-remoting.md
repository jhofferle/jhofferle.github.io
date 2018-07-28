---
id: 838
title: VMware View Cmdlets and PowerShell Implicit Remoting
date: 2012-01-24T09:00:26+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=838
permalink: /vmware-view-cmdlets-and-powershell-implicit-remoting/
categories:
  - PowerShell
tags:
  - PowerShell
  - Remoting
  - VMware
---
One of the differences between the VMware View cmdlets and PowerCLI is that the View cmdlets can only be run on the connection server itself. Despite the lack of a Connect-VIServer equivalent, with PowerShell Implicit Remoting it&#8217;s still possible to use these cmdlets from a workstation.

First, PowerShell Remoting needs to be enabled on the Connection Server. There are several ways to configure remoting, but in a domain environment I like to turn it on with group policy. Enabling the automatic configuration of listeners is usually all the configuration necessary to enable remoting on a domain server, but lots of information is available for different situations. The <a href="http://go.microsoft.com/fwlink/?LinkID=135188" title="about_remote_troubleshooting" target="_blank">about_remote_troubleshooting</a> help file is handy, and there are books specific to remoting on my <a href="https://www.hofferle.com/archives/624" title="List of Free PowerShell eBooks" target="_blank">list of free PowerShell ebooks</a>.

[<img src="/assets/img/AllowAutomaticConfigurationOfListeners_1.png" alt="Group Policy Setting for &quot;Allow Automatic Configuration of Listeners&quot;" title="AllowAutomaticConfigurationOfListeners" width="640" height="572" class="alignnone size-full wp-image-1273" srcset="https://www.hofferle.com/wp-content/uploads/2012/02/AllowAutomaticConfigurationOfListeners_1.png 640w, https://www.hofferle.com/wp-content/uploads/2012/02/AllowAutomaticConfigurationOfListeners_1-300x268.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/AllowAutomaticConfigurationOfListeners_1.png)

```powershell
$session = New-PSSession -ComputerName "NameOfConnectionServer"
```

With remoting enabled, a session is opened to the connection server from a workstation. A session is a persistent connection that can be referenced when using subsequent remoting commands.

```powershell
Invoke-Command -Session $session -ScriptBlock {Add-PSSnapin VMware*}
```

The VMware cmdlets aren&#8217;t loaded by default, so `Invoke-Command` is used to tell the PowerShell session on the remote server to load the VMware snapin.

```powershell
Import-PSSession -Session $session -Prefix VDI -Module VMware*
```

The cmdlets are loaded in the server&#8217;s PowerShell session, but they must be imported in order to run them on the local workstation. The module parameter specifies to only import cmdlets from the VMware module, and the prefix parameter will prefix the noun in each cmdlet with &#8220;VDI&#8221; to avoid possible conflicts with local cmdlets. A cmdlet named Get-DesktopVM will become Get-VDIDesktopVM.

With the View cmdlets imported from the remote session, those commands can now be executed as if they were installed locally. PowerShell is doing all the work behind the scenes to _implicitly_ run the commands on the remote server and return the results to the local workstation.

[<img src="/assets/img/VMwareViewImplicitRemoting_1.png" alt="Using PowerShell Implicit Remoting to use VMware View Cmdlets" title="VMwareViewImplicitRemoting" width="640" height="518" class="alignnone size-full wp-image-1272" srcset="https://www.hofferle.com/wp-content/uploads/2012/02/VMwareViewImplicitRemoting_1.png 640w, https://www.hofferle.com/wp-content/uploads/2012/02/VMwareViewImplicitRemoting_1-300x242.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/VMwareViewImplicitRemoting_1.png)

```powershell
Remove-PSSession $session
```

The session to the connection server must remain open for the View cmdlets to be available. When finished working, removing the session closes it out and will unload the commands.