---
id: 702
title: 'PowerShell: Analyzing Virtual Machine Activity Logs to Determine Uptime and Downtime'
summary: 'PowerShell: Analyzing Virtual Machine Activity Logs to Determine Uptime and Downtime'
date: '2023-10-12T10:29:25+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=702'
permalink: /2023/10/12/azure-vm-log-analysis-powershell/
categories:
    - Azure
    - Code
---

## PowerShell: Analyzing Virtual Machine Activity Logs to Determine Uptime and Downtime

One common task many administrators often face is monitoring and analyzing virtual machine activity logs to determine uptime and downtime.

Here is a script that can help you getting started:

```powershell
Set-AzContext -Subscription 'YOURSUBID'
$resourceGroupName = "YOURRGNAME"
$vmName = "YOURVMNAME"
$daysAgo = 90 #Lookback duration

Write-Host "Fetching activity logs for the specified VM in the last $daysAgo days..."
$logs = Get-AzLog -ResourceGroupName $resourceGroupName -StartTime (Get-Date).AddDays(-$daysAgo)

Write-Host "Filtering logs for VM start, stop, and deallocate actions that have 'Succeeded' status..."
$relevantLogs = $logs | Where-Object {
    (
        $_.OperationName -eq 'Start Virtual Machine' -or 
        $_.OperationName -eq 'Deallocate Virtual Machine'
    ) -and 
    $_.Status -eq 'Succeeded'
} | Sort-Object EventTimestamp

Write-Host "Analyzing durations between deallocation and start..."
$deallocateTimestamp = $null
$durations = @()

foreach ($log in $relevantLogs) {
    if ($log.OperationName -eq 'Deallocate Virtual Machine') {
        $deallocateTimestamp = $log.EventTimestamp
        $durations += [PSCustomObject] @{
            VMName    = $vmName
            Action    = $log.OperationName
            Timestamp = $log.EventTimestamp
            Duration  = $null
        }
    }
    elseif ($log.OperationName -eq 'Start Virtual Machine' -and $deallocateTimestamp) {
        $timeSpan = $log.EventTimestamp - $deallocateTimestamp
        $durationInMinutes = [math]::Round($timeSpan.TotalMinutes, 2)
        $durations += [PSCustomObject] @{
            VMName    = $vmName
            Action    = $log.OperationName
            Timestamp = $log.EventTimestamp
            DurationOffline_in_Minutes_Before_this_Start  = $durationInMinutes
        }
        $deallocateTimestamp = $null
    }
}

Write-Host "Displaying the durations between deallocations and subsequent starts..."
$durations | Format-Table -Property VMName, Action, Timestamp, DurationOffline_in_Minutes_Before_this_Start
```

Sample output:

![screenshot](https://cdn.porotnikov.com/media/2023/10/12103206/image-16-1024x249.png)
