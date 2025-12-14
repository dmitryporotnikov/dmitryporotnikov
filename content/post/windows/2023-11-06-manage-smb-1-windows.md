---
id: 712
title: 'Current configuration options for SMB 1.0 in Windows Environments'
summary: 'Current configuration options for SMB 1.0 in Windows Environments'
date: '2023-11-06T11:38:12+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=712'
permalink: /2023/11/06/manage-smb-1-windows/
categories:
    - Windows
---

## Current Configuration Options for SMB 1.0 in Windows Environments

SMB 1.0 is an older protocol version that, despite its vulnerabilities, remains necessary for compatibility with certain legacy systems. Due to these security risks, modern Windows installations have moved away from SMB 1.0 in favor of its successors, SMB 2.0 and 3.0. Managing SMB 1.0 in modern Windows environments requires understanding the tools available for enabling or disabling it. Here’s a look at these tools and how they interact:

- **Windows Feature Toggle**: Controls the installation of SMB 1.0. If removed, the server cannot use SMB 1.0 at all.

![](https://cdn.porotnikov.com/media/2023/11/06112746/image-1-1024x712.jpg)

- **PowerShell’s Set-SmbServerConfiguration**: This cmdlet enables or disables SMB 1.0 on a server by modifying a specific registry key. ([PowerShell Comandlet Documentation](https://learn.microsoft.com/en-us/powershell/module/smbshare/set-smbserverconfiguration?view=windowsserver2022-ps))

- **Registry Settings**: Directly toggling the registry key related to SMB 1.0 offers the same enable/disable control as the PowerShell cmdlet. ([Microsoft Learn](https://learn.microsoft.com/en-us/windows-server/storage/file-server/troubleshoot/detect-enable-and-disable-smbv1-v2-v3?tabs=server#registry-editor))

### Key Interactions:

- **Registry and PowerShell**: Both methods affect the same registry key. A change by one will be represented in the other.

![](https://cdn.porotnikov.com/media/2023/11/06112930/image-1-1-1024x491.jpg)

- **SMB 1.0 Uninstalled**: If uninstalled, attempts by a client to use SMB 1.0 will lead to a “Connection Reset” response from the server.

![](https://cdn.porotnikov.com/media/2023/11/06113148/image-1-3-1024x466.jpg)

- **Installed SMB 1.0 with Modern Windows**: When SMB 1.0 is installed and both client and server support newer versions, Windows will default to the highest version available for security and efficiency.

![](https://cdn.porotnikov.com/media/2023/11/06113210/image-1-4-1024x734.jpg)

Please note, that SMB 1.0 is susceptible to various types of cyber attacks. The most notorious of these is the exploit known as EternalBlue, which was used by the WannaCry ransomware to spread across networks globally. This vulnerability can allow attackers to execute arbitrary code on the target system.  

SMB 1.0 also does not support encryption, which means data is transferred in plaintext over the network. This allows potential eavesdroppers to intercept and read the data or modify it in transit, leading to man-in-the-middle attacks.

I strongly advise against using SMB 1.0 and recommend upgrading to SMB 2 or SMB 3, which provide more secure authentication, encryption, and performance enhancements.
