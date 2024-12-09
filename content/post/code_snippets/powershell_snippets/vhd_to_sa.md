---
title: 'Copy VM VHD to storage account directly from disk export'
summary: 'Copy VM VHD to storage account directly from disk export'
date: '2024-12-09T00:00:00+00:00'
---

# Copy VM VHD to storage account directly from disk export

```powershell

[Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::TLS12

# This part will download azcopy to the current folder: https://learn.microsoft.com/en-us/azure/storage/common/storage-use-azcopy-v10#use-in-a-script
$link = (Invoke-WebRequest -Uri https://aka.ms/downloadazcopy-v10-windows -MaximumRedirection 0 -ErrorAction SilentlyContinue).headers.location
Invoke-WebRequest -Uri $link -OutFile 'azcopy.zip'

Expand-archive -Path '.\azcopy.zip' -Destinationpath '.\azcopy'
$AzCopy = (Get-ChildItem -path '.\azcopy' -Recurse -File -Filter 'azcopy.exe').FullName
& $AzCopy                # This should run without issue if azcopy was successful

$osDisk = 'DISK EXPORT URL'

# SA VHD SAAS URL
$destination = 'https://<>.blob.core.windows.net/vhds?YOURKEY'

$blobName = ($osDisk -split '/')[-1].Split('?')[0]
if (-not $blobName.EndsWith('.vhd')) {
    $blobName += '.vhd'
}

$destinationBase = $destination.Substring(0, $destination.IndexOf('?'))
$sasPart = $destination.Substring($destination.IndexOf('?'))

$newDestination = $destinationBase.TrimEnd('/') + '/' + $blobName + $sasPart

& $AzCopy cp $osDisk $newDestination

```
