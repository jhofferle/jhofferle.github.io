---
id: 333
title: Getting computer serial numbers over the network
date: 2010-07-30T14:24:13+00:00
author: Jason Hofferle
#layout: post
guid: http://www.hofferle.com/?p=333
permalink: /getting-computer-serial-numbers-over-the-network/
categories:
  - VBScript
tags:
  - registry
  - VBScript
  - WMI
---
I hate inventory. Running around writing down serial numbers seems like something we shouldn’t have to do these days, but we still do. At least I do anyway.

This script will attach to a specified computer over the network and grab some information about it, including the serial number of the system and monitors attached to it. Kudos to Michael Baird and Denny Mansart for all that EDID code.

The big deal is the monitor serial numbers, because that is a tricky piece of information to get. Windows stores the monitor EDID information in the registry. This script grabs that data and then decodes it to get some information that is nice to have like the monitor’s serial number.

This is more of a demonstration script. I’ve used the code in other scripts that will poll subnets for computers and dump the info to a spreadsheet instead of just displaying it.

I used to have a link to the original code, but unfortunately Clarence Washington’s Win32Scripting site has closed it doors.

Microsoft introduced the WmiMonitorID class with Windows Vista and Server 2008. This class does the work of extracting monitor EDID data automatically. An example of using this WMI class with PowerShell is [here](https://www.hofferle.com/retrieve-monitor-serial-numbers-with-powershell/).

Sample info.vbs output:

Serial Number: USU4190DSD
  
Maximum Capacity: 4194304
  
Maximum Capacity: 512
  
Part Number:
  
Serial Number: USU4190DSD
  
Asset Tag: USU4190DSD

Monitor A)
  
&#8230;&#8230;&#8230;.VESA Manufacturer ID= HWP
  
&#8230;&#8230;&#8230;.Device ID= 2604
  
&#8230;&#8230;&#8230;.Manufacture Date= 7/2003
  
&#8230;&#8230;&#8230;.Serial Number= Serial Number Not Found in EDID data <1>
  
&#8230;&#8230;&#8230;.Model Name= hp 7550
  
&#8230;&#8230;&#8230;.EDID Version= 1.3
  
Monitor B)
  
&#8230;&#8230;&#8230;.VESA Manufacturer ID= VSC
  
&#8230;&#8230;&#8230;.Device ID= CE1B
  
&#8230;&#8230;&#8230;.Manufacture Date= 4/2005
  
&#8230;&#8230;&#8230;.Serial Number= PPJ051501908
  
&#8230;&#8230;&#8230;.Model Name= VA712b
  
&#8230;&#8230;&#8230;.EDID Version= 1.3

