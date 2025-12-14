---
id: 82
title: 'Bicep: Create a ZRS Storage Account with a random Storage Account name'
summary: "Bicep: Create a ZRS Storage Account with a random Storage Account name"
date: '2023-08-22T15:24:47+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=82'
permalink: /2023/08/22/bicep-create-a-zrs-storage-account-with-a-random-storage-account-name/
categories:
    - Bicep
---

## Bicep: Create a ZRS Storage Account with a random Storage Account name

```bicep
param location string = 'northeurope'
var prefix = 'stor'
var uniqueId = uniqueString(location, resourceGroup().id)
var storageAccountName = substring('${prefix}${uniqueId}', 0, 17)

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-08-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_ZRS'
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    accessTier: 'Hot'
    allowBlobPublicAccess: false
  }
}
```
