---
id: 99
title: 'Bicep: Basic Windows Server deployment template, no Public IP, no NSG, deployment in existing network.'
summary: 'Bicep: Basic Windows Server deployment template, no Public IP, no NSG, deployment in existing network.'
date: '2023-08-23T10:13:46+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=99'
permalink: /2023/08/23/bicep-basic-windows-server-deployment-template-no-public-ip-no-nsg-deployment-in-existing-network/
categories:
    - Bicep
---

## Bicep: Basic Windows Server deployment template, no Public IP, no NSG, deployment in existing network

```bicep
@description('Name of the virtual machine.')
@maxLength(15)
param vmName string = 'NAMEYOURVM'

@description('Username for the Virtual Machine.')
param adminUsername string = 'azureuser'

@description('Password for the Virtual Machine.')
@minLength(12)
param adminPassword string

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
        'https://YOURSA.blob.core.windows.net/scripts/EnableAnsibleWinRM.ps1'
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

resource vm 'Microsoft.Compute/virtualMachines@2021-03-01' = {
  name: vmName
  location: location
  properties: {
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
```
