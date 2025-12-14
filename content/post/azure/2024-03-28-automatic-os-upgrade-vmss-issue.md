---
id: 793
title: 'Automatic OS Upgrade is not supported for this Virtual Machine Scale Set because a health probe or health extension was not specified.'
summary: 'Automatic OS Upgrade is not supported for this Virtual Machine Scale Set because a health probe or health extension was not specified.'
date: '2024-03-28T18:09:38+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=793'
permalink: /2024/03/28/automatic-os-upgrade-vmss-issue/
categories:
    - Azure
    - Bicep
---

## Automatic OS Upgrade is not supported for this Virtual Machine Scale Set because a health probe or health extension was not specified

If you face this issue during your VMSS deployment:

- **ErrorCode:** DeploymentFailed
- **Content:** {"status":"Failed","error":{"code":"DeploymentFailed","target":"/subscriptions/YOURID/resourceGroups/vmss/providers/Microsoft.Resources/deployments/VMSS_AutomaticOsUpgrade-240328-1303","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see [https://aka.ms/arm-deployment-operations](https://aka.ms/arm-deployment-operations) for usage details.","details":[{"code":"BadRequest","message":"**Automatic OS Upgrade is not supported for this Virtual Machine Scale Set because a health probe or health extension was not specified.**"}]}}

It is very easy to bypass it.

If you do not have a port for your app to monitor, you can monitor management ports, like so:

###### Linux ARM:
```json
{
  "extensionProfile": {
    "extensions": [
      {
        "name": "HealthExtension",
        "properties": {
          "publisher": "Microsoft.ManagedServices",
          "type": "ApplicationHealthLinux",
          "autoUpgradeMinorVersion": true,
          "typeHandlerVersion": "1.0",
          "settings": {
            "protocol": "tcp",
            "port": 22,
            "intervalInSeconds": 5,
            "numberOfProbes": 1
          }
        }
      }
    ]
  }
}
```

###### Linux Bicep:
```yaml
extensionProfile: {
  extensions: [
    {
      name: 'HealthExtension'
      properties: {
        publisher: 'Microsoft.ManagedServices'
        type: 'ApplicationHealthLinux'
        autoUpgradeMinorVersion: true
        typeHandlerVersion: '1.0'
        settings: {
          protocol: 'tcp'
          port: 22
          intervalInSeconds: 5
          numberOfProbes: 1
        }
      }
    }
  ]
}
```

###### Windows ARM:
```json
{
  "extensionProfile": {
    "extensions": [
      {
        "name": "HealthExtension",
        "properties": {
          "publisher": "Microsoft.ManagedServices",
          "type": "ApplicationHealthWindows",
          "autoUpgradeMinorVersion": true,
          "typeHandlerVersion": "1.0",
          "settings": {
            "protocol": "tcp",
            "port": 3389,
            "intervalInSeconds": 5,
            "numberOfProbes": 1
          }
        }
      }
    ]
  }
}
```

###### Windows Bicep:
```yaml
extensionProfile: {
  extensions: [
    {
      name: 'HealthExtension'
      properties: {
        publisher: 'Microsoft.ManagedServices'
        type: 'ApplicationHealthWindows'
        autoUpgradeMinorVersion: true
        typeHandlerVersion: '1.0'
        settings: {
          protocol: 'tcp'
          port: 3389
          intervalInSeconds: 5
          numberOfProbes: 1
        }
      }
    }
  ]
}
```

The monitoring happens from within VMSS instance, so there is no need to allow those ports on firewalls:

![](https://cdn.porotnikov.com/media/2024/03/28180822/image-1-1024x247.jpg)
