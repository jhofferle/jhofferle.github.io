---
id: 1464
title: Import-Csv and Headers in PowerShell
date: 2015-02-14T11:45:43+00:00
author: Jason Hofferle
layout: post
guid: http://www.hofferle.com/?p=1464
permalink: /import-csv-and-headers-in-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
---
The Import-Csv cmdlet in PowerShell is awesome. It saves a lot of time and provides a great integration point since the csv format is a common option for exporting data in many applications. However, all csv files are not created equal and there are a couple problems I&#8217;ve run across often enough that I had to write about it.

A common scenario is generating a csv with some system or tool, then importing that data to process it with PowerShell. Many csv files will look relatively fine when opened in Excel.

[<img src="/assets/img/ExcelDataCsv.png" alt="CSV File in Excel" width="640" height="548" class="alignleft size-full wp-image-1465" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/ExcelDataCsv.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/ExcelDataCsv-150x128.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/ExcelDataCsv-300x257.png 300w, https://www.hofferle.com/wp-content/uploads/2015/02/ExcelDataCsv-561x480.png 561w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/ExcelDataCsv.png)

Importing the csv with PowerShell seems to work fine, and Get-Member shows the expected property was generated from the header. However, when we try to access this property we&#8217;re not getting the expected results.

[<img src="/assets/img/ImportDataCsv.png" alt="Importing CSV with PowerShell" width="640" height="434" class="alignleft size-full wp-image-1466" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/ImportDataCsv.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/ImportDataCsv-150x102.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/ImportDataCsv-300x203.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/ImportDataCsv.png)

Taking a closer look at the csv shows that the header is actually padded with spaces.

[<img src="/assets/img/CsvWithPadding.png" alt="CSV file with padded data" width="640" height="511" class="alignleft size-full wp-image-1467" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/CsvWithPadding.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/CsvWithPadding-150x120.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/CsvWithPadding-300x240.png 300w, https://www.hofferle.com/wp-content/uploads/2015/02/CsvWithPadding-601x480.png 601w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/CsvWithPadding.png)

This results in the property name of our file not being &#8220;RelativeUrl&#8221; but instead &#8220;RelativeUrl&#8221; plus a space. Trying to access our data by typing out &#8220;R-e-l-a-t-i-v-e-U-r-l&#8221; doesn&#8217;t return anything, but if we use tab completion and type &#8220;r-e-l&#8221;+Tab the autocomplete feature automatically wraps the property in quotes because of the space.

[<img src="/assets/img/PropertyNameSpaces.png" alt="Property name with spaces" width="640" height="191" class="alignleft size-full wp-image-1468" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/PropertyNameSpaces.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/PropertyNameSpaces-150x45.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/PropertyNameSpaces-300x90.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/PropertyNameSpaces.png)

At this point, we can manually remove the spaces from the file (yuck), remember to wrap the property with quotes and add the space every time with access it (meh), or utilize the header parameter of Import-Csv to specify our own header.

[<img src="/assets/img/CsvImportWithHeaderParameter.png" alt="Importing CSV with header parameter" width="640" height="378" class="alignleft size-full wp-image-1469" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/CsvImportWithHeaderParameter.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/CsvImportWithHeaderParameter-150x89.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/CsvImportWithHeaderParameter-300x177.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/CsvImportWithHeaderParameter.png)

Now the supplied header name becomes the property name, and we can access our data as expected. The header parameter takes an array of strings, so if the csv has multiple columns of data, multiple header names can be specified.

But now we have another problem. Our header might be correct, but now Import-Csv is treating the original header row in the file as legitimate data. Plus, the application that generated the csv added a row of dashes, probably in misguided attempt to make the file &#8220;pretty.&#8221; At some point this is where many admins will break down and manually massage the csv so they can get a clean import. This might be ok if you&#8217;re only doing this once, but if we&#8217;re building an automated routine we don&#8217;t any manual steps.

[<img src="/assets/img/ConvertFromCsv.png" alt="Convert From Csv" width="640" height="220" class="alignleft size-full wp-image-1470" srcset="https://www.hofferle.com/wp-content/uploads/2015/02/ConvertFromCsv.png 640w, https://www.hofferle.com/wp-content/uploads/2015/02/ConvertFromCsv-150x52.png 150w, https://www.hofferle.com/wp-content/uploads/2015/02/ConvertFromCsv-300x103.png 300w" sizes="(max-width: 640px) 100vw, 640px" />](/assets/img/ConvertFromCsv.png)

Here we take a completely different approach and skip using the Import-Csv cmdlet altogether. This allows us to utilize Select-Object to skip the first couple lines of garbage we don&#8217;t want, and we&#8217;re left with clean data.

This is nothing mind-blowing or innovative, but at this point I&#8217;ve seen so many people manually editing csv files to deal with spaces in headers and junk formatting that I figured somebody might benefit from a quick post.