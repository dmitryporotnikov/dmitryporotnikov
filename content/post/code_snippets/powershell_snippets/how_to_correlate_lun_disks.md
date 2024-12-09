---
title: "How to correlate azure disk object with actual disk within the VM?"
summary: "Guide with pictures"
date: '2024-12-09T00:00:00+00:00'
draft: false
---

# How to correlate azure disk object with actual disk within the VM?

You can easily do so by looking at LUN ID. On azure portal, check those values:

![View on Azure portal](https://cdn.porotnikov.com/githubpages/2024/picture_disk_luns.png "View on Azure portal")


And inside the VM, in disk management blade, check disk properties:

![Picture disk properties](https://cdn.porotnikov.com/githubpages/2024/picture_disk_properties.png)


Look at the LUN id within the OS:

![Picture disk properties](https://cdn.porotnikov.com/githubpages/2024/picture_disk_properties_lunid.png)

The values from the portal and the OS should match and will help you to identify the correct drive. 

You can also use the script below, if you are not a big fan of clicking to see location and drive letter of each drive:

```powershell

$disks = Get-CimInstance -Namespace root\Microsoft\Windows\Storage -ClassName MSFT_Disk
foreach ($disk in $disks) {

    $lun = $null
    if ($disk.Location -match "SCSI\((\d+),(\d+),(\d+)\)") {
        $lun = [int]$matches[3]
    }

    $partitions = Get-CimInstance -Namespace root\Microsoft\Windows\Storage -ClassName MSFT_Partition |
                  Where-Object { $_.DiskNumber -eq $disk.Number }

    $driveLetters = $partitions | ForEach-Object {
        if ($_.AccessPaths) {
            foreach ($path in $_.AccessPaths) {
                if ($path -match "^[A-Z]:\\$") {
                    $path.Substring(0,2)
                }
            }
        }
    }

    [PSCustomObject]@{
        DiskNumber    = $disk.Number
        FriendlyName  = $disk.FriendlyName
        Location      = $disk.Location
        DriveLetters  = ($driveLetters -join ', ')
    }
}


```

The output should look like this:
![Powershell output](https://cdn.porotnikov.com/githubpages/2024/disk_lun_ps_output.png)

## Linux

Linux is much easier:

```shell

[root@rhel89latestminor ~]# dmesg | grep "Attached SCSI"
[    9.511375] sd 1:0:0:2: [sdd] Attached SCSI disk
[    9.514221] sd 0:0:0:1: [sda] Attached SCSI disk
[    9.522442] sd 1:0:0:0: [sdc] Attached SCSI disk
[    9.545461] sd 1:0:0:1: [sdb] Attached SCSI disk
[    9.620817] sd 0:0:0:0: [sde] Attached SCSI disk

```

or

```shell

[root@rhel89latestminor ~]# lsscsi
[0:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sde
[0:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sda
[1:0:0:0]    disk    Msft     Virtual Disk     1.0   /dev/sdc
[1:0:0:1]    disk    Msft     Virtual Disk     1.0   /dev/sdb
[1:0:0:2]    disk    Msft     Virtual Disk     1.0   /dev/sdd

```

0:0:0:0 -> always OS 
0:0:0:1 -> always temp
1:0.0.X -> always Data, where X = Lun number

