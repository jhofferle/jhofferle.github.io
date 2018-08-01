---
title: Exporting Exchange Global Address List to Excel with VBScript
date: 2010-06-28T14:11:01+00:00
author: Jason Hofferle
permalink: /exporting-exchange-global-address-list-to-excel-with-vbscript/
categories:
  - VBScript
tags:
  - Active Directory
  - VBScript
---
This vbscript exports an Exchange GAL to an Excel spreadsheet. The important line to change is objCommand.CommandText which must be modified to reflect your Active Directory domain and the address book field you want to search for. Any information about the user object can be retrieved by additional modification of the script.

An example of [querying Active Directory](http://www.hofferle.com/query-active-directory-with-powershell/) using PowerShell custom objects is also available.

```vb
'******************************************************************************
'GAL_Export.vbs
'
'Exports Exchange Global Address List to Excel
'
'Jason Hofferle
'21 June 2007
'
'Be sure to modify the objCommand.CommandText line to reflect your domain and fields
'******************************************************************************

Set objConnection = CreateObject("ADODB.Connection")
Set objCommand =   CreateObject("ADODB.Command")
objConnection.Provider = "ADsDSOObject"
objConnection.Open "Active Directory Provider"
Set objCommand.ActiveConnection = objConnection

objCommand.Properties("Page Size") = 1000

objCommand.CommandText = "Select department, l, title, telephonenumber, givenname, sn, initials, displayname, name," _
& "physicalDeliveryOfficeName, streetAddress, st, postalCode, c, company FROM 'LDAP://dc=your,dc=domain,dc=here,dc=com'" _
& "WHERE objectCategory='user' AND company='Your Company Field'"

Set objRecordSet = objCommand.Execute
objRecordSet.MoveFirst

Set ObjExcel = CreateObject("Excel.Application")
objExcel.Visible = True
objExcel.Workbooks.Add

objExcel.Cells(1, 1).Value = "Last"
objExcel.Cells(1, 2).Value = "First"
objExcel.Cells(1, 3).Value = "Initials"
objExcel.Cells(1, 4).Value = "Company"
objExcel.Cells(1, 5).Value = "Office"
objExcel.Cells(1, 6).Value = "Address"
objExcel.Cells(1, 7).Value = "City"
objExcel.Cells(1, 8).Value = "State"
objExcel.Cells(1, 9).Value = "Zip code"
objExcel.Cells(1, 10).Value = "Country"
objExcel.Cells(1, 11).Value = "Phone"
objExcel.Cells(1, 12).Value = "Title"
objExcel.Cells(1, 13).Value = "Department"


i = 2
Do Until objRecordSet.EOF
    Wscript.Echo objRecordSet.Fields("Name").Value
    objExcel.Cells(i, 1).Value = objRecordSet.Fields("sn").Value
    objExcel.Cells(i, 2).Value = objRecordSet.Fields("givenname").Value
    objExcel.Cells(i, 3).Value = objRecordSet.Fields("initials").Value
    objExcel.Cells(i, 4).Value = objRecordSet.Fields("company").Value
    objExcel.Cells(i, 5).Value = objRecordSet.Fields("physicalDeliveryOfficeName").Value
    objExcel.Cells(i, 6).Value = objRecordSet.Fields("streetAddress").Value
    objExcel.Cells(i, 7).Value = ObjRecordSet.Fields("l").Value
    objExcel.Cells(i, 8).Value = objRecordSet.Fields("st").Value
    objExcel.Cells(i, 9).Value = objRecordSet.Fields("postalCode").Value
    objExcel.Cells(i, 10).Value = objRecordSet.Fields("c").Value
    objExcel.Cells(i, 11).Value = objRecordset.Fields("telephoneNumber").Value
    objExcel.Cells(i, 12).Value = objRecordset.Fields("title").Value
    objExcel.Cells(i, 13).Value = objRecordset.Fields("department").Value

    i = i + 1
    objRecordSet.MoveNext
Loop

wscript.echo objRecordset.recordcount & " contacts found."
```

To use, modify the following line:

```vb
objCommand.CommandText = "Select department, l, title, telephonenumber, givenname, sn, initials, displayname, name," _
& "physicalDeliveryOfficeName, streetAddress, st, postalCode, c, company FROM 'LDAP://dc=your,dc=domain,dc=here,dc=com'" _
& "WHERE objectCategory='user' AND company='Your Company Field'"
```

For example, if your Active Directory domain is internal.microsoft.com and you want a spreadsheet of everyone that had Operations for the company field, the line would be modified to this:

```vb
objCommand.CommandText = "Select department, l, title, telephonenumber, givenname, sn, initials, displayname, name," _
& "physicalDeliveryOfficeName, streetAddress, st, postalCode, c, company FROM 'LDAP://dc=internal,dc=microsoft,dc=com'" _
& "WHERE objectCategory='user' AND company='Operations'"
```

If you wanted an Excel spreadsheet of everyone with Vice President in the title field, it should be changed to this:

```vb
objCommand.CommandText = "Select department, l, title, telephonenumber, givenname, sn, initials, displayname,name," _
& "physicalDeliveryOfficeName, streetAddress, st, postalCode, c, company FROM 'LDAP://dc=internal,dc=microsoft,dc=com'" _
& "WHERE objectCategory='user' AND title='Vice President'"
```
