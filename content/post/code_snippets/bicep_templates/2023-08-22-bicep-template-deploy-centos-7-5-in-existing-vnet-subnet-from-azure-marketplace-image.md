---
id: 14
title: 'Bicep Template: Deploy Centos 7.5 in existing VNET/Subnet from Azure MarketPlace Image'
summary: 'Bicep Template: Deploy Centos 7.5 in existing VNET/Subnet from Azure MarketPlace Image'
date: '2023-08-22T08:53:08+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=14'
permalink: /2023/08/22/bicep-template-deploy-centos-7-5-in-existing-vnet-subnet-from-azure-marketplace-image/
categories:
    - Bicep
---

## Bicep Template: Deploy Centos 7.5 in existing VNET/Subnet from Azure MarketPlace Image

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
        publisher: 'OpenLogic'
        offer: 'CentOS'
        sku: '7.5'
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
```bicep
