---
id: 790
title: Enable RSAT Features with PowerShell and Dism
date: 2012-01-03T09:00:53+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=790
permalink: /enable-rsat-features-with-powershell-and-dism/
categories:
  - PowerShell
tags:
  - PowerShell
---
Checking boxes gets old quickly, and I&#8217;ve installed and checked those little boxes for the Remote Server Administration Tools (RSAT) on Windows 7 enough times for it to be annoying. I just wanted a static script with the names of the features to enable, but [Xenophane&#8217;s Blog](http://www.xipher.dk/WordPress/?p=338 "Xenophane's Blog") has some examples of extracting the output of the /Get-Features switch to dynamically generate the command to enable everything in RSAT.

[Enable-RSATFeatures.zip](/assets/img/Enable-RSATFeatures.zip)

```powershell
dism /Online /Enable-Feature `
/FeatureName:RemoteServerAdministrationTools `
/FeatureName:RemoteServerAdministrationTools-ServerManager `
/FeatureName:RemoteServerAdministrationTools-Roles `
/FeatureName:RemoteServerAdministrationTools-Roles-CertificateServices `
/FeatureName:RemoteServerAdministrationTools-Roles-CertificateServices-CA `
/FeatureName:RemoteServerAdministrationTools-Roles-CertificateServices-OnlineResponder `
/FeatureName:RemoteServerAdministrationTools-Roles-AD `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-DS `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-DS-SnapIns `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-DS-AdministrativeCenter `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-DS-NIS `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-LDS `
/FeatureName:RemoteServerAdministrationTools-Roles-AD-Powershell `
/FeatureName:RemoteServerAdministrationTools-Roles-DHCP `
/FeatureName:RemoteServerAdministrationTools-Roles-DNS `
/FeatureName:RemoteServerAdministrationTools-Roles-FileServices `
/FeatureName:RemoteServerAdministrationTools-Roles-FileServices-Dfs `
/FeatureName:RemoteServerAdministrationTools-Roles-FileServices-Fsrm `
/FeatureName:RemoteServerAdministrationTools-Roles-FileServices-StorageMgmt `
/FeatureName:RemoteServerAdministrationTools-Roles-HyperV `
/FeatureName:RemoteServerAdministrationTools-Roles-RDS `
/FeatureName:RemoteServerAdministrationTools-Features `
/FeatureName:RemoteServerAdministrationTools-Features-BitLocker `
/FeatureName:RemoteServerAdministrationTools-Features-Clustering `
/FeatureName:RemoteServerAdministrationTools-Features-GP `
/FeatureName:RemoteServerAdministrationTools-Features-LoadBalancing `
/FeatureName:RemoteServerAdministrationTools-Features-StorageExplorer `
/FeatureName:RemoteServerAdministrationTools-Features-StorageManager `
/FeatureName:RemoteServerAdministrationTools-Features-Wsrm
```