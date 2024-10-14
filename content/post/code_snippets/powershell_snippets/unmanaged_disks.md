---
title: 'Create VM with unmanaged disks, attach unmanaged data disks, get the Azure Resource Graph report on it'
summary: 'Create VM with unmanaged disks, attach unmanaged data disks, get the Azure Resource Graph report on it'
date: '2024-10-14T00:00:00+00:00'
---

## Unmanaged disks are going away

If you're using Azure unmanaged disks, it's time to start planning your migration to managed disks. As of January 2024, Azure will no longer allow new unmanaged disks, and by September 2025, any virtual machines (VMs) still running on unmanaged disks will be stopped and deallocated. Managed disks bring several advantages such as improved reliability, scalability, and performance, and they remove the need to manage storage accounts manually.

To assist with this process, this post will show you how to create a VM with unmanaged disks, attach data disks, and generate a report using Azure Resource Graph to identify unmanaged disks across your Azure subscriptions.

## Create VM with unmanaged disks

```powershell
# Set variables
$location = "polandcentral"
$rgName = "unmanageddisks"

# Create a resource group
New-AzResourceGroup -Name $rgName -Location $location

# Create a subnet configuration
$subnetConfig = New-AzVirtualNetworkSubnetConfig -Name "mySubnet" -AddressPrefix "192.168.1.0/24"

# Create a virtual network
$vnet = New-AzVirtualNetwork -ResourceGroupName $rgName -Location $location `
    -Name "MyVNET" -AddressPrefix "192.168.0.0/16" -Subnet $subnetConfig

# Create a public IP address
$pip = New-AzPublicIpAddress -ResourceGroupName $rgName -Location $location `
    -AllocationMethod Static -IdleTimeoutInMinutes 4 -Name "mypublicdns$(Get-Random)"

# Create network security group rules
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name "myNetworkSecurityGroupRuleRDP" -Protocol Tcp `
    -Direction Inbound -Priority 1000 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 3389 -Access Allow

$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name "myNetworkSecurityGroupRuleWWW" -Protocol Tcp `
    -Direction Inbound -Priority 1001 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix * `
    -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $rgName -Location $location `
    -Name "myNetworkSecurityGroup" -SecurityRules $nsgRuleRDP,$nsgRuleWeb

# Create a virtual network card and associate with public IP address and NSG
$nic = New-AzNetworkInterface -Name "myNic" -ResourceGroupName $rgName -Location $location `
    -SubnetId $vnet.Subnets[0].Id -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id

# Define a credential object
$cred = Get-Credential

# VM configuration
$vmName = "myVM"
$vmSize = "Standard_DS2_v2"
$vm = New-AzVMConfig -VMName $vmName -VMSize $vmSize

# Set VM operating system
$vm = Set-AzVMOperatingSystem -VM $vm -Windows -ComputerName $vmName -Credential $cred

# Set VM source image
$vm = Set-AzVMSourceImage -VM $vm -PublisherName "MicrosoftWindowsServer" `
    -Offer "WindowsServer" -Skus "2019-Datacenter" -Version "latest"

# Add the network interface to the VM
$vm = Add-AzVMNetworkInterface -VM $vm -Id $nic.Id

# Create a new storage account for unmanaged disks
$storageAccountName = "storacc" + (Get-Random -Minimum 100000 -Maximum 999999)
New-AzStorageAccount -ResourceGroupName $rgName -Name $storageAccountName `
    -Location $location -SkuName Standard_LRS -Kind StorageV2

# Get the storage account key and create a context
$storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $rgName -Name $storageAccountName).Value[0]
$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

# Configure the OS disk as unmanaged
$osDiskName = "$vmName-OSDisk"
$osDiskUri = $storageContext.BlobEndPoint + "vhds/" + $osDiskName + ".vhd"
$vm = Set-AzVMOSDisk -VM $vm -Name $osDiskName -VhdUri $osDiskUri -CreateOption FromImage

# Create the virtual machine with unmanaged disk
New-AzVM -ResourceGroupName $rgName -Location $location -VM $vm
```

## Attach 3 data disks to it

```powershell
# Set variables
$rgName = "unmanageddisks2"
$vmName = "myVM"
$location = "polandcentral"
$storageAccountName = "storacc954938" # Replace with your actual storage account name
$containerName = "vhds"
$numberOfDisks = 3

# Get the existing VM
$vm = Get-AzVM -ResourceGroupName $rgName -Name $vmName

# Get storage account key and create context
$storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $rgName -Name $storageAccountName).Value[0]
$storageContext = New-AzStorageContext -StorageAccountName $storageAccountName -StorageAccountKey $storageAccountKey

# Create container if it doesn't exist
New-AzStorageContainer -Name $containerName -Context $storageContext -ErrorAction SilentlyContinue

# Add data disks
for ($i = 1; $i -le $numberOfDisks; $i++) {
    $diskName = "$vmName-DataDisk-$i"
    $diskSize = 128 # Size in GB
    $diskUri = $storageContext.BlobEndPoint + "$containerName/$diskName.vhd"

    Add-AzVMDataDisk -VM $vm -Name $diskName -VhdUri $diskUri -CreateOption Empty -DiskSizeInGB $diskSize -Lun $i
}

# Update the VM with the new disks
Update-AzVM -ResourceGroupName $rgName -VM $vm

Write-Output "Added $numberOfDisks unmanaged data disks to $vmName"
```
To manage your migration effectively, it’s crucial to identify all VMs that are still using unmanaged disks. Azure Resource Graph (ARG) allows you to generate a report of VMs running on unmanaged disks. The provided query helps you identify these VMs, showing details like the VM name, OS disk, and data disk URIs. This information is critical to ensure that no VM is left behind in the migration process.
## Get a report from ARG

```kusto
resources
| where type == "microsoft.compute/virtualmachines"
| where properties.storageProfile.osDisk.managedDisk == ""
| extend osDiskName = properties.storageProfile.osDisk.name
| extend osDiskUri = properties.storageProfile.osDisk.vhd.uri
| project vmName = name, osDiskName, osDiskUri, resourceGroup
| union (
    resources
    | where type == "microsoft.compute/virtualmachines"
    | mvexpand dataDisk = properties.storageProfile.dataDisks
    | where dataDisk.managedDisk == ""
    | extend dataDiskName = dataDisk.name
    | extend dataDiskUri = dataDisk.vhd.uri
    | project vmName = name, dataDiskName, dataDiskUri, resourceGroup
)
```
