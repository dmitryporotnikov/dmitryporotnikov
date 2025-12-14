---
id: 90
title: 'Bicep: Deploy 40 VMs at once, with CSE to auto-configure them.'
summary: 'Bicep: Deploy 40 VMs at once, with CSE to auto-configure them.'
date: '2023-08-22T16:10:34+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=90'
permalink: /2023/08/22/bicep-deploy-40-vms-at-once-with-cse-to-auto-configure-them/
categories:
    - Bicep
---

## Bicep: Deploy 40 VMs at once, with CSE to auto-configure them

```bicep
var managedIdentityId = '/UMIRESOURCEID/UMI'

@description('Username for the Virtual Machine.')
param adminUsername string = 'azureuser'

@description('Password for the Virtual Machine.')
@minLength(12)
param adminPassword string

@description('The Windows version for the VM. This will pick a fully patched image of this given Windows version.')
@allowed([
  '2019-Datacenter-smalldisk'
])
param OSVersion string = '2019-Datacenter-smalldisk'

@description('Size of the virtual machine.')
param vmSize string = 'Standard_B2ms'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('Your VNET name')
param vnetname string = 'EUN-Virtual-Network'

@description('Your Subnet name')
param subnetname string = 'SD-WAN'

@description('Your VNET RG name')
param vnetrg string = 'SD-WAN'

@description('DNS Server 1')
param DNS1 string = '192.168.1.210'

resource ngfwvnet 'Microsoft.Network/virtualnetworks@2021-02-01' existing = {
  name: vnetname
  scope: resourceGroup(vnetrg)
}
resource subnet 'Microsoft.Network/virtualNetworks/subnets@2021-02-01' existing = {
  parent: ngfwvnet
  name: subnetname
}

@description('Base name of the virtual machines. A unique number will be appended to this base name for each VM.')
@maxLength(10)
param baseVmName string = 'VM'

// other parameters are same

// Loops for VM, NIC, and extension resources
resource nics 'Microsoft.Network/networkInterfaces@2022-01-01' = [for i in range(0, 40): {
  name: '${baseVmName}-${i}-NIC01'
  location: location
  dependsOn: [
    subnet
  ]
  properties: {
    dnsSettings: {
      dnsServers: [
        DNS1
      ]
    }
    ipConfigurations: [
      {
        name: 'ipConfig'
        properties: {
          privateIPAllocationMethod: 'Dynamic'    
          subnet: {
            id: subnet.id
          }
          primary: true
          privateIPAddressVersion: 'IPv4'
        }
      }
    ]
  }
}]

resource vms 'Microsoft.Compute/virtualMachines@2021-03-01' = [for i in range(0, 40): {
  identity: {
    type: 'UserAssigned'
    userAssignedIdentities: {
      '${managedIdentityId}': {}
    }
  }
  name: '${baseVmName}-${i}'
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: '${baseVmName}-${i}'
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    storageProfile: {
      imageReference: {
        publisher: 'MicrosoftWindowsServer'
        offer: 'WindowsServer'
        sku: OSVersion
        version: 'latest'
      }
      osDisk: {
        createOption: 'FromImage'
        name: '${baseVmName}-${i}-OSdisk'
        managedDisk: {
          storageAccountType: 'StandardSSD_LRS'
        }
      }
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nics[i].id
        }
      ]
    }
    diagnosticsProfile: {
      bootDiagnostics: {
        enabled: false
      }
    }
  }
}]

resource extensions 'Microsoft.Compute/virtualMachines/extensions@2021-07-01' = [for i in range(0, 40): {
  parent: vms[i]
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
      commandToExecute: 'powershell -ExecutionPolicy Unrestricted -File EnableAnsibleWinRM.ps1' 
      managedIdentity: { clientId: '0000-0000-0000a-0000'}
      fileUris: [
        'https://YOURSA.blob.core.windows.net/scripts/EnableAnsibleWinRM.ps1' 
      ]
    }
  }
}]
```
