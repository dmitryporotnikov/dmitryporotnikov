---
id: 62
title: 'PowerShell: Perform a process dump of a windows service via Sysinternals procdump tool'
summary: 'PowerShell: Perform a process dump of a windows service via Sysinternals procdump tool'
date: '2023-08-22T13:43:58+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=62'
permalink: /2023/08/22/powershell-perform-a-process-dump-of-a-windows-service-via-sysinternals-procdump-tool/
categories:
    - Code
---
## PowerShell: Perform a process dump of a windows service via Sysinternals procdump tool

```powershell
# URL of the zip file to download
$zipFileUrl = "https://download.sysinternals.com/files/Procdump.zip"

# Destination path for the downloaded zip file
$downloadPath = "$env:TEMP\downloadedFile.zip"

# Directory where the zip file will be extracted
$destinationPath = "C:\procdump"

# Download the zip file
Write-Host "Downloading the zip file from $zipFileUrl..."
Invoke-WebRequest -Uri $zipFileUrl -OutFile $downloadPath

# Check if the zip file has been downloaded
if (Test-Path $downloadPath) {
    Write-Host "Zip file downloaded successfully."

    # Create the destination directory if it doesn't exist
    if (-not (Test-Path $destinationPath)) {
        New-Item -ItemType Directory -Path $destinationPath | Out-Null
    }

    # Extract the zip file contents
    Write-Host "Extracting the contents of the zip file to $destinationPath..."
    Expand-Archive -Path $downloadPath -DestinationPath $destinationPath -Force

    Write-Host "Extraction complete. The contents of the zip file have been extracted to $destinationPath."

    # Clean up by removing the downloaded zip file
    Remove-Item -Path $downloadPath
} else {
    Write-Warning "Failed to download the zip file from $zipFileUrl."
}

# Path to the Procdump executable (Update this to the correct path on your system)
$procdumpPath = "C:\procdump\procdump.exe"

# Service name
$serviceName = "TermService"

# Check if Procdump exists
if (-not (Test-Path $procdumpPath)) {
    Write-Warning "Procdump not found at the specified path: $procdumpPath"
    #exit
}

# Get the process ID of the gpsvc service
$gpsvcProcess = Get-WmiObject -Query "SELECT * FROM Win32_Service WHERE Name = '$serviceName'"
if ($gpsvcProcess -and $gpsvcProcess.ProcessId -gt 0) {
    $processId = $gpsvcProcess.ProcessId
    $outputPath = "C:\procdump\procdump_$processId.dmp"

    # Create process dump using Procdump
    Write-Host "Creating process dump for the $serviceName service (Process ID: $processId)..."
    & $procdumpPath -accepteula -ma $processId $outputPath

    if (Test-Path $outputPath) {
        Write-Host "Process dump successfully created at: $outputPath"
    } else {
        Write-Warning "Failed to create the process dump."
    }
} else {
    Write-Warning "The $serviceName service is not running or does not exist on this system."
}
```
