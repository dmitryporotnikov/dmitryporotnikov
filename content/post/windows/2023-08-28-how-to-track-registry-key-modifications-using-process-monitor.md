---
id: 190
title: 'How to Track Registry Key Modifications Using Process Monitor'
summary: 'How to Track Registry Key Modifications Using Process Monitor'
date: '2023-08-28T12:57:25+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=190'
permalink: /2023/08/28/how-to-track-registry-key-modifications-using-process-monitor/
categories:
    - Troubleshooting
    - Windows
---
## How to Track Registry Key Modifications Using Process Monitor

Modifying registry keys is a common operation that can be performed by both system and user processes. But what happens when a registry key value changes unexpectedly, leading to system behavior that’s hard to diagnose? Knowing exactly which process modified a registry key can be invaluable in system administration and debugging scenarios. One powerful tool that can help you identify such changes in real-time is Sysinternals Process Monitor.

## Pre-requisites

- A test VM or system where you have administrative rights
- Familiarity with PowerShell

## Step 1: Download Process Monitor

### Using PowerShell

If your test VM has unrestricted internet access, you can use this PowerShell script to download and run Process Monitor:

```powershell
$downloadUrl = "https://download.sysinternals.com/files/ProcessMonitor.zip"
$outputPath = "C:\Temp"
if (!(Test-Path -Path $outputPath -PathType Container)) {
    New-Item -Path $outputPath -ItemType Directory
}
Invoke-WebRequest -Uri $downloadUrl -OutFile "$outputPath\ProcessMonitor.zip"
Expand-Archive -Path "$outputPath\ProcessMonitor.zip" -DestinationPath $outputPath -Force
$processMonitorPath = Join-Path -Path $outputPath -ChildPath "Procmon.exe"
Start-Process -FilePath $processMonitorPath -ArgumentList '/accepteula'
```

### Manual Download

If you prefer a manual approach, [download Process Monitor from Microsoft's website](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon), unpack it to your chosen directory and double-click to run.

## Step 2: Prepare the Environment

Once Process Monitor is running, stop the logging of current activity by pressing `Control+E`. Clear the already captured events by pressing `Control+X`.

![screen](https://cdn.porotnikov.com/media/2023/08/24235746/image-6.png)

## Step 3: Set Up Filtering

Press `Control+L` to open the filter settings dialog.

![screen](https://cdn.porotnikov.com/media/2023/08/24235745/image-7.png)  

Add a filter to capture only registry-related events:

1. For “Event class,” select `Registry`.
2. Click “Add.

To focus on a particular registry path, add another filter:

- For “Path,” switch the condition from “is” to “contains”.
- Enter the path for which you want to capture events. For this example, we’ll use `HKLM\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU`.

To make the logs easier to read, check the “Drop filtered events” option. This will prevent any filtered events from being recorded.

![screen](https://cdn.porotnikov.com/media/2023/08/24235745/image-8-1024x174.png)
![screen](https://cdn.porotnikov.com/media/2023/08/24235743/image-10-1024x147.png)
![screen](https://cdn.porotnikov.com/media/2023/08/24235742/image-11.png)

## Step 4: Test the Configuration (Optional)

Before capturing events related to your issue, test the setup:

```powershell
$regKeyPath = "HKLM:\SOFTWARE\Policies\Microsoft\Windows\WindowsUpdate\AU"
$newValue = 1

if (Test-Path $regKeyPath) {
    Set-ItemProperty -Path $regKeyPath -Name "NoAutoUpdate" -Value $newValue
    Write-Host "Value of $regKeyPath\NoAutoUpdate has been changed to $newValue"
} else {
    Write-Host "Registry key $regKeyPath does not exist."
}
```

Run this PowerShell script and verify that you see a `RegSetValue` event in Process Monitor. Once confirmed, clear the capture again by pressing `Control+X`.

![screen](https://cdn.porotnikov.com/media/2023/08/24235742/image-12-1024x77.png)

## Step 5: Catch the Culprit

Start the capture by pressing `Control+E`. Reproduce the issue you are facing and keep an eye out for `RegSetValue` events. These will indicate which process has modified the registry key.
