---
title: 'Powershell snippet to check for a free IP in Azure Virtual Network'
summary: 'Powershell snippet to check for a free IP in Azure Virtual Network'
date: '2024-12-04T00:00:00+00:00'
---

## Powershell snippet to check for a free IP in Azure Virtual Network

```powershell

# Get your Azure access token via:
# az account get-access-token --query 'accessToken' -o tsv

# Replace YOURTOKEN with your actual Azure access token
$headers = @{
    "Authorization" = "Bearer TOKENHERE"
    "Content-Type" = "application/json"
}

# Replace the placeholders with your info
$subscriptionId = "SUBID"
$resourceGroupName = "RGNAME"
$virtualNetworkName = "VNETNAME"
$ipAddressToCheck = "10.1.0.4"
$apiVersion = "2024-05-01"

# Construct the URI for the REST API call
$uri = "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Network/virtualNetworks/$virtualNetworkName/CheckIPAddressAvailability?ipAddress=$ipAddressToCheck&api-version=$apiVersion"

# Make the REST API call
$response = Invoke-RestMethod -Uri $uri -Method Get -Headers $headers

# Output the response
$response

# Process the response to display IP availability and suggested available IPs
if ($response.available -eq $true) {
    Write-Host "The IP address $ipAddressToCheck is available."
} else {
    Write-Host "The IP address $ipAddressToCheck is NOT available."
    Write-Host "Suggested available IP addresses:"
    $response.availableIPAddresses | ForEach-Object { Write-Host $_ }
}

```
