---
id: 68
title: 'PowerShell: Send telegram message via Bot using RestAPI and Bot Token'
summary: 'PowerShell: Send telegram message via Bot using RestAPI and Bot Token'
date: '2023-08-22T13:52:35+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=68'
permalink: /2023/08/22/powershell-send-telegram-message-via-bot-using-restapi-and-bot-token/
categories:
    - Code
---
## PowerShell: Send telegram message via Bot using RestAPI and Bot Token

```powershell
$text_to_send = "*" + $VM_Name + "*" + " " + "Deployed by: " + "*" + $env:UserName + "*" + " " + "In RG: " 

$token = "BOT TOKEN"
$preview_mode = "True"
$markdown_mode = "Markdown"

$payload = @{
	"chat_id"					   = "-0000000";
	"text"						   = $text_to_send;
	"parse_mode"				   = $markdown_mode;
	"disable_web_page_preview"	   = $preview_mode;
}
try
{
	$notify = Invoke-WebRequest `
								-Uri ("https://api.telegram.org/bot{0}/sendMessage" -f $token) `
								-Method Post `
								-ContentType "application/json;charset=utf-8" `
								-Body (ConvertTo-Json -Compress -InputObject $payload)
}
catch
{
}
```
