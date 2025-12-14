---
id: 766
title: 'Encountering issues with Azure IAAS Virtual Machines not joining Azure AD post-deployment from a sysprepped template?'
summary: 'Encountering issues with Azure IAAS Virtual Machines not joining Azure AD post-deployment from a sysprepped template?'
date: '2024-02-02T11:09:15+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=766'
permalink: /2024/02/02/azure-vm-deployment-azure-ad-issue/
categories:
    - Azure
    - Windows
---

## Encountering issues with Azure IAAS Virtual Machines not joining Azure AD post-deployment from a sysprepped template?

Sysprep is designed to prepare an installation of Windows for duplication, clearing out specific information like registry keys, certificates, and more to avoid duplication errors. Yet, it doesn’t entirely remove all data, especially those related to Azure AD/Microsoft Entra ID or Intune enrollments. These services attach certificates and store various information across the registry, which sysprep overlooks.

Cloning a device already joined to AzureAD/Microsoft Entra ID or enrolled in Intune creates identical digital twins from Azure’s AAD/Entra perspective, making it impossible for Azure AD/Intune to differentiate between them due to identical device IDs and certificates.

It is actually documented here:  
[Sysprep will not run correctly on a Windows 10 device that has been MDM enrolled – Intune | Microsoft Learn](https://learn.microsoft.com/en-us/troubleshoot/mem/intune/device-enrollment/troubleshoot-sysprep-windows-10-device-enrolled-mdm)

**Solution?**

Solution involves running **‘dsregcmd /leave’** to disassociate the device from Azure AD/Microsoft Entra ID, followed by deleting the registry key under ‘KEY\_LOCAL\_MACHINE\\SOFTWARE\\Microsoft\\Windows Azure\\CurrentVersion\\AADLoginForWindowsExtension’ before sysprepping. This process essentially resets the device’s state, allowing for successful image creation.

Alternatively, a more preventative approach is to refrain from joining machines to Azure AD/Microsoft Entra ID if they are intended to serve as deployment templates.