---
id: 66
title: 'Powershell: Restart Group Policy Client Service (Can only be used as Run-Command on Azure Portal)'
summary: 'Powershell: Restart Group Policy Client Service (Can only be used as Run-Command on Azure Portal)'
date: '2023-08-22T13:48:49+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=66'
permalink: /2023/08/22/powershell-restart-group-policy-client-service-can-be-used-as-run-command-on-azure-portal/
categories:
    - Code
---
## Powershell: Restart Group Policy Client Service (Can only be used as Run-Command on Azure Portal)

```powershell
# Service name
$serviceName = "gpsvc"

# Check if the service exists
if (Get-Service -Name $serviceName -ErrorAction SilentlyContinue) {
    # Stop the service if it's running
    if ((Get-Service -Name $serviceName).Status -eq 'Running') {
        Write-Host "Stopping the $serviceName service..."
        Stop-Service -Name $serviceName -Force
    }

    # Start the service
    Write-Host "Starting the $serviceName service..."
    Start-Service -Name $serviceName

    # Verify if the service is running
    if ((Get-Service -Name $serviceName).Status -eq 'Running') {
        Write-Host "The $serviceName service has been restarted successfully."
    } else {
        Write-Warning "Failed to restart the $serviceName service."
    }
} else {
    Write-Warning "The $serviceName service does not exist on this system."
}
```
