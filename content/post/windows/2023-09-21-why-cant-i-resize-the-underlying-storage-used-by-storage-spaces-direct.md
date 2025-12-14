---
id: 403
title: "Why can't I resize the underlying storage used by Storage Spaces?"
summary: "Why can't I resize the underlying storage used by Storage Spaces?"
date: '2023-09-21T22:56:14+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=403'
permalink: /2023/09/21/why-cant-i-resize-the-underlying-storage-used-by-storage-spaces-direct/
categories:
    - Azure
    - Windows
---

## Why can't I resize the underlying storage used by Storage Spaces?

Storage Spaces is a Windows Server storage technology that pools the storage capacity of multiple disks into a single virtual disk. This technology is often used to create highly available and scalable storage solutions for virtual machines and other workloads.

One of the limitations of Storage Spaces is that it does not support resizing the underlying storage. This means that if you need to add more storage to a Storage Spaces, you must do so by adding new drives.

If you need to add more storage to a Storage Spaces, the best practice is to follow the instructions in the [Microsoft documentation](https://learn.microsoft.com/en-us/azure-stack/hci/manage/manage-volumes?toc=%2Fwindows-server%2Fstorage%2Ftoc.json&bc=%2Fwindows-server%2Fbreadcrumbs%2Ftoc.json&tabs=windows-admin-center#expand-volumes). This will ensure that your data is protected and that your storage performance is not impacted.  
  
To check if a disk is a member of a Storage Spaces, you can use the following **Diskpart** command:

```shell
diskpart
select disk <number from Disk management console>
detail disk
```

If the type of the device is “Microsoft Storage Spaces Device”, then the disk is a member of a Storage Spaces.

![screenshot](https://cdn.porotnikov.com/media/2023/09/24235651/image-1-1024x388.png)
**Why Resizing Isn’t Supported:** Microsoft’s official documentation clearly states that resizing the underlying storage used by Storage Spaces is not supported. This is especially true for those running Storage Spaces in virtualized storage environments, including Azure. Attempting to resize or alter the characteristics of the storage devices used by the virtual machines can lead to data becoming inaccessible.
