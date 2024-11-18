---
title: "Script to Identify Unused VHDs in Azure Storage Accounts"
summary: "Script to Identify Unused VHDs in Azure Storage Accounts"
date: '2024-11-15T00:00:00+01:00'
draft: false
---

# Script to Identify Unused VHDs in Azure Storage Accounts

This script identifies unused VHDs in Azure storage accounts. It first retrieves the list of VHDs associated with virtual machines. Then, it collects all VHDs available in the storage accounts. After that, it compares both lists to find orphaned VHDs that are not attached to any virtual machine, and outputs the URIs of those unused VHDs.

```powershell

# Script to Identify Unused VHDs in Azure Storage Accounts
# Author: Dmitry Porotnikov
# Disclaimer: This script is provided "as is" without warranty of any kind.
# Pre-requisites: Install the Azure Resource Graph Module using the following command: 
# Install-Module -Name Az.ResourceGraph -AllowClobber -Force
# ------------------------------------------------
# If you will face an issue with Az.ResourceGraph complaining about Az.Accounts being too old:
# Istall-Module -Name Az.Accounts -RequiredVersion 3.0.5
# Then open a new cloud shell session and do:
# Import-Module -Name Az.Accounts -RequiredVersion 3.0.5 -Force
# Import-Module -Name Az.ResourceGraph
#  ------------------------------------------------
# Note: Script tested in Azure Cloud Shell (PowerShell mode)

# Step 1: Get the list of VHDs associated with VMs using Azure Resource Graph
$vmVhds = Search-AzGraph -Query @"
resources
| where type == "microsoft.compute/virtualmachines"
| where properties.storageProfile.osDisk.managedDisk == ""
| extend diskUri = properties.storageProfile.osDisk.vhd.uri
| project diskUri
| union (
    resources
    | where type == "microsoft.compute/virtualmachines"
    | mvexpand dataDisk = properties.storageProfile.dataDisks
    | where dataDisk.managedDisk == ""
    | extend diskUri = dataDisk.vhd.uri
    | project diskUri
)
"@ 

$vmVhdUris = $vmVhds.diskUri

# Step 2: Get all VHDs from storage accounts
$storageAccounts = Get-AzStorageAccount
$allVhds = @()
foreach ($storageAccount in $storageAccounts) {
    $context = $storageAccount.Context
    $containers = Get-AzStorageContainer -Context $context

    foreach ($container in $containers) {
        $blobs = Get-AzStorageBlob -Container $container.Name -Context $context
        $vhdBlobs = $blobs | Where-Object { $_.Name -like "*.vhd" }
        foreach ($vhdBlob in $vhdBlobs) {
            $vhdUri = ($context.BlobEndPoint + $container.Name + "/" + $vhdBlob.Name)
            $allVhds += $vhdUri
        }
    }
}

# Step 3: Find orphaned VHDs by comparing the lists
$orphanedVhds = $allVhds | Where-Object { $_ -notin $vmVhdUris }

# Output the orphaned VHD URIs
$orphanedVhds

```

# To check only certain Storage Accounts in Certain Resource Groups
```powershell

$resourceGroupName = "YourResourceGroupName"

# Step 1: Get the list of VHDs associated with VMs using Azure Resource Graph
$vmVhds = Search-AzGraph -Query @"
resources
| where type == "microsoft.compute/virtualmachines"
| where properties.storageProfile.osDisk.managedDisk == ""
| extend diskUri = properties.storageProfile.osDisk.vhd.uri
| project diskUri
| union (
    resources
    | where type == "microsoft.compute/virtualmachines"
    | mvexpand dataDisk = properties.storageProfile.dataDisks
    | where dataDisk.managedDisk == ""
    | extend diskUri = dataDisk.vhd.uri
    | project diskUri
)
"@ 

$vmVhdUris = $vmVhds.diskUri

# Step 2: Get all storage accounts in the resource group
$storageAccounts = Get-AzStorageAccount -ResourceGroupName $resourceGroupName

# Step 3: Get all VHDs from all storage accounts
$allVhds = @()

foreach ($storageAccount in $storageAccounts) {
    $context = $storageAccount.Context
    $containers = Get-AzStorageContainer -Context $context

    foreach ($container in $containers) {
        $blobs = Get-AzStorageBlob -Container $container.Name -Context $context
        $vhdBlobs = $blobs | Where-Object { $_.Name -like "*.vhd" }

        foreach ($vhdBlob in $vhdBlobs) {
            $vhdUri = ($context.BlobEndPoint + $container.Name + "/" + $vhdBlob.Name)
            $allVhds += $vhdUri
        }
    }
}

# Step 4: Find orphaned VHDs by comparing the lists
$orphanedVhds = $allVhds | Where-Object { $_ -notin $vmVhdUris }

# Output the orphaned VHD URIs
$orphanedVhds


```

# Multiple Resource Groups

```powershell

# Define the list of resource groups
$resourceGroupNames = @("ResourceGroup1", "ResourceGroup2", "ResourceGroup3")

# Step 1: Get the list of VHDs associated with VMs using Azure Resource Graph
$vmVhds = Search-AzGraph -Query @"
resources
| where type == "microsoft.compute/virtualmachines"
| where properties.storageProfile.osDisk.managedDisk == ""
| extend diskUri = properties.storageProfile.osDisk.vhd.uri
| project diskUri
| union (
    resources
    | where type == "microsoft.compute/virtualmachines"
    | mvexpand dataDisk = properties.storageProfile.dataDisks
    | where dataDisk.managedDisk == ""
    | extend diskUri = dataDisk.vhd.uri
    | project diskUri
)
"@ 

$vmVhdUris = $vmVhds.diskUri

# Step 2: Initialize an array to store VHD URIs
$allVhds = @()

# Step 3: Iterate over each resource group and get the storage accounts
foreach ($resourceGroupName in $resourceGroupNames) {
    $storageAccounts = Get-AzStorageAccount -ResourceGroupName $resourceGroupName

    foreach ($storageAccount in $storageAccounts) {
        $context = $storageAccount.Context
        $containers = Get-AzStorageContainer -Context $context

        foreach ($container in $containers) {
            $blobs = Get-AzStorageBlob -Container $container.Name -Context $context
            $vhdBlobs = $blobs | Where-Object { $_.Name -like "*.vhd" }

            foreach ($vhdBlob in $vhdBlobs) {
                $vhdUri = ($context.BlobEndPoint + $container.Name + "/" + $vhdBlob.Name)
                $allVhds += $vhdUri
            }
        }
    }
}

# Step 4: Find orphaned VHDs by comparing the lists
$orphanedVhds = $allVhds | Where-Object { $_ -notin $vmVhdUris }

# Output the orphaned VHD URIs
$orphanedVhds

```
