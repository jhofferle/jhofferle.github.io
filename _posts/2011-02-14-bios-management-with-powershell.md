---
title: BIOS Management with PowerShell
date: 2011-02-14T16:14:49+00:00
author: Jason Hofferle
permalink: /bios-management-with-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
  - WMI
---
Hewlett-Packard's [Client Management Interface](https://www8.hp.com/us/en/ads/clientmanagement/overview.html) and Dell's [OpenManage Client Instrumentation](http://www.delltechcenter.com/page/OpenManage+Client+Instrumentation+(OMCI)) allow their hardware to be managed through various enterprise management tools. After installing the CMI or OMCI client, the BIOS on these computers can be accessed using Windows Management Instrumentation.

**HP Boot Order**

```powershell
$bios = Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class hp_biosSetting
($bios | Where-Object {$_.Name -eq 'Boot Order'}).Value.Split(',')
```

**Dell Boot Order**

```powershell
Get-WmiObject -Namespace root/dellOMCI -Class Dell_BootDeviceSequence |
Select BootDeviceName, BootOrder, Status |
Sort-Object BootOrder |
Format-Table -AutoSize
```

![image-center](http://hofferle.com/wordpress/wp-content/uploads/2011/02/Dell_BootOrder3.png)

**HP BIOS Settings**

```powershell
Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class hp_biosEnumeration |
Format-Table Name,Value -AutoSize
```

**Modifying HP Setting**

```powershell
$bios = Get-WmiObject -Namespace root/hp/instrumentedBIOS -Class HP_BIOSSettingInterface
$bios.SetBIOSSetting('After Power Loss', 'On')
```

![image-center](http://hofferle.com/wordpress/wp-content/uploads/2011/02/HP_SetAfterPowerLoss.png)
