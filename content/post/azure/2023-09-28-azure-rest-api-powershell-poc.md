---
id: 541
title: 'Rest API: Azure REST API POC'
summary: 'Rest API: Azure REST API POC'
date: '2023-09-28T13:20:38+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=541'
permalink: /2023/09/28/azure-rest-api-powershell-poc/
categories:
    - Azure
    - Code
---

### Rest API: Azure REST API POC

This PowerShell script serves as a Proof of Concept for teaching how to interact with Azure resources using REST API calls via PowerShell. It automates a series of Azure resource management tasks, providing a hands-on example for those looking to understand Azure REST API operations.

#### Operations Covered

1. **Create a Managed Disk**: Demonstrates how to allocate a managed disk in Azure.
2. **Create a Virtual Network (VNet)**: Shows the steps to set up a virtual network.
3. **Create a Subnet**: Explains how to add a subnet to the existing virtual network.
4. **Create a Disk Access Resource**: Walks through the process of initializing a disk access resource.
5. **Create and Associate a Private Endpoint**: Adds private endpoint and link it with the disk access resource.
6. **Update Managed Disk**: Updates the managed disk to include the disk access resource.

#### Usage

1. **Authorization**:
    - Use `az account get-access-token --query 'accessToken' -o tsv` to fetch your Azure access token.
    - Substitute `YOURTOKEN` in the `$headers` section.
2. **Resource Group**:
    - Replace the subscription ID and resource group name in the URIs with your specific details.

```powershell
# Get your access token via
# az account get-access-token --query 'accessToken' -o tsv

$headers = @{
    "Authorization" = "Bearer YOURTOKEN"
    "Content-Type" = "application/json"
}

#### Create a Managed Disk
$jsonBody = @"
{
    "location": "eastus",
    "properties": {
        "creationData": {
            "createOption": "Empty"
        },
        "diskSizeGB": 100
    }
}
"@

Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Compute/disks/myManagedDisk?api-version=2021-04-01" -Method Put -Headers $headers -Body $jsonBody

#### Create a Virtual Network
$jsonBody = @"
{
    "location": "eastus",
    "properties": {
        "addressSpace": {
            "addressPrefixes": ["10.0.0.0/16"]
        }
    }
}
"@

Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Network/virtualNetworks/myVnet?api-version=2021-02-01" -Method Put -Headers $headers -Body $jsonBody

#### CREATE SUBNET
$jsonBodySubnet = @"
{
    "properties": {
        "addressPrefix": "10.0.1.0/24"
    }
}
"@

$subnetUri = "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet?api-version=2021-02-01"
Invoke-RestMethod -Uri $subnetUri -Method Put -Headers $headers -Body $jsonBodySubnet

#### CREATE DISK ACCESS RESOURCE
$jsonBodyDiskAccess = @"
{
    "location": "eastus",
    "properties": {}
}
"@

Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Michael.Compute/diskAccesses/myDiskAccess?api-version=2021-04-01" -Method Put -Headers $headers -Body $jsonBodyDiskAccess

### Create and Associate Private Endpoint with Disk Access
$jsonBodyPrivateEndpoint = @"
{
    "location": "eastus",
    "properties": {
        "privateLinkServiceConnections": [
            {
                "name": "myPrivateLinkConnection",
                "properties": {
                    "privateLinkServiceId": "/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Compute/diskAccesses/myDiskAccess",
                    "groupIds": ["disks"]
                }
            }
        ],
        "subnet": {
            "id": "/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Network/virtualNetworks/myVnet/subnets/mySubnet"
        }
    }
}
"@

Invoke-RestMethod -Uri "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Network/privateEndpoints/myPrivateEndpoint?api-version=2021-02-01" -Method Put -Headers $headers -Body $jsonBodyPrivateEndpoint

## Update disk to include disk access resource:

$diskAccessId = "/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Compute/diskAccesses/myDiskAccess"

$jsonBody = @"
{
    "location": "eastus",
    "properties": {
        "diskAccessId": "$diskAccessId"
    }
}
"@

$uri = "https://management.azure.com/subscriptions/00000000-0000-0000-0000-00000000000/resourceGroups/AzureAPIPOC/providers/Microsoft.Compute/disks/myManagedDisk?api-version=2021-04-01"

# Make the REST API call to update the managed disk with the disk access resource
Invoke-RestMethod -Uri $uri -Method PATCH -Body $jsonBody -ContentType "application/json" -Headers $headers
```