```vb
&#039;**************************************Heading*********************************
&#039; info.vbs
&#039;
&#039; Jason Hofferle
&#039; 08/16/2006 - version 0.1.1
&#039;
&#039; Monitor EDID code written by Michael Baird and modified by Denny MANSART
&#039;
&#039; Abstract:
&#039;           info.vbs - Command line utility that gathers hardware information
&#039;                      including serial numbers from a remote computer.
&#039;
&#039;
&#039; Usage:
&#039;           info computername
&#039;
&#039; Example:
&#039;           info CXAFLSTPAW101
&#039;           info ?
&#039;
&#039;******************************************************************************

&#039;Option Explicit

Dim objArgs
Dim strComputer

Const strGeneral_01 = "Abstract:"
Const strGeneral_02 = "             info.vbs - Command line utility that gathers"
Const strGeneral_03 = "                        hardware information including   "
Const strGeneral_04 = "                        serial numbers from a remote     "
Const strGeneral_05 = "                        computer.                        "
Const strGeneral_06 = ""
Const strGeneral_07 = "Jason Hofferle - jason.hofferle@dcma.mil"
Const strGeneral_08 = "6/17/2005 version 0.1.0"
Const strGeneral_09 = "Monitor EDID code written by Michael Baird and modified  "
Const strGeneral_10 = "by Denny MANSART                                         "

Const strUsage_01   = "Usage:"
Const strUsage_02   = "               info computername"
Const strUsage_03   = "Examples:"
Const strUsage_04   = "               info CXAFLSTPAW101"
Const strUsage_05   = "               info ?"

Const strError_ComputerNameNotFound = "Computer name not found, using local system."
Const strError_ConnectingToComputer = "Error connecting to specified computer."
Const strError_ComputerNotFound = "Specified computer could not be found on the network."
Const strError_AlreadyAdmin = "Specified user is already a member of the Administrators group."
Const strError_UserNotFound = "Specified user could not be found in the domain."
Const strError_AddingUser = "Error adding user to specified computer."
Const strError_CannotPing = "Cannot ping remote computer."

On Error Resume Next
Err.Clear
Set objArgs = WScript.Arguments
If objArgs.Count &gt; 0 Then
    strComputer = objArgs(0)
    if Not Pingable(strComputer) then
        Wscript.echo strError_CannotPing
        strComputer = "?"
    End If
Else
    Wscript.Echo strError_ComputerNameNotFound
    strComputer = "."
End If

If strComputer = "?" Then
    WScript.Echo strGeneral_01
    WScript.Echo strGeneral_02
    WScript.Echo strGeneral_03
    WScript.Echo strGeneral_04
    WScript.Echo strGeneral_05
    WScript.Echo strGeneral_06
    WScript.Echo strGeneral_07
    WScript.Echo strGeneral_08
    WScript.Echo strGeneral_09
    WScript.Echo strGeneral_10
    WScript.Echo ""
    WScript.Echo strUsage_01
    WScript.Echo strUsage_02
    WScript.Echo strUsage_03
    WScript.Echo strUsage_04
    WScript.Echo strUsage_05
    WScript.Quit
End If


winmgmt1 = "winmgmts:{impersonationLevel=impersonate}!//"& strComputer &""
Set SerialN = GetObject( winmgmt1 ).InstancesOf ("Win32_BIOS")

For each Serial in SerialN
WScript.Echo "Serial Number: " & Serial.SerialNumber
Next

Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" & strComputer & "\root\cimv2")

Set colItems = objWMIService.ExecQuery _
    ("Select * from Win32_PhysicalMemoryArray")

For Each objItem in colItems
    &#039;Wscript.Echo "Description: " & objItem.Description
    Wscript.Echo "Maximum Capacity: " & objItem.MaxCapacity
    &#039;Wscript.Echo "Memory Devices: " & objItem.MemoryDevices
    &#039;Wscript.Echo "Memory Error Correction: " & objItem.MemoryErrorCorrection
Next


Set objWMIService = GetObject("winmgmts:" _
    & "{impersonationLevel=impersonate}!\\" _
    & strComputer & "\root\cimv2")
Set colSMBIOS = objWMIService.ExecQuery _
    ("Select * from Win32_SystemEnclosure")
For Each objSMBIOS in colSMBIOS
    Wscript.Echo "Part Number: " & objSMBIOS.PartNumber
    WScript.Echo "Serial Number: " _
        & objSMBIOS.SerialNumber
    WScript.Echo "Asset Tag: " _
        & objSMBIOS.SMBIOSAssetTag
Next

WScript.Echo ""


&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;
&#039; Monitor EDID Information&#039;
&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;&#039;
&#039;17 June 2004
&#039;coded by Michael Baird
&#039;
&#039;Modified by Denny MANSART (27/07/2004)
&#039;
&#039;and released under the terms of GNU open source license agreement
&#039;(that is of course if you CAN release code that uses WMI under GNU)
&#039;
&#039;Please give me credit if you use my code

&#039;this code is based on the EEDID spec found at http://www.vesa.org
&#039;and by my hacking around in the windows registry
&#039;the code was tested on WINXP,WIN2K and WIN2K3
&#039;it should work on WINME and WIN98SE
&#039;It should work with multiple monitors, but that hasn&#039;t been tested either.

Dim oDisplaySubKeys : Set oDisplaySubKeys = CreateObject("Scripting.Dictionary")
Dim oRawEDID : Set oRawEDID = CreateObject("Scripting.Dictionary")
Const HKLM = &H80000002 &#039;HKEY_LOCAL_MACHINE

Int intMonitorCount=0
Int intDisplaySubKeysCount=0
Int i=0

&#039; Set WshNet=CreateObject("Wscript.Network")
&#039; MyComputerName=WshNet.ComputerName
&#039;
&#039; strComputer=MyComputerName
&#039;strComputer=wscript.arguments(0)

Set oRegistry = GetObject("winmgmts:{impersonationLevel=impersonate}!\\" & strComputer & 

"/root/default:StdRegProv")
strDisplayBaseKey = "SYSTEM\CurrentControlSet\Enum\DISPLAY\"

&#039; Retrieving EISA-Id from HKLM\SYSTEM\CurrentControlSet\Enum\DISPLAY and storing in strarrDisplaySubKeys
iRC = oRegistry.EnumKey(HKLM, strDisplayBaseKey, strarrDisplaySubKeys)

&#039; Deleting from strarrDisplaySubKeys "Default_Monitor" value
For Each sKey In strarrDisplaySubKeys

If sKey ="Default_Monitor" Then
intDisplaySubKeysCount=intDisplaySubKeysCount - 1
Else
oDisplaySubKeys.add sKey, intDisplaySubKeysCount
End If

intDisplaySubKeysCount=intDisplaySubKeysCount + 1

Next

&#039; Storing result in oDisplaySubKeys
strResultDisplaySubKeys=oDisplaySubKeys.Keys

toto=0

For i = 0 to oDisplaySubKeys.Count -1

strEisaIdBaseKey = strDisplayBaseKey & strResultDisplaySubKeys(i) & "\"

&#039; Retrieving Pnp-Id from HKLM\SYSTEM\CurrentControlSet\Enum\DISPLAY\EISA-Id and storing in 

strarrEisaIdSubKeys
iRC2 = oRegistry.EnumKey(HKLM, strEisaIdBaseKey, strarrEisaIdSubKeys)

For Each sKey2 In strarrEisaIdSubKeys
oRegistry.GetMultiStringValue HKLM, strEisaIdBaseKey & sKey2 & "\", "HardwareID", sValue

For tmpctr=0 to ubound(svalue)

If lcase(Left(svalue(tmpctr),8))="monitor\" then

strMsIdBaseKey = strEisaIdBaseKey & sKey2 & "\"

iRC3 = oRegistry.EnumKey(HKLM, strMsIdBaseKey, strarrMsIdSubKeys)

For Each sKey3 In strarrMsIdSubKeys

If skey3="Control" then

toto=toto + 1

oRegistry.GetBinaryValue HKLM, strMsIdBaseKey & "Device Parameters\", "EDID", intarrEDID
&#039; Test file
&#039;Dim objFileSystem, objOutputFile
&#039;Dim strOutputFile
&#039;Set objFileSystem = CreateObject("Scripting.fileSystemObject")
&#039;strOutputFile = "C:/temp/" & Split(WScript.ScriptName, ".")(0) & "-" & toto & ".txt"
&#039;Set objOutputFile = objFileSystem.CreateTextFile(strOutputFile, TRUE)
&#039;objOutputFile.WriteLine(strMsIdBaseKey & "Device Parameters\" & "EDID")

&#039; Reinitializing string to null to clear all datas
strRawEDID=""
strRawEDIDb=""

If vartype(intarrEDID) = 8204 then

For each strByteValue in intarrEDID

strRawEDID=strRawEDID & Chr(strByteValue)
strRawEDIDb=strRawEDIDb & Chr(strByteValue)
&#039; Test file
&#039;objOutputFile.WriteLine(strRawEDIDb)

Next

Else

strRawEDID="EDID Not Available"

End If
&#039; Test file
&#039;objOutputFile.WriteLine(strRawEDIDb)
&#039;objOutputFile.Close
&#039;Set objFileSystem = Nothing

oRawEDID.add intMonitorCount , strRawEDID
intMonitorCount=intMonitorCount + 1

End If
Next
End If
Next
Next
Next


&#039;*****************************************************************************************
&#039;now the EDID info For each active monitor is stored in an dictionnary of strings called oRawEDID
&#039;so we can process it to get the good stuff out of it which we will store in a 5 dimensional array
&#039;called arrMonitorInfo, the dimensions are as follows:
&#039;0=VESA Mfg ID, 1=VESA Device ID, 2=MFG Date (M/YYYY),3=Serial Num (If available),4=Model Descriptor
&#039;5=EDID Version
&#039;*****************************************************************************************

strResultRawEDID=oRawEDID.Keys

dim arrMonitorInfo()
redim arrMonitorInfo(intMonitorCount-1,5)
dim location(3)

&#039; Test file
&#039;Set objFileSystem = CreateObject("Scripting.fileSystemObject")
&#039;strOutputFile = "C:/temp/" & Split(WScript.ScriptName, ".")(0) & "-test.txt"
&#039;Set objOutputFile = objFileSystem.CreateTextFile(strOutputFile, TRUE)

For i=0 to oRawEDID.Count - 1

If oRawEDID(i) &lt;&gt; "EDID Not Available" then

&#039;*********************************************************************
&#039;first get the model and serial numbers from the vesa descriptor
&#039;blocks in the edid. the model number is required to be present
&#039;according to the spec. (v1.2 and beyond)but serial number is not
&#039;required. There are 4 descriptor blocks in edid at offset locations
&#039;&H36 &H48 &H5a and &H6c each block is 18 bytes long
&#039;*********************************************************************

location(0)=mid(oRawEDID(i),&H36+1,18)
location(1)=mid(oRawEDID(i),&H48+1,18)
location(2)=mid(oRawEDID(i),&H5a+1,18)
location(3)=mid(oRawEDID(i),&H6c+1,18)

&#039; Test file
&#039;objOutputFile.WriteLine("Location-0")
&#039;objOutputFile.WriteLine(location(0))
&#039;objOutputFile.WriteLine("Location-1")
&#039;objOutputFile.WriteLine(location(1))
&#039;objOutputFile.WriteLine("Location-2")
&#039;objOutputFile.WriteLine(location(2))
&#039;objOutputFile.WriteLine("Location-3")
&#039;objOutputFile.WriteLine(location(3))

&#039;you can tell If the location contains a serial number If it starts with &H00 00 00 ff
strSerFind=Chr(&H00) & Chr(&H00) & Chr(&H00) & Chr(&Hff)

&#039;or a model description If it starts with &H00 00 00 fc
strMdlFind=Chr(&H00) & Chr(&H00) & Chr(&H00) & Chr(&Hfc)

intSerFoundAt=-1
intMdlFoundAt=-1

For findit = 0 to 3
If instr(location(findit),strSerFind)&gt;0 then

intSerFoundAt=findit

End If

If instr(location(findit),strMdlFind)&gt;0 then

intMdlFoundAt=findit

End If

Next

&#039;If a location containing a serial number block was found then store it
If intSerFoundAt&lt;&gt;-1 then

tmp=Right(location(intSerFoundAt),14)

If instr(tmp,Chr(&H0a))&gt;0 then

tmpser=Trim(Left(tmp,instr(tmp,Chr(&H0a))-1))

Else

tmpser=Trim(tmp)

End If

&#039;although it is not part of the edid spec it seems as though the
&#039;serial number will frequently be preceeded by &H00, this
&#039;compensates For that
If Left(tmpser,1)=Chr(0) then tmpser=Right(tmpser,Len(tmpser)-1)

Else

tmpser="Serial Number Not Found in EDID data"

End If

&#039;If a location containing a model number block was found then store it
If intMdlFoundAt&lt;&gt;-1 then

tmp=Right(location(intMdlFoundAt),14)

If instr(tmp,Chr(&H0a))&gt;0 then

tmpmdl=Trim(Left(tmp,instr(tmp,Chr(&H0a))-1))

Else

tmpmdl=Trim(tmp)

End If

&#039;although it is not part of the edid spec it seems as though the
&#039;serial number will frequently be preceeded by &H00, this
&#039;compensates For that
If Left(tmpmdl,1)=Chr(0) then tmpmdl=Right(tmpmdl,Len(tmpmdl)-1)

Else

tmpmdl="Model Descriptor Not Found in EDID data"

End If

&#039;**************************************************************
&#039;Next get the mfg date
&#039;**************************************************************
&#039;the week of manufacture is stored at EDID offset &H10
tmpmfgweek=Asc(mid(oRawEDID(i),&H10+1,1))

&#039;the year of manufacture is stored at EDID offset &H11
&#039;and is the current year -1990
tmpmfgyear=(Asc(mid(oRawEDID(i),&H11+1,1)))+1990

&#039;store it in month/year format
tmpmdt=month(dateadd("ww",tmpmfgweek,DateValue("1/1/" & tmpmfgyear))) & "/" & tmpmfgyear

&#039;**************************************************************
&#039;Next get the edid version
&#039;**************************************************************
&#039;the version is at EDID offset &H12
tmpEDIDMajorVer=Asc(mid(oRawEDID(i),&H12+1,1))

&#039;the revision level is at EDID offset &H13
tmpEDIDRev=Asc(mid(oRawEDID(i),&H13+1,1))

&#039;store it in month/year format
If tmpEDIDMajorVer &lt; 255-48 and tmpEDIDRev &lt; 255-48 Then

tmpver=Chr(48+tmpEDIDMajorVer) & "." & Chr(48+tmpEDIDRev)

Else
tmpver="Not available"

End If

&#039;**************************************************************
&#039;Next get the mfg id
&#039;**************************************************************
&#039;the mfg id is 2 bytes starting at EDID offset &H08
&#039;the id is three characters long. using 5 bits to represent
&#039;each character. the bits are used so that 1=A 2=B etc..
&#039;
&#039;get the data
tmpEDIDMfg=mid(oRawEDID(i),&H08+1,2)

Char1=0 : Char2=0 : Char3=0

Byte1=Asc(Left(tmpEDIDMfg,1)) &#039;get the first half of the string
Byte2=Asc(Right(tmpEDIDMfg,1)) &#039;get the first half of the string

&#039;now shift the bits
&#039;shift the 64 bit to the 16 bit
If (Byte1 and 64) &gt; 0 then Char1=Char1+16

&#039;shift the 32 bit to the 8 bit
If (Byte1 and 32) &gt; 0 then Char1=Char1+8

&#039;etc....
If (Byte1 and 16) &gt; 0 then Char1=Char1+4
If (Byte1 and 8) &gt; 0 then Char1=Char1+2
If (Byte1 and 4) &gt; 0 then Char1=Char1+1

&#039;the 2nd character uses the 2 bit and the 1 bit of the 1st byte
If (Byte1 and 2) &gt; 0 then Char2=Char2+16
If (Byte1 and 1) &gt; 0 then Char2=Char2+8

&#039;and the 128,64 and 32 bits of the 2nd byte
If (Byte2 and 128) &gt; 0 then Char2=Char2+4
If (Byte2 and 64) &gt; 0 then Char2=Char2+2
If (Byte2 and 32) &gt; 0 then Char2=Char2+1

&#039;the bits For the 3rd character don&#039;t need shifting
&#039;we can use them as they are
Char3=Char3+(Byte2 and 16)
Char3=Char3+(Byte2 and 8)
Char3=Char3+(Byte2 and 4)
Char3=Char3+(Byte2 and 2)
Char3=Char3+(Byte2 and 1)

tmpmfg=Chr(Char1+64) & Chr(Char2+64) & Chr(Char3+64)

&#039;**************************************************************
&#039;Next get the device id
&#039;**************************************************************
&#039;the device id is 2bytes starting at EDID offset &H0a
&#039;the bytes are in reverse order.
&#039;this code is not text. it is just a 2 byte code assigned
&#039;by the manufacturer. they should be unique to a model
tmpEDIDDev1=hex(Asc(mid(oRawEDID(i),&H0a+1,1)))
tmpEDIDDev2=hex(Asc(mid(oRawEDID(i),&H0b+1,1)))

If Len(tmpEDIDDev1)=1 then tmpEDIDDev1="0" & tmpEDIDDev1
If Len(tmpEDIDDev2)=1 then tmpEDIDDev2="0" & tmpEDIDDev2

tmpdev=tmpEDIDDev2 & tmpEDIDDev1

&#039;**************************************************************
&#039;finally store all the values into the array
&#039;**************************************************************
arrMonitorInfo(i,0)=tmpmfg
arrMonitorInfo(i,1)=tmpdev
arrMonitorInfo(i,2)=tmpmdt
arrMonitorInfo(i,3)=tmpser
arrMonitorInfo(i,4)=tmpmdl
arrMonitorInfo(i,5)=tmpver
End If

wscript.echo "Monitor " & Chr(i+65) & ")"
wscript.echo ".........." & "VESA Manufacturer ID= " & arrMonitorInfo(i,0)
wscript.echo ".........." & "Device ID= " & arrMonitorInfo(i,1)
wscript.echo ".........." & "Manufacture Date= " & arrMonitorInfo(i,2)
wscript.echo ".........." & "Serial Number= " & arrMonitorInfo(i,3)
wscript.echo ".........." & "Model Name= " & arrMonitorInfo(i,4)
wscript.echo ".........." & "EDID Version= " & arrMonitorInfo(i,5)


Next

&#039; Test file
&#039;objOutputFile.Close
&#039;Set objFileSystem = NothingART


Function ErrorCheck(strErrorNumber)
    Select Case strErrorNumber
        Case "-2147024843"
            ErrorCheck =  strError_ComputerNotFound
        Case "-2147023518"
            ErrorCheck = strError_AlreadyAdmin
        Case "-2147023509"
            ErrorCheck = strError_UserNotFound
    End Select
End Function

Function Pingable(strComputer)
Dim objShell
Dim strTemp
Dim objFSO
Dim iReturn
Dim objTextFile
Pingable = False
Set objShell = CreateObject("WScript.Shell")
strTemp = objShell.ExpandEnvironmentStrings("%temp%") & _
 "/tempping.txt"
iReturn = objShell.Run("%comspec% /C ping " & strComputer & _
         " -n 1 &gt; " & strTemp, 0, True)
Set objFSO = CreateObject("Scripting.FileSystemObject")
Set objTextFile = objFSO.OpenTextFile(strTemp, 1)
While Not objTextFile.AtEndOfStream
    If InStr(objTextFile.ReadLine, "Reply") Then Pingable = True
Wend
objTextFile.Close
objFSO.DeleteFile (strTemp)

End Function
```