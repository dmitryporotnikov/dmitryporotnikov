---
id: 684
title: 'PowerShell: Script-Based Workaround for Renaming Azure VMs and Switching to Unsupported Sizes'
date: '2023-10-10T14:48:05+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=684'
permalink: /2023/10/10/bypassing-azure-vm-limitations-script-based-solution/
categories:
    - Azure
    - Code
    - Troubleshooting
    - Virtualization
---
# PowerShell: Script-Based Workaround for Renaming Azure VMs and Switching to Unsupported Sizes

Azure is great, however, as with any platform, Azure is not without its limitations, e.g:

- Inability to change the name of an existing Azure VM.
- Constraints when resizing VMs, specifically: 
    - You can’t resize a VM size that has a local temp disk to a VM size without one, and vice versa.

The only combinations allowed for resizing are:

1. VM (with no local temp disk) ➔ VM (with no local temp disk)
2. VM (with local temp disk) ➔ VM (with local temp disk)

### Workaround

Below is a PowerShell script that allows you to circumvent the VM name and size restrictions while keeping the disk, network interface card, and extensions in place.  
This script removes the existing VM and creates a new one with the parameters you specify. This action ensures that all the attached resources, like disks and NICs, remain unchanged.

### Script

```powershell
Write-Host "Logging into Azure account..."
Login-AzAccount

Write-Host "Setting Azure subscription context..."
Set-AzContext -Subscription "YOUR SUB ID"

# Define variables. Keep the same name if you want to get rid of storage profile reference in VM view, or just change VM size, e.g. VM (with no local temp disk) -> VM (with local temp disk).
Write-Host "Defining variables for Resource Group and VM names..."
$resourceGroup = "RENAMERG"
$oldVmName = "BADNAME"
$newVmName = "GOODNAME"

Write-Host "Fetching the existing VM object..."
$vm = Get-AzVM -ResourceGroupName $resourceGroup -Name $oldVmName

Write-Host "Saving key properties from the VM object..."
$vmSize = $vm.HardwareProfile.VmSize #Change VM SKU Size here if needed, e.g Standard_B2ms
$osDisk = $vm.StorageProfile.OsDisk
$dataDisks = $vm.StorageProfile.DataDisks
$nic = $vm.NetworkProfile.NetworkInterfaces[0].Id
$osType = $vm.StorageProfile.OsDisk.OsType

Write-Host "Fetching extensions from the old VM..."
$vmExtensions = Get-AzVMExtension -ResourceGroupName $resourceGroup -VMName $oldVmName

Write-Host "Removing the old VM but retaining all attached resources..."
Remove-AzVM -ResourceGroupName $resourceGroup -Name $oldVmName -Force

Write-Host "Creating a new VM configuration..."
$vmConfig = New-AzVMConfig -VMName $newVmName -VMSize $vmSize

Write-Host "Setting the OS disk and type..."
if ($osType -eq 'Windows') {
    $vmConfig = Set-AzVMOSDisk -VM $vmConfig -CreateOption Attach -ManagedDiskId $osDisk.ManagedDisk.Id -Name $osDisk.Name -Windows
} else {
    $vmConfig = Set-AzVMOSDisk -VM $vmConfig -CreateOption Attach -ManagedDiskId $osDisk.ManagedDisk.Id -Name $osDisk.Name -Linux
}

Write-Host "Adding the network interface..."
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic

Write-Host "Checking and adding any data disks..."
foreach ($dataDisk in $dataDisks) {
    $disk = Get-AzDisk -ResourceGroupName $resourceGroup -DiskName $dataDisk.Name
    if ($disk.Id -eq $dataDisk.ManagedDisk.Id) {
        $vmConfig = Add-AzVMDataDisk -VM $vmConfig -Name $dataDisk.Name -ManagedDiskId $dataDisk.ManagedDisk.Id -Caching $dataDisk.Caching -Lun $dataDisk.Lun -DiskSizeInGB $dataDisk.DiskSizeGB -CreateOption Attach
    } else {
        Write-Host ("Disk name " + $dataDisk.Name + " does not match with given managed disk ID.")
    }
}

$vmConfig = Set-AzVMBootDiagnostic -VM $vmConfig -Disable

Write-Host "Creating the new VM..."
New-AzVM -ResourceGroupName $resourceGroup -Location $vm.Location -VM $vmConfig

Write-Host "Adding extensions to the new VM..."
$newVmName = $vmConfig.Name
foreach ($ext in $vmExtensions) {
    Set-AzVMExtension -ResourceGroupName $resourceGroup -VMName $newVmName -Name $ext.Name -Publisher $ext.Publisher -ExtensionType $ext.ExtensionType -TypeHandlerVersion $ext.TypeHandlerVersion -Settings $ext.Settings
}
```

1. The script starts by logging into your Azure account and setting the subscription context.
2. It fetches the existing VM object and saves essential properties like VM size, OS disk, data disks, NICs, etc.
3. The existing VM is then removed, but all the attached resources are retained.
4. A new VM configuration is created using the saved properties.
5. Lastly, the script reinstalls any extensions you had on your old VM.
