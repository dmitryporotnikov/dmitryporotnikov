---
id: 654
title: 'Bicep: Deploy 3 RHEL 9.2 VMs with shared disks to form a cluster'
summary: 'Bicep: Deploy 3 RHEL 9.2 VMs with shared disks to form a cluster'
date: '2023-10-08T13:19:15+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=654'
permalink: /2023/10/08/deploy-3-rhel-9_2-vms-with-shared-disks-to-form-a-cluster/
categories:
    - Bicep
---

## Bicep: Deploy 3 RHEL 9.2 VMs with shared disks to form a cluster

This Bicep template automates the deployment of a highly available cluster of Red Hat Enterprise Linux (RHEL) virtual machines in Azure.

### Parameters

- **vmNames**: Specifies the names of the virtual machines. The maximum length for each name is 15 characters.
- **adminUsername**: Sets the username for the administrator account on each VM.
- **adminPassword**: Sets the password for the administrator account on each VM. The minimum length for the password is 12 characters.
- **datasizes**: Defines the sizes of the disks you want to add to the cluster.
- **vmSize**: Specifies the size of the virtual machines. The default size is `Standard_B2ms`.
- **location**: Sets the Azure region where all resources will be deployed. The default is the location of the resource group.
- **shared\_disksize**: Specifies the size of the shared disk in gigabytes.
- **vnetname**: Sets the name of your Virtual Network (VNET).
- **subnetname**: Sets the name of your subnet within the VNET.
- **vnetrg**: Specifies the resource group where your VNET resides.
- **DNS1**: Sets the primary DNS server.

#### Template
```bicep
@description('Name of the virtual machines.')
@maxLength(15)
param vmNames array = ['NAMEYOURVM1', 'NAMEYOURVM2', 'NAMEYOURVM3']

@description('Username for the Virtual Machine.')
param adminUsername string = 'azureuser'

@description('Password for the Virtual Machine.')
@minLength(12)
param adminPassword string

@description('How many disks you want to add to failover cluster?')
param datasizes array = [
  1
  2
  3
]

@description('Size of the virtual machine.')
param vmSize string = 'Standard_B2ms'

@description('Location for all resources.')
param location string = resourceGroup().location

@description('Shared disk size in gigabytes')
param shared_disksize int = 125

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

resource availabilitySet 'Microsoft.Compute/availabilitySets@2022-03-01' = {
  name: 'ClusterAS'
  location: location
  sku: {
    name: 'Aligned'
  }
  properties: {
    platformFaultDomainCount: 3
    platformUpdateDomainCount: 3
  }
}


resource clusterdisk 'Microsoft.Compute/disks@2022-03-02' = [for (datasize, lun) in datasizes: {
  name: 'clusterdisk-00${lun}'
  location: location
  sku: {
    name: 'Premium_LRS'
  }
  properties: {
    burstingEnabled: false
    creationData: {
      createOption: 'Empty'
    }
    diskSizeGB: shared_disksize
    maxShares: 3
  }
}]

output datadisksId array = [for i in range(0, length(datasizes)): {
  name: 'clusterdisk-00${i}'
  id: clusterdisk[i].id
}]
output datadisks array = [for i in range(0, length(datasizes)): {
  name: 'clusterdisk-00${i}'
  id: clusterdisk[i]
}]

resource vms 'Microsoft.Compute/virtualMachines@2022-03-01' = [for (vmName, index) in vmNames: {
  name: vmName
  location: location
  properties: {
    availabilitySet: {
      id: availabilitySet.id
    }
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName
      adminUsername: adminUsername
      adminPassword: adminPassword
    }
    storageProfile: {
      imageReference: {
        publisher: 'RedHat'
        offer: 'RHEL'
        sku: '9_2'
        version: 'latest'
      }
      osDisk: {
        createOption: 'FromImage'
        name: '${vmName}-OSdisk'
        managedDisk: {
          storageAccountType: 'StandardSSD_LRS'
        }
      }
      dataDisks: [for (disk, lun) in datasizes: {
        lun: lun+2
        createOption: 'Attach'
        managedDisk:  {
          storageAccountType: 'StandardSSD_LRS'
          id: clusterdisk[lun].id
        }
        toBeDetached: false
        deleteOption: 'Detach'
        caching: 'None'
      }]
    }
    networkProfile: {
      networkInterfaces: [
        {
          id: nic[index].id
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

resource nic 'Microsoft.Network/networkInterfaces@2022-01-01' = [for (vmName, index) in vmNames: {
  name: '${vmName}-NIC0${index+1}'
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
```
