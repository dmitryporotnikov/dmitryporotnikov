---
title: "Sample Azure REST API call to change the image reference of a VMSS"
summary: "Sample Azure REST API call to change the image reference of a VMSS"
date: '2024-09-03T00:00:00+00:00'
draft: false
---

# REST API call to change the image reference of a VMSS

This script may be useful to migrate a VMSS from one image to another. If you are using Azure CLI to do the same, you will send a full JSON payload, including all the other settings of the VMSS, which may be a problem if you have limited permissions, lets say to assign a managed identity to the VMSS. Sending only the image reference, as we do in the script below, using Patch, may be a solution.

```powershell
# Get your access token via
# az account get-access-token --query 'accessToken' -o tsv
$headers = @{
    "Authorization" = "Bearer YOURACCESSTOKEN"
    "Content-Type" = "application/json"
}

# Replace with your subscribtion IDs and VMss
$subscriptionId = "YOURSUBID"
$resourceGroupName = "testvmss"
$vmssName = "testvmss"

# Replace with yours ID
$imageReferenceId = "/subscriptions/$subscriptionId/resourceGroups/testimage/providers/Microsoft.Compute/galleries/testgallery/images/test/versions/0.0.1"

# Update VMSS Image Reference - JSON payload
$jsonBody = @"
{
    "properties": {
        "virtualMachineProfile": {
            "storageProfile": {
                "imageReference": {
                    "id": "$imageReferenceId"
                }
            }
        }
    }
}
"@

$uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Compute/virtualMachineScaleSets/$vmssName`?api-version=2022-11-01"
Invoke-RestMethod -Uri $uri -Method Patch -Headers $headers -Body $jsonBody

```