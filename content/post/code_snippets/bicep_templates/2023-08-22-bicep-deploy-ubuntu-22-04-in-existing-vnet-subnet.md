---
id: 80
title: 'Bicep: Deploy Ubuntu 22.04 in existing VNET/Subnet'
summary: 'Bicep: Deploy Ubuntu 22.04 in existing VNET/Subnet'
date: '2023-08-22T15:08:16+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=80'
permalink: /2023/08/22/bicep-deploy-ubuntu-22-04-in-existing-vnet-subnet/
categories:
    - Bicep
---

## Bicep: Deploy Ubuntu 22.04 in existing VNET/Subnet

```bicep
@description('The name of you Virtual Machine.')
param vmName string = 'NAMEYOURVM'

@description('Username for the Virtual Machine.')
param adminUsername string = 'azureuser'

@description('Type of authentication to use on the Virtual Machine. SSH key is recommended.')
@allowed([
  'sshPublicKey'
  'password'
])
param authenticationType string = 'password'

@description('SSH Key or password for the Virtual Machine. SSH key is recommended.')
@secure()
param adminPasswordOrKey string


@description('Location for all resources.')
param location string = 'northeurope'

@description('The size of the VM')
param vmSize string = 'Standard_B2s'

var linuxConfiguration = {
  disablePasswordAuthentication: true
  ssh: {
    publicKeys: [
      {
        path: '/home/${adminUsername}/.ssh/authorized_keys'
        keyData: adminPasswordOrKey
      }
    ]
  }
}
@description('Your VNET name')
param vnetname string = 'EUN-Virtual-Network'

@description('Your Subnet name')
param subnetname string = 'SD-WAN'

@description('Your VNET RG name')
param vnetrg string = 'SD-WAN'

resource ngfwvnet 'Microsoft.Network/virtualnetworks@2021-02-01' existing = {
  name: vnetname
  scope: resourceGroup(vnetrg)
}
resource subnet 'Microsoft.Network/virtualNetworks/subnets@2021-02-01' existing = {
  parent: ngfwvnet
  name: subnetname
}

resource nic 'Microsoft.Network/networkInterfaces@2021-02-01' = {
  name: '${vmName}-NIC01'
  location: location
  dependsOn: [
    subnet
  ]
  properties: {
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
}

resource vm 'Microsoft.Compute/virtualMachines@2021-11-01' = {
  name: vmName
  location: location
  properties: {
    hardwareProfile: {
      vmSize: vmSize
    }
    storageProfile: {
      osDisk: {
        name: '${vmName}-OSdisk'
        createOption: 'FromImage'
        managedDisk: {
          storageAccountType: 'StandardSSD_LRS'
        }
      }
      imageReference: {
        publisher: 'canonical'
        offer: '0001-com-ubuntu-server-jammy'
        sku: '22_04-lts'
        version: 'latest'
      }
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic.id
        }
      ]
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPasswordOrKey
      linuxConfiguration: ((authenticationType == 'password') ? null : linuxConfiguration)
    }
  }
}

output adminUsername string = adminUsername
```
