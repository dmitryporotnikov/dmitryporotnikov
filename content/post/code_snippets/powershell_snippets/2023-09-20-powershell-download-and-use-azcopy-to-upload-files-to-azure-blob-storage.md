---
id: 379
title: 'PowerShell: Download and Use AzCopy (Windows) to Upload Files to Azure Blob Storage'
summary: 'PowerShell: Download and Use AzCopy (Windows) to Upload Files to Azure Blob Storage'
date: '2023-09-20T15:22:09+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=379'
permalink: /2023/09/20/powershell-download-and-use-azcopy-to-upload-files-to-azure-blob-storage/
categories:
    - Azure
    - Code
---
## PowerShell: Download and Use AzCopy (Windows) to Upload Files to Azure Blob Storage

AzCopy is a powerful command-line utility that allows for streamlined data transfer to and from Azure Blob, File, and Table storage.
  
Script below will download, install, and utilize AzCopy on a Windows machine to upload a file to Azure Blob Storage.

```powershell
# Fill with your Azure Storage Account details and file path
$sourceFilePath = "C:\MEMORY.DMP" #YOURFILE
$destinationUrl = "https://YOURSTORAGEACCOUNT.blob.core.windows.net/upload"
$SAS_Token = "?sp=YOURTOKEN"

$azCopyUrl = "https://aka.ms/downloadazcopy-v10-windows"
$destination = "C:\Temp\AzCopy.zip"

# Ensure C:\Temp exists
if (-not (Test-Path "C:\Temp")) {
    New-Item -ItemType Directory -Path "C:\Temp" -Force
}
Invoke-WebRequest -Uri $azCopyUrl -OutFile $destination
Expand-Archive -Path $destination -DestinationPath "C:\Temp\azcopy"
$azCopyExe = Get-ChildItem -Path "C:\Temp\azcopy" -Recurse -Filter "azcopy.exe" | Select-Object -First 1
if ($null -eq $azCopyExe) {
    Write-Error "AzCopy.exe not found!"
    exit
}
$azCopyPath = $azCopyExe.FullName

# Upload file using AzCopy
& $azCopyPath copy $sourceFilePath "$destinationUrl$SAS_Token"

```
