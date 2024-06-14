---
id: 86
title: 'Bicep: Deploy 2 Node Windows Failover Cluster, in a Proximity Placement Group (PPG), with 3 shared clustered disks and CSE for automatic cluster configuration'
summary: 'Bicep: Deploy 2 Node Windows Failover Cluster, in a Proximity Placement Group (PPG), with 3 shared clustered disks and CSE for automatic cluster configuration'
date: '2023-08-22T15:36:06+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=86'
permalink: /2023/08/22/bicep-deploy-2-node-windows-failover-cluster-in-a-proximity-placement-group-ppg-with-3-shared-clustered-disks-and-cse-for-automatic-cluster-configuration/
categories:
    - Bicep
---

## Bicep: Deploy 2 Node Windows Failover Cluster, in a Proximity Placement Group (PPG), with 3 shared clustered disks and CSE for automatic cluster configuration

```bicep
@description('Name of the virtual machine.')
@maxLength(15)
param vmName string = 'NAMEYOURVM'

@description('Name of the second virtual machine.')
@maxLength(15)
param vmName2 string = 'NAMEYOURVM2'

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

@description('The Windows version for the VM. This will pick a fully patched image of this given Windows version.')
@allowed([
'2008-R2-SP1'
'2008-R2-SP1-smalldisk'
'2012-Datacenter'
'2012-datacenter-gensecond'
'2012-Datacenter-smalldisk'
'2012-datacenter-smalldisk-g2'
'2012-Datacenter-zhcn'
'2012-datacenter-zhcn-g2'
'2012-R2-Datacenter'
'2012-r2-datacenter-gensecond'
'2012-R2-Datacenter-smalldisk'
'2012-r2-datacenter-smalldisk-g2'
'2012-R2-Datacenter-zhcn'
'2012-r2-datacenter-zhcn-g2'
'2016-Datacenter'
'2016-datacenter-gensecond'
'2016-datacenter-gs'
'2016-Datacenter-Server-Core'
'2016-datacenter-server-core-g2'
'2016-Datacenter-Server-Core-smalldisk'
'2016-datacenter-server-core-smalldisk-g2'
'2016-Datacenter-smalldisk'
'2016-datacenter-smalldisk-g2'
'2016-Datacenter-with-Containers'
'2016-datacenter-with-containers-g2'
'2016-datacenter-with-containers-gs'
'2016-Datacenter-zhcn'
'2016-datacenter-zhcn-g2'
'2019-Datacenter'
'2019-Datacenter-Core'
'2019-datacenter-core-g2'
'2019-Datacenter-Core-smalldisk'
'2019-datacenter-core-smalldisk-g2'
'2019-Datacenter-Core-with-Containers'
'2019-datacenter-core-with-containers-g2'
'2019-Datacenter-Core-with-Containers-smalldisk'
'2019-datacenter-core-with-containers-smalldisk-g2'
'2019-datacenter-gensecond'
'2019-datacenter-gs'
'2019-Datacenter-smalldisk'
'2019-datacenter-smalldisk-g2'
'2019-Datacenter-with-Containers'
'2019-datacenter-with-containers-g2'
'2019-datacenter-with-containers-gs'
'2019-Datacenter-with-Containers-smalldisk'
'2019-datacenter-with-containers-smalldisk-g2'
'2019-Datacenter-zhcn'
'2019-datacenter-zhcn-g2'
'2022-datacenter'
'2022-datacenter-azure-edition'
'2022-datacenter-azure-edition-core'
'2022-datacenter-azure-edition-core-smalldisk'
'2022-datacenter-azure-edition-smalldisk'
'2022-datacenter-core'
'2022-datacenter-core-g2'
'2022-datacenter-core-smalldisk'
'2022-datacenter-core-smalldisk-g2'
'2022-datacenter-g2'
'2022-datacenter-smalldisk'
'2022-datacenter-smalldisk-g2'
])
param OSVersion string = '2022-datacenter-azure-edition-core'

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

resource PPG 'Microsoft.Compute/proximityPlacementGroups@2022-03-01' = {
  name: 'FailoverClusterPPG'
  location: location
}

resource extension 'Microsoft.Compute/virtualMachines/extensions@2015-06-15' = {
  parent: vm
  name: 'enableansible'
  location: location
  properties: {
    publisher: 'Microsoft.Compute'
    type: 'CustomScriptExtension'
    typeHandlerVersion: '1.10'
    autoUpgradeMinorVersion: true
    settings: {}
    protectedSettings: {
      commandToExecute: 'powershell -ExecutionPolicy Unrestricted -File EnableAnsibleWinRM.ps1'
      fileUris: [
        'URL TO YOUR POWERSHELL SCRIPT'
      ]
    }
  }
}

resource extension2 'Microsoft.Compute/virtualMachines/extensions@2015-06-15' = {
  parent: vm2
  name: 'enableansible2'
  location: location
  properties: {
    publisher: 'Microsoft.Compute'
    type: 'CustomScriptExtension'
    typeHandlerVersion: '1.10'
    autoUpgradeMinorVersion: true
    settings: {}
    protectedSettings: {
      commandToExecute: 'powershell -ExecutionPolicy Unrestricted -File EnableAnsibleWinRM.ps1'
      fileUris: [
        'URL TO YOUR POWERSHELL SCRIPT'
      ]
    }
  }
}



resource nic 'Microsoft.Network/networkInterfaces@2022-01-01' = {
  name: '${vmName}-NIC01'
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
    maxShares: 2

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


resource vm 'Microsoft.Compute/virtualMachines@2022-03-01' = {
  name: vmName
  location: location
  properties: {
    proximityPlacementGroup: {
      id: PPG.id
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
        publisher: 'MicrosoftWindowsServer'
        offer: 'WindowsServer'
        sku: OSVersion
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

        //caching: 'None'
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
          id: nic.id
        }
      ]
    }
    diagnosticsProfile: {
      bootDiagnostics: {
        enabled: false
      }
    }
  }
}

resource nic2 'Microsoft.Network/networkInterfaces@2022-01-01' = {
  name: '${vmName2}-NIC02'
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
}

resource vm2 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: vmName2
  dependsOn: [
    vm
  ]
  location: location
  properties: {
    proximityPlacementGroup: {
      id: PPG.id
    }
    hardwareProfile: {
      vmSize: vmSize
    }
    osProfile: {
      computerName: vmName2
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
        name: '${vmName2}-OSdisk'
        managedDisk: {
          storageAccountType: 'StandardSSD_LRS'
        }
      }
      dataDisks: [for (disk, lun) in datasizes: {
        lun: lun+2

        //caching: 'None'
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
          id: nic2.id
        }
      ]
    }
    diagnosticsProfile: {
      bootDiagnostics: {
        enabled: false
      }
    }
  }
}

```
