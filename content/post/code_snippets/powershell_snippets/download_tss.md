---
title: 'Download TSS'
summary: 'Download TSS'
date: '2024-07-15T00:00:00+00:00'
---

## PowerShell: Script to download TSS toolkit

```powershell
$url = "https://aka.ms/getTSS"
$outputPath = "$env:USERPROFILE\Downloads\TSS.zip"

Invoke-WebRequest -Uri $url -OutFile $outputPath

if (Test-Path $outputPath) {
    Write-Host "File successfully downloaded and saved to: $outputPath"
} else {
    Write-Host "Failed to download the file."
}
```
