---
id: 298
title: Get-Geocode
date: 2011-04-06T22:53:06+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=298
permalink: /get-geocode/
categories:
  - PowerShell
tags:
  - Google
  - PowerShell
---
```powershell
Function Get-Geocode
{
    [CmdletBinding()]
    Param
    (
        [Parameter(Mandatory=$True)]
        [String]$Address,
        
        [ValidateSet("xml", "json")]
        [String]$Output = "xml",
        
        [String]$Region = "",
        
        [String]$Language = "",
        
        [String]$Proxy
    )
 
    Begin
    {
        $webClient = New-Object System.Net.WebClient
        $url = "http://maps.googleapis.com/maps/api/geocode/$($Output.ToLower())?address="
        $sensor = "&sensor=false"
        if ($Region -ne "") { $Region = "&region=$Region" }
        if ($Language -ne "") { $Language = "&language=$Language" }
        if ($Proxy)
        {
            Write-Debug $Proxy
            $webProxy = New-Object System.Net.WebProxy $Proxy
            $webProxy.UseDefaultCredentials
            $webClient.Proxy = $webProxy
        }
    }
 
    Process
    { 
        $queryString = $url + `
                       ($Address -replace ' ','+' -join '+') + `
                       $sensor + `
                       $Region + `
                       $Language
                       
        Write-Debug $queryString
 
        $returnValue = $webClient.DownloadString($queryString)
 
        Write-Output $returnValue
    }
    
    End
    {
    }
    
    &lt;#
      .Synopsis
      Uses the Google Geocoding API V3 to return geocoding 
      information for the specified address.
      
      .Description
      The Get-Geocode function uses the Google Geocoding API V3 to return 
      geocoding information for the specified address.
      
      Note that Google's service license restrictions require this service be 
      used in conjunction with a Google map. This code is for demonstration 
      purposes only.
       
      .parameter Address
      The address you want to geocode. This parameter is mandatory.

      .parameter Output
      Specifies the format of the data returned by the API. Valid values are
      xml and json (JavaScript Object Notation). The default value is xml.

      .parameter Region
      The two-character ccTLD region code. This returns results biased for a
      particular region. The default is to influence results based on the
      region from which the request is sent.

      .parameter Language
      The language in which to return results. The default is to use the native
      language from the domain from which the request is sent. A supported list
      of languages can be found here in the Google Maps API FAQ.
      http://code.google.com/apis/maps/faq.html#languagesupport
      
      .parameter Proxy
      The address of the web proxy to use for the http request. The credentials of
      the currently logged-on user are used. The default is to not use a proxy.

      .Example
      Get-Geocode "1600 Amphitheatre Parkway Mountain View CA 94043"

      Description
      -----------
      This command converts the specified address to geographic coordinates and
      returns the information in xml format.

      .Link
      http://code.google.com/apis/maps/documentation/geocoding/
      
      .Notes
      NAME:     Get-Geocode
      AUTHOR:   Jason Hofferle
      LASTEDIT: 4/6/2011
    #&gt;
}
```