---
id: 58
title: 'PowerShell: Check if Accelerated Networking Adapter is present within Windows VM'
summary: 'PowerShell: Check if Accelerated Networking Adapter is present within Windows VM'
date: '2023-08-22T13:33:40+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=58'
permalink: /2023/08/22/powershell-check-if-accelerated-networking-adapter-is-present-within-windows-vm/
categories:
    - Code
---
## PowerShell: Check if Accelerated Networking Adapter is present within Windows VM

```powershell
# You might want to check extra adapters type/names here
$AdapterName = "Mellanox ConnectX-4 Lx Virtual Ethernet Adapter"

$AdapterExists = Get-NetAdapter | Where-Object { $_.InterfaceDescription -match $AdapterName }

if ($AdapterExists) {
    Write-Host "The '$AdapterName' network adapter exists in the system." -ForegroundColor Green
} else {
    Write-Host "The '$AdapterName' network adapter does not exist in the system." -ForegroundColor Red
}
```
