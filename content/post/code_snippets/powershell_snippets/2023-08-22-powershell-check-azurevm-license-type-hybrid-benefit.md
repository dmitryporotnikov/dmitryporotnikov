---
id: 60
title: 'PowerShell: Check AzureVM license type (Hybrid Benefit)'
summary: 'PowerShell: Check AzureVM license type (Hybrid Benefit)'
date: '2023-08-22T13:35:06+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=60'
permalink: /2023/08/22/powershell-check-azurevm-license-type-hybrid-benefit/
categories:
    - Code
---

## PowerShell: Check AzureVM license type (Hybrid Benefit)

```powershell
# Login to Azure
Connect-AzAccount

# Set the subscription context
$subscriptionId = "<your subscription id>"
Set-AzContext -SubscriptionId $subscriptionId

# Get all Linux VMs in the subscription
$vms = Get-AzVM | Where-Object { $_.StorageProfile.OSDisk.OSType -eq "Linux" }

# Loop through each VM and check the license type (You should see *-BYOS, e.g rhel-byos in case of BYOS)
foreach ($vm in $vms) {

    $vmConfig = Get-AzVM -ResourceGroupName $vm.ResourceGroupName -Name $vm.Name
    Write-Host "VM $($vm.Name) publisher: $($vmConfig.StorageProfile.ImageReference.Publisher)"
    Write-Host "VM $($vm.Name) offer: $($vmConfig.StorageProfile.ImageReference.Offer)"

    #checking for HybridBenefit (https://learn.microsoft.com/en-us/azure/virtual-machines/linux/azure-hybrid-benefit-linux)
    if ($vmConfig.LicenseType -eq $null) {
        Write-Host "VM $($vm.Name) has no hybrid benefit enabled"
    }
    else {
        Write-Host "VM $($vm.Name) has hybrid benefit enabled"
    }
    Write-Host "------""
    ```
