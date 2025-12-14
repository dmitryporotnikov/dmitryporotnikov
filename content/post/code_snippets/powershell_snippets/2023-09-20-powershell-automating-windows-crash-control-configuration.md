---
id: 376
title: 'PowerShell: Automating Windows Crash Control Configuration'
summary: 'PowerShell: Automating Windows Crash Control Configuration'
date: '2023-09-20T14:29:42+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=376'
permalink: /2023/09/20/powershell-automating-windows-crash-control-configuration/
amazonS3_cache:
    - 'a:2:{s:55:"//porotnikov.com/wp-content/uploads/2023/08/image-6.png";a:1:{s:9:"timestamp";i:1695292569;}s:63:"//porotnikov.com/wp-content/uploads/2023/08/image-6-300x124.png";a:1:{s:9:"timestamp";i:1695292569;}}'
categories:
    - Code
---
## PowerShell: Automating Windows Crash Control Configuration

This script aims to automate the task of configuring several settings under the Windows `CrashControl` registry path. The configurations include specifying memory dump paths, enabling auto-reboot, setting logging levels, and more.  
  
**Reference documentation:**[  
https://learn.microsoft.com/en-us/troubleshoot/windows-client/performance/generate-a-kernel-or-complete-crash-dump](
https://learn.microsoft.com/en-us/troubleshoot/windows-client/performance/generate-a-kernel-or-complete-crash-dump)

```powershell
$dumpFilePath = 'D:\MEMORY.DMP'
$minidumpDir = '%SystemRoot%\Minidump'
$registryPath = "HKLM:\SYSTEM\CurrentControlSet\Control\CrashControl"

try
{
Set-ItemProperty -Path $registryPath -Name "AutoReboot" -Value 1
Set-ItemProperty -Path $registryPath -Name "CrashDumpEnabled" -Value 1
Set-ItemProperty -Path $registryPath -Name "DisableEmoticon" -Value 1
Set-ItemProperty -Path $registryPath -Name "DumpFile" -Value $dumpFilePath
Set-ItemProperty -Path $registryPath -Name "DumpLogLevel" -Value 0
Set-ItemProperty -Path $registryPath -Name "EnableLogFile" -Value 1
Set-ItemProperty -Path $registryPath -Name "LogEvent" -Value 1
Set-ItemProperty -Path $registryPath -Name "MinidumpDir" -Value $minidumpDir

Set-ItemProperty -Path $registryPath -Name "MinidumpsCount" -Value 5
Set-ItemProperty -Path $registryPath -Name "Overwrite" -Value 1
#Set-ItemProperty -Path $registryPath -Name "DumpFilters" -Value "dumpfve.sys"
Set-ItemProperty -Path $registryPath -Name "AlwaysKeepMemoryDump" -Value 1
Set-ItemProperty -Path $registryPath -Name "NMICrashDump" -Value 1
Set-ItemProperty -Path $registryPath -Name "IgnorePagefileSize" -Value 1
Write-Output "Registry values have been set successfully."
}
catch
{

Write-Output "Something went wrong. Are you running the script as Admin?"
}
```
