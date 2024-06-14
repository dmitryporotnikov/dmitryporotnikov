---

id: 261
title: 'PowerShell: Gather DNS Server Logs from Windows'
summary: 'PowerShell: Gather DNS Server Logs from Windows'
date: '2023-09-13T14:25:29+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=261'
permalink: /2023/09/13/powershell-gather-dns-server-logs-from-windows/
categories:
    - Code
    - Windows

---
## PowerShell: Gather DNS Server Logs from Windows

When maintaining a Windows DNS server, pulling the DNS logs periodically is essential for monitoring and troubleshooting. PowerShell offers a powerful yet simple method to automate this. Below is a script that not only fetches the DNS server logs but also gathers detailed DNS configurations using native commands.

Here’s how you can use PowerShell to gather DNS server logs from a Windows server:

```powershell
$destDirectory = "C:\DNSLogs"

if (-not (Test-Path $destDirectory)) {
    New-Item -ItemType Directory -Path $destDirectory
}

Get-WinEvent -LogName 'DNS Server' | Export-Csv -Path "$destDirectory\DNSServerEvents.csv" -NoTypeInformation

dnscmd /info > "$destDirectory\DNSConfig.txt"

Write-Output "Diagnostic logs have been gathered and saved to $destDirectory"
```
