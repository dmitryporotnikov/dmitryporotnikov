---
id: 373
title: 'PowerShell: Create a NAT VMSwitch on Hyper-V'
summary: 'PowerShell: Create a NAT VMSwitch on Hyper-V'
date: '2023-09-18T09:38:37+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=373'
permalink: /2023/09/18/powershell-create-a-nat-vmswitch-on-hyper-v/
categories:
    - Code
    - Virtualization
    - Windows
---
## PowerShell: Create a NAT VMSwitch on Hyper-V

<https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/user-guide/setup-nat-network>  
  
Live migration in clustered scenario won’t work for VMs using this NAT.

Quick migration will work.

```powershell
#Clear previous messages
clear

# Define the NAT Switch Name and NAT Configuration Parameters
$switchName = "NATSwitch"
$natSubnet = "192.168.100.0/24"
$gatewayAddress = "192.168.100.1"
$netAdapaterName = "vEthernet ("+$switchName+")"

# Create a new virtual switch
New-VMSwitch -SwitchName $switchName -SwitchType Internal

# Retrieve the actual Network Adapter for the created switch
$netAdapter = Get-NetAdapter -Name $netAdapaterName


# Set the IP address on the NAT Gateway (the virtual switch)
New-NetIPAddress -IPAddress $gatewayAddress -PrefixLength 24 -InterfaceIndex $netAdapter.ifIndex

# Create the NAT rule itself
New-NetNat -Name "$switchName" -InternalIPInterfaceAddressPrefix $natSubnet

#Check Status
Get-NetNat
Get-VMSwitch
```

If you need a NAT rule to RDP to the VM:

```powershell
$natName = "NATSwitch"           
$externalPort = 33389            
$internalPort = 3389            
$internalIPAddress = "192.168.100.10"  #Replace with guest OS IP

Add-NetNatStaticMapping -NatName $natName -Protocol TCP -ExternalIPAddress 0.0.0.0 -ExternalPort $externalPort -InternalIPAddress $internalIPAddress -InternalPort $internalPort
```
