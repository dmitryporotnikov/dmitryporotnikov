---
id: 84
title: 'Bicep: Create empty Managed Disk'
date: '2023-08-22T15:28:12+00:00'
summary: 'Bicep: Create empty Managed Disk'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=84'
permalink: /2023/08/22/bicep-create-empty-managed-disk/
categories:
    - Uncategorized
---

## Bicep: Create empty Managed Disk'

```bicep
param diskName string = 'testdisk'
param diskSizeGb int = 128
param diskSku string = 'Premium_LRS'

resource disk 'Microsoft.Compute/disks@2023-01-02' = {
  name: diskName
  location: resourceGroup().location
  sku: {
    name: diskSku
  }
  properties: {
    creationData: {
      createOption: 'Empty'
    }
    diskSizeGB: diskSizeGb
  }
  tags: {
    environment: 'demo'
  }
}

output diskId string = disk.id
```
