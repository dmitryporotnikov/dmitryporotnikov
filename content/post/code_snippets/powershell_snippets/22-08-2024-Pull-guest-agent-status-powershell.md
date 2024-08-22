---
id: 60
title: 'PowerShell: Check AzureVM license type (Hybrid Benefit)'
summary: 'PowerShell: Check AzureVM license type (Hybrid Benefit)'
date: '2023-08-21T13:35:06+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=60'
permalink: /2023/08/22/powershell-check-azurevm-license-type-hybrid-benefit/
categories:
    - Code
---

## PowerShell: Check VM guest agent status for all VMs across subscription 

```powershell
# Azure VM Agent Status Check Script
# Purpose: This script checks the status and version of the VM agent for all running VMs in a specified Azure subscription.
# It provides information about the OS type, VM agent provisioning status, and current agent version.

# Uncomment the following line if you need to authenticate to Azure
# Connect-AzAccount

# Connect-AzAccount
Select-AzSubscription -Subscription "YOURSUBID"
$vms = Get-AzVM

foreach ($vm in $vms) {
    $vmStatus = Get-AzVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name -Status
    $powerState = ($vmStatus.Statuses | Where-Object Code -eq "PowerState/running").DisplayStatus
    if ($powerState -ne "VM running") {
        Write-Host "Skipping VM: $($vm.Name) - Not running" -ForegroundColor Yellow
        continue
    }

    Write-Host "Checking VM: $($vm.Name) in Resource Group: $($vm.ResourceGroupName)" -ForegroundColor Cyan
 
    $osType = $vm.StorageProfile.OsDisk.OsType
    $provisionVmAgent = if ($osType -eq "Windows") {
        $vm.OSProfile.WindowsConfiguration.ProvisionVMAgent
    } else {
        $vm.OSProfile.LinuxConfiguration.ProvisionVMAgent
    }
    
    Write-Host "OS Type: $osType"
    Write-Host "Provision VM Agent: $provisionVmAgent"
    
 
    $agentStatus = ($vmStatus.VMAgent.Statuses | Where-Object Code -like "ProvisioningState/*").DisplayStatus
    
    if ($agentStatus) {
        Write-Host "Agent Status: $agentStatus" -ForegroundColor Green
    } else {
        Write-Host "Agent Status: Not available or not installed" -ForegroundColor Yellow
    }
    
 
    $agentVersion = $vmStatus.VMAgent.VMAgentVersion
    if ($agentVersion) {
        Write-Host "Agent Version: $agentVersion" -ForegroundColor Green
    } else {
        Write-Host "Agent Version: Not available" -ForegroundColor Yellow
    }
    
    Write-Host "------------------------"
}
    ```
