---
title: Send Email with Attachment using VBScript
date: 2010-06-19T14:28:56+00:00
author: Jason Hofferle
permalink: /send-email-with-attachment-using-vbscript/
categories:
  - VBScript
tags:
  - email
  - VBScript
---
This is a way to send email with an attachment using an SMTP server that requires authentication.

```vb
Set email = CreateObject("CDO.Message")

email.Subject = "Test Email"
email.From = "me@mydomain.com"
email.To = "recipient@theirdomain.com"
email.TextBody = "Message Text."
email.AddAttachment "c:\document.txt"

email.Configuration.Fields.Item("http://schemas.microsoft.com/cdo/configuration/sendusername") = "UserName"
email.Configuration.Fields.Item("http://schemas.microsoft.com/cdo/configuration/sendpassword") = "PassWord"


email.Configuration.Fields.Item("http://schemas.microsoft.com/cdo/configuration/sendusing")=2
email.Configuration.Fields.Item("http://schemas.microsoft.com/cdo/configuration/smtpserver")="smtp.yourserver.com"
email.Configuration.Fields.Item("http://schemas.microsoft.com/cdo/configuration/smtpserverport")=25

email.Configuration.Fields.Update
email.Send
set email = Nothing
```
