---
id: 64
title: 'PowerShell: Download and run Sysinternals PSPing to test network connectivity over TCP'
summary: 'PowerShell: Download and run Sysinternals PSPing to test network connectivity over TCP'
date: '2023-08-22T13:45:53+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=64'
permalink: /2023/08/22/powershell-download-and-run-sysinternals-psping-to-test-network-connectivity-over-tcp/
categories:
    - Code
---
## PowerShell: Download and run Sysinternals PSPing to test network connectivity over TCP

```powershell
# Define the URL and the output file path for PsPing
$pspingURL = "https://live.sysinternals.com/PsPing.exe"
$pspingFilePath = "C:\Temp\PsPing.exe"

# Create the output directory if it doesn't exist
If (!(Test-Path -Path (Split-Path -Path $pspingFilePath))) {
    New-Item -ItemType Directory -Path (Split-Path -Path $pspingFilePath) -Force
}

# Download PsPing
Invoke-WebRequest -Uri $pspingURL -OutFile $pspingFilePath

# Test the connection using PsPing with the -accepteula flag
& $pspingFilePath -accepteula "8.8.8.8:443"
```
