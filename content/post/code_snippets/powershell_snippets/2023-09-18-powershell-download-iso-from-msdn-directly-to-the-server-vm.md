---
id: 371
title: 'PowerShell: Download ISO from MSDN Directly to the Server/VM'
summary: 'PowerShell: Download ISO from MSDN Directly to the Server/VM'
date: '2023-09-18T09:22:23+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=371'
permalink: /2023/09/18/powershell-download-iso-from-msdn-directly-to-the-server-vm/
categories:
    - Code
---
## PowerShell: Download ISO from MSDN Directly to the Server/VM'

```powershell
# Define the URL of the ISO file
$isoUrl = "https://myvs.download.prss.microsoft.com/yourisofilepath.iso"

# Define the destination path where the ISO file will be saved
$destinationPath = "C:\Temp\WS2012.iso"

# Download the ISO file
Invoke-WebRequest -Uri $isoUrl -OutFile $destinationPath

# Output the location where the ISO file has been saved
Write-Host "ISO file has been downloaded to $destinationPath
```
