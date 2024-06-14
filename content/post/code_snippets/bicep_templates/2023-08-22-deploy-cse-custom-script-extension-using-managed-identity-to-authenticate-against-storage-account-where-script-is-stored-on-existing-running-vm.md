---
id: 88
title: 'Deploy CSE (Custom Script Extension) using Managed Identity to authenticate against storage account where script is stored on existing running VM'
summary: 'Deploy CSE (Custom Script Extension) using Managed Identity to authenticate against storage account where script is stored on existing running VM'
date: '2023-08-22T15:48:56+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=88'
permalink: /2023/08/22/deploy-cse-custom-script-extension-using-managed-identity-to-authenticate-against-storage-account-where-script-is-stored-on-existing-running-vm/
categories:
    - Bicep
---

## Deploy CSE (Custom Script Extension) using Managed Identity to authenticate against storage account where script is stored on existing running VM

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2022-04-01' existing = {
  name: 'YOURVM'
}

@description('Location for all resources.')
param location string = resourceGroup().location

resource extension 'Microsoft.Compute/virtualMachines/extensions@2021-07-01' = {
  parent: vm
  name: 'CustomScriptExtension'
  location: location
  properties: {
    publisher: 'Microsoft.Compute'
    type: 'CustomScriptExtension'
    typeHandlerVersion: '1.10'
    autoUpgradeMinorVersion: true
    settings: {
    }
    protectedSettings: {
      commandToExecute: 'powershell -ExecutionPolicy Unrestricted -File testscript.ps1' 
      managedIdentity: { clientId: '0000-0000-0000-0000'}
      fileUris: [
        'https://YOURSA.blob.core.windows.net/scripts/testscript.ps1' 
      ]
    }
    
  }
}
```
