---
title: "Your Availability Zone 1 in one subscription might not be the same physical zone as Availability Zone 1 in another"
summary: "Your Availability Zone 1 in one subscription might not be the same physical zone as Availability Zone 1 in another"
date: '2025-04-11T00:00:00+00:00'
draft: false
---

# Your "Availability Zone 1" in one subscription might not be the same physical zone as "Availability Zone 1" in another

Did you know that “Availability Zone 1” in one Azure subscription might not point to the same physical datacenter as “Zone 1” in another?

Logical zones (1, 2, 3) are just labels Azure uses per subscription. The actual physical zones may or may not match your logical zones.

If you deploy two VMs: one in Zone 1 and another in Zone 2 across two subscriptions, they could end up in the same physical datacenter. This might accidentally undermine your redundancy plans for stretched clusters or disaster recovery. 

Luckily, you can check your physical zones per subscription like so:

```powershell

$subscriptionId = (Get-AzContext).Subscription.ID
$response = Invoke-AzRestMethod -Method GET -Path "/subscriptions/$subscriptionId/locations?api-version=2022-12-01"
$locations = ($response.Content | ConvertFrom-Json).value
$filteredLocation = $locations | Where-Object { $_.name -eq "YOURAZUREREGION" }
$filteredLocation

```

Look at my own subscription below - South East Asia zone 1 is actually physical zone 3 :)

![screenshot](https://cdn.porotnikov.com/media/2025/4/11/zones.png)
