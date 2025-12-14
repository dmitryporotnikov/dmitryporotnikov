---
id: 516
title: "How to Upgrade Legacy Azure Windows VMs Not Supported by Azure's Native In-Place Upgrade Feature"
summary: "How to Upgrade Legacy Azure Windows VMs Not Supported by Azure's Native In-Place Upgrade Feature"
date: '2023-09-27T00:39:31'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=516'
permalink: /2023/09/27/how-to-upgrade-legacy-azure-windows-vms/
categories:
    - Azure
    - Virtualization
    - Windows
---
## How to Upgrade Legacy Azure Windows VMs Not Supported by Azure's Native In-Place Upgrade Feature

Here, I’ll walk you through the process of upgrading a legacy Azure Windows VM without relying on any on-premises solutions or exporting the drive.

![](https://cdn.porotnikov.com/media/2023/09/27002621/image-27.png)

## Prerequisites

- Agreed downtime for the VM
- Basic familiarity with Azure Portal, Hyper-V, and PowerShell

## Step 1: Create an OS Disk Snapshot

1. Stop the VM.
2. Create a snapshot of the OS disk.

![](https://cdn.porotnikov.com/media/2023/09/27002742/image-28.png)

## Step 2: Create a Disk from the Snapshot

1. Navigate to the snapshot you’ve just created.
2. Create a disk from this snapshot.

![](https://cdn.porotnikov.com/media/2023/09/27002753/image-29.png)

## Step 3: Deploy a Hyper-V VM

Ensure that the VM SKU you choose supports nested virtualization. For this example, I’m using `Standard_F32s_v2`. You can verify SKU support for this feature [here](https://learn.microsoft.com/en-us/azure/virtual-machines/fsv2-series).

![](https://cdn.porotnikov.com/media/2023/09/27002902/image-31.jpg)

If you’re not using Bicep or ARM templates, consider adopting them. :)

![](https://cdn.porotnikov.com/media/2023/09/27003034/image-31.png)

## Step 4: Attach the Disk

1. Go to your newly provisioned Hyper-V VM.
2. Attach the disk you created from the snapshot.

![](https://cdn.porotnikov.com/media/2023/09/27003054/image-32-1024x460.png)

## Step 5: Install Hyper-V Role

1. RDP into your VM.
2. Open PowerShell as an administrator and run the following command:

```shell
Install-WindowsFeature -Name Hyper-V -IncludeManagementTools -Restart
```

The server will reboot automatically upon completion.

## Step 6: Prepare the Disk for Upgrade

1. After reboot, open Disk Management.
2. Bring the attached disk offline.

![](https://cdn.porotnikov.com/media/2023/09/27003131/image-33.png)

## Step 7: Create a New Virtual Machine in Hyper-V

- Open Hyper-V Manager.
- Create a new virtual machine.

![](https://cdn.porotnikov.com/media/2023/09/27003209/image-34.png)

- Specify the generation for your new VM (most likely Generation 1 for legacy workloads).

![](https://cdn.porotnikov.com/media/2023/09/27003219/image-35.png)

- Select “I will install the operating system later.”
- Add the existing physical hard disk as the VM’s disk.

![](https://cdn.porotnikov.com/media/2023/09/27003305/image-36.png)

## Step 8: Boot the VM

Start the VM. You should see your legacy workload booting up.

![](https://cdn.porotnikov.com/media/2023/09/27003326/image-37.png)

## Step 9: Perform the Upgrade

For this example, I’m using ISOs from a Visual Studio subscription. Your organization may have its own ISO repository. The upgrade path for a Windows Server 2008 R2, in this case, would be 2008 R2 -> 2012 R2 -> 2016. Note that you can’t skip versions during an in-place upgrade.

![](https://cdn.porotnikov.com/media/2023/09/27003352/image-38.png)

### Download the ISOs

You can download the ISOs directly to your server using a script like this:

```powershell
if (-Not (Test-Path "C:\Temp")) {
    New-Item -Path "C:\Temp" -ItemType Directory
}
$url = "https://myvs.download.prss.microsoft.com/dbazure/en_windows_server_2012_r2_x64_dvd_2707946.iso?MYTOKEN"
$destination = "C:\Temp\en_windows_server_2012_r2_x64_dvd_2707946.iso"

Invoke-WebRequest -Uri $url -OutFile $destination

if (Test-Path $destination) {
    Write-Host "File downloaded successfully to $destination"
} else {
    Write-Host "File download failed"
}
```

### Proceed with the Upgrade

- Attach the downloaded ISO to your VM.

![](https://cdn.porotnikov.com/media/2023/09/27003501/image-39.png)

- Follow the on-screen instructions to upgrade.

![](https://cdn.porotnikov.com/media/2023/09/27003510/image-40.png)
![](https://cdn.porotnikov.com/media/2023/09/27003524/image-41.png)

## Step 10: Finalize the Upgrade

- Shut down the VM gracefully.

![](https://cdn.porotnikov.com/media/2023/09/27003602/image-42.png)

- Detach the upgraded drive from the Hyper-V VM.

![](https://cdn.porotnikov.com/media/2023/09/27003657/image-43-1024x471.png)

- Swap the OS disk on your original Azure VM with the upgraded disk.

![](https://cdn.porotnikov.com/media/2023/09/27003716/image-44.png)
![](https://cdn.porotnikov.com/media/2023/09/27003745/image-45.png)

- Start the VM and perform basic smoke tests.

![](https://cdn.porotnikov.com/media/2023/09/27003754/image-46-1024x279.png)
![](https://cdn.porotnikov.com/media/2023/09/27003806/image-47-1024x426.png)

Congratulations, you’ve successfully upgraded your legacy Azure Windows VM!
