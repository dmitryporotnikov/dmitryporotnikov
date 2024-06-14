---
id: 472
title: 'In Place Upgrade Your Windows Server Azure VM: Step by Step Guide'
summary: 'In Place Upgrade Your Windows Server Azure VM: Step by Step Guide'
date: '2023-09-24T16:28:12+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=472'
permalink: /2023/09/24/windows-server-azure-vm-in-place-upgrade/

categories:
    - Azure
    - Windows
---

## In Place Upgrade Your Windows Server Azure VM: Step by Step Guide

Upgrading your Windows Server Azure VM to a newer version can seem daunting, but with the right steps, it can be a smooth process. This guide will walk you through a step-by-step process, complete with visuals, to upgrade your Windows VM in Azure, for instance, from WS 2016 to 2019.

## Prerequisites for a Successful Upgrade

Before diving into the upgrade, ensure the following prerequisites are met:

- **Managed Disk Usage**  
Your VM should be using a managed disk. You can verify this via the Azure portal in the JSON view. If your VM isn’t using a managed disk, you can migrate to one.

![Managed Disk](https://cdn.porotnikov.com/media/2023/09/24235645/image-2-1024x335.png)

- **Volume Licensing Configuration**  
Your VM should be set up to use volume licensing as a KMS client. To check this:
  - Within the VM, run the command: `slmgr/dlv`
  - The description should display: `VOLUME_KMSCLIENT channel`

![Volume Licensing](https://cdn.porotnikov.com/media/2023/09/24235639/image-3.jpg)

- **Sufficient Disk Space**  
Ensure that the VM OS drive has ample disk space for an in-place upgrade.

- **Snapshot Precaution**  
Before proceeding with the upgrade, take a snapshot of the OS drive. This acts as a safety net, allowing you to roll back if anything goes south.

![Snapshot Example](https://cdn.porotnikov.com/media/2023/09/24235634/image-3-1024x332.png)

## Upgrade Process

Since Azure doesn’t provide access to the Virtual Machine ISO mounting feature at the hypervisor level, you’ll need to create a managed disk with the upgrade media. Use the following script, sourced from [Microsoft’s official documentation](https://learn.microsoft.com/en-us/azure/virtual-machines/windows-in-place-upgrade):

```powershell
# Login to Azure
Connect-AzAccount

# Set the subscription context
$subscriptionId = "YOUR SUB ID"
Set-AzContext -SubscriptionId $subscriptionId

# Resource group of the source VM
$resourceGroup = "LETSUPGRADE"

# Location of the source VM
$location = "North Europe"

# Zone of the source VM, if any
$zone = "" 

# Disk name for the that will be created
$diskName = "WindowsServer2022UpgradeDisk"

# Target version for the upgrade - must be either server2022Upgrade or server2019Upgrade
$sku = "server2022Upgrade"

# Common parameters
$publisher = "MicrosoftWindowsServer"
$offer = "WindowsServerUpgrade"
$managedDiskSKU = "Standard_LRS"

# Get the latest version of the special (hidden) VM Image from the Azure Marketplace
$versions = Get-AzVMImage -PublisherName $publisher -Location $location -Offer $offer -Skus $sku | sort-object -Descending {[version] $_.Version}
$latestString = $versions[0].Version

# Create Resource Group if it doesn't exist
if (-not (Get-AzResourceGroup -Name $resourceGroup -ErrorAction SilentlyContinue)) {
    New-AzResourceGroup -Name $resourceGroup -Location $location    
}

# Create Managed Disk from LUN 0
if ($zone){
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU -CreateOption FromImage -Zone $zone -Location $location
} else {
    $diskConfig = New-AzDiskConfig -SkuName $managedDiskSKU -CreateOption FromImage -Location $location
} 
Set-AzDiskImageReference -Disk $diskConfig -Id $image.Id -Lun 0
New-AzDisk -ResourceGroupName $resourceGroup -DiskName $diskName -Disk $diskConfig
```

After running the script, you should see the disk created in the resource group you specified:

![Created Disk](https://cdn.porotnikov.com/media/2023/09/24235628/image-4.png)

Attach the upgrade disk to the virtual machine. If you’re unable to view the drive you’re trying to attach, consider clicking “Refresh” button on the Disks view:

![Attach Disk](https://cdn.porotnikov.com/media/2023/09/24235623/image-5-1024x586.png)

Connect to the VM using RDP. You should now see the upgrade disk in the file explorer.

![RDP View](https://cdn.porotnikov.com/media/2023/09/24235621/image-6.png)

Run the installer with the following parameters, as recommended by Microsoft: `.\setup.exe /auto upgrade /dynamicupdate disable`

![Installer](https://cdn.porotnikov.com/media/2023/09/24235620/image-7.png)

Follow the installer prompts and wait for your system to transition to install mode. This is a good time to take a break or start the installation on another server.

![Installation Progress](https://cdn.porotnikov.com/media/2023/09/24235619/image-8-1024x433.png)
![Progress View](https://cdn.porotnikov.com/media/2023/09/24235618/image-9.png)

After the RDP session disconnects, monitor the installation progress using Azure boot diagnostics.

![Boot Diagnostics](https://cdn.porotnikov.com/media/2023/09/24235616/image-10.png)

Once the boot diagnostic displays the login screen:

![Login Screen](https://cdn.porotnikov.com/media/2023/09/24235615/image-11.jpg)

Reconnect via RDP and confirm the upgrade’s success by running the `winver` command.

## Post-Upgrade Cleanup

- **Disk Cleanup**  
Use the disk cleanup utility to remove the `windows.old` folder located in the root of your C:\\ drive.

![Disk Cleanup](https://cdn.porotnikov.com/media/2023/09/24235614/image-11.png)

- **Upgrade Media**  
Detach the upgrade media and delete the managed disk you created with the script.

- **Snapshot**  
Now that your upgrade is complete and verified, you can delete the OS disk snapshot you created earlier.

## A Note on Image Plan

Post-upgrade, you might notice that your Image Plan in the Azure Portal remains unchanged. This is purely cosmetic and reflects the image from which the VM was originally deployed.

![Image Plan](https://cdn.porotnikov.com/media/2023/09/24235613/image-12.png)

As you see upgrading the Windows Server Azure VM in place is pretty straightforward process. By following the steps outlined in this guide, you can safely and easily upgrade your VM to a new version. And if you encounter any problems with the upgrade, you can always restore your VM from the snapshot you created before starting the upgrade. You’ve created the snapshot, right? 😉
