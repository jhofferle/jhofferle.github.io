---
title: Import-Csv and Headers in PowerShell
date: 2015-02-14T11:45:43+00:00
author: Jason Hofferle
permalink: /import-csv-and-headers-in-powershell/
categories:
  - PowerShell
tags:
  - PowerShell
---
The Import-Csv cmdlet in PowerShell is awesome. It saves a lot of time and provides a great integration point since the csv format is a common option for exporting data in many applications. However, all csv files are not created equal and there are a couple problems I've run across often enough that I had to write about it.

A common scenario is generating a csv with some system or tool, then importing that data to process it with PowerShell. Many csv files will look relatively fine when opened in Excel.

![image-center](/assets/img/ExcelDataCsv.png)

Importing the csv with PowerShell seems to work fine, and Get-Member shows the expected property was generated from the header. However, when we try to access this property we're not getting the expected results.

![image-center](/assets/img/ImportDataCsv.png)

Taking a closer look at the csv shows that the header is actually padded with spaces.

![image-center](/assets/img/CsvWithPadding.png)

This results in the property name of our file not being "RelativeUrl" but instead "RelativeUrl" plus a space. Trying to access our data by typing out "R-e-l-a-t-i-v-e-U-r-l" doesn't return anything, but if we use tab completion and type "r-e-l"+Tab the autocomplete feature automatically wraps the property in quotes because of the space.

![image-center](/assets/img/PropertyNameSpaces.png)

At this point, we can manually remove the spaces from the file (yuck), remember to wrap the property with quotes and add the space every time with access it (meh), or utilize the header parameter of Import-Csv to specify our own header.

![image-center](/assets/img/CsvImportWithHeaderParameter.png)

Now the supplied header name becomes the property name, and we can access our data as expected. The header parameter takes an array of strings, so if the csv has multiple columns of data, multiple header names can be specified.

But now we have another problem. Our header might be correct, but now Import-Csv is treating the original header row in the file as legitimate data. Plus, the application that generated the csv added a row of dashes, probably in misguided attempt to make the file "pretty." At some point this is where many admins will break down and manually massage the csv so they can get a clean import. This might be ok if you're only doing this once, but if we're building an automated routine we don't any manual steps.

![image-center](/assets/img/ConvertFromCsv.png)

Here we take a completely different approach and skip using the Import-Csv cmdlet altogether. This allows us to utilize Select-Object to skip the first couple lines of garbage we don't want, and we're center with clean data.

This is nothing mind-blowing or innovative, but at this point I've seen so many people manually editing csv files to deal with spaces in headers and junk formatting that I figured somebody might benefit from a quick post.
