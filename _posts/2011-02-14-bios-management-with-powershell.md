---
id: 150
title: BIOS Management with PowerShell
date: 2011-02-14T16:14:49+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=150
permalink: /bios-management-with-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
  - WMI
---
Hewlett-Packard&#8217;s [Client Management Interface](http://h20331.www2.hp.com/Hpsub/cache/284014-0-0-225-121.html) and Dell&#8217;s [OpenManage Client Instrumentation](http://www.delltechcenter.com/page/OpenManage+Client+Instrumentation+(OMCI)) allow their hardware to be managed through various enterprise management tools. After installing the CMI or OMCI client, the BIOS on these computers can be accessed using Windows Management Instrumentation.

**HP Boot Order**

<pre class="lang:powershell decode:true">$bios = Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class hp_biosSetting
($bios | Where-Object {$_.Name -eq &#039;Boot Order&#039;}).Value.Split(&#039;,&#039;)
</pre>

**Dell Boot Order**

<pre class="lang:powershell decode:true">Get-WmiObject -Namespace root/dellOMCI -Class Dell_BootDeviceSequence |
Select BootDeviceName, BootOrder, Status |
Sort-Object BootOrder |
Format-Table -AutoSize
</pre>

[<img class="alignnone size-medium wp-image-167" title="Dell_BootOrder" src="https://hofferle.com/wordpress/wp-content/uploads/2011/02/Dell_BootOrder3-300x110.png" alt="Dell BootOrder" width="300" height="110" srcset="https://www.hofferle.com/wp-content/uploads/2011/02/Dell_BootOrder3-300x110.png 300w, https://www.hofferle.com/wp-content/uploads/2011/02/Dell_BootOrder3.png 868w" sizes="(max-width: 300px) 100vw, 300px" />](http://hofferle.com/wordpress/wp-content/uploads/2011/02/Dell_BootOrder3.png)

**HP BIOS Settings**

<pre class="lang:powershell decode:true">Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class hp_biosEnumeration |
Format-Table Name,Value -AutoSize
</pre>

**Modifying HP Setting**

<pre class="lang:powershell decode:true">$bios = Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class HP_BIOSSettingInterface
$bios.SetBIOSSetting(&#039;After Power Loss&#039;, &#039;On&#039;)
</pre>

[<img class="alignnone size-medium wp-image-171" title="HP_SetAfterPowerLoss" src="https://hofferle.com/wordpress/wp-content/uploads/2011/02/HP_SetAfterPowerLoss-300x170.png" alt="HP Set After Power Loss" width="300" height="170" srcset="https://www.hofferle.com/wp-content/uploads/2011/02/HP_SetAfterPowerLoss-300x170.png 300w, https://www.hofferle.com/wp-content/uploads/2011/02/HP_SetAfterPowerLoss.png 988w" sizes="(max-width: 300px) 100vw, 300px" />](http://hofferle.com/wordpress/wp-content/uploads/2011/02/HP_SetAfterPowerLoss.png)