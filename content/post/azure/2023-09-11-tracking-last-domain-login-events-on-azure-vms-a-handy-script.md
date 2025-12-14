---

id: 255
title: 'Powershell: Tracking Last Domain Login Events on Azure VMs'
summary: 'Powershell: Tracking Last Domain Login Events on Azure VMs'
date: '2023-09-11T07:29:10+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=255'
permalink: /2023/09/11/tracking-last-domain-login-events-on-azure-vms-a-handy-script/
categories:
    - Azure
    - Code
    - Windows

---

## Powershell: Tracking Last Domain Login Events on Azure VMs

When managing virtual machines (VMs) in a cloud environment like Azure, it’s crucial to monitor user activities for various reasons. Whether it’s for auditing, security, or simply to track unused resources, knowing the last time someone logged in can provide valuable insights. If you’re not sure whether anyone is even logging into your Azure VM, you can check the last domain login events without breaking a sweat. All you need is PowerShell and Azure’s “Run Command” feature.  
[It’s a built-in Azure feature](https://learn.microsoft.com/en-us/azure/virtual-machines/windows/run-command) that allows you to run scripts directly on a virtual machine. This is especially useful for remote management and automation tasks. You can invoke it through the Azure Portal, PowerShell, Azure CLI, or even REST API.  

Here’s the script you can run to find the last domain login events:

```powershell
$eventID = 4624
$logName = "Security"

$lastLoginEvents = Get-WinEvent -LogName $logName -MaxEvents 10000 -FilterXPath "*[System[Provider[@Name='Microsoft-Windows-Security-Auditing'] and (EventID=$eventID)]]"

$lastDomainLogin = $null

foreach ($event in $lastLoginEvents) {
    $eventXml = [xml]$event.ToXml()
    $username = $eventXml.Event.EventData.Data | Where-Object { $_.Name -eq 'TargetUserName' } | Select-Object -ExpandProperty '#text'
    $domain = $eventXml.Event.EventData.Data | Where-Object { $_.Name -eq 'SubjectDomainName' } | Select-Object -ExpandProperty '#text'
    
    # Exclude unwanted usernames and domains
    if ($username -notmatch '^(SYSTEM|CLIUSR)$' -and $domain -ne 'NT AUTHORITY') {
        $lastDomainLogin = $event
        break
    }
}

if ($lastDomainLogin -ne $null) {
    $timestamp = $lastDomainLogin.TimeCreated
    $eventXml = [xml]$lastDomainLogin.ToXml()
    $username = $eventXml.Event.EventData.Data | Where-Object { $_.Name -eq 'TargetUserName' } | Select-Object -ExpandProperty '#text'

    Write-Host "Last domain account login event:"
    Write-Host ("User: " + $username)
    Write-Host ("Date: " + $timestamp)
} else {
    Write-Host "No domain account login events found with Event ID $eventID in log $logName."
}
```

### Key Components of the Script

- **Event ID 4624**: This is a Windows security event ID that represents successful logon events.
- **Get-WinEvent**: This PowerShell cmdlet fetches events from the event log.
- **XPath Filter**: The filter isolates specific event providers and IDs, helping you to pinpoint the exact information you’re interested in.
- **Excluding Unwanted Entries**: The script filters out system-generated and irrelevant usernames like “SYSTEM” and “CLIUSR”.

### How to Run the Script

1. Navigate to your Azure VM on the Azure Portal.
2. Go to the left menu, find the “Payload” section, and select “Run Command.”
3. Choose “RunPowerShellScript” and paste the script.
4. Hit the “Run” button to execute it.

![screenshot](https://cdn.porotnikov.com/media/2023/09/24235656/run_command-1024x528.png)
