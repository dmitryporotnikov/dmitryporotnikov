---
id: 738
title: 'How to install WinDBG preview (Windows Store version) on Windows Server?'
summary: 'How to install WinDBG preview (Windows Store version) on Windows Server?'
date: '2024-01-30T11:20:24+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=738'
permalink: /2024/01/30/install-windbg-windows-server-no-windows-store/
categories:
    - Troubleshooting
    - Windows
---

## How to install WinDBG preview (Windows Store version) on Windows Server?

Windows Debugger is an essential tool for developers and IT professionals working with Windows environments. However, if you’re using Windows Server, you might encounter a unique challenge: WinDbg preview is typically available as a Windows Store package, but Windows Server installations do not include the Windows Store. This guide will walk you through an alternative method to install WinDbg on your Windows Server.

1. The first step is to visit the official WinDbg documentation page at [Microsoft’s WinDbg Guide](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/). On this page, you will find the WinDbg download, which comes as a Windows Store package. Since we can’t use the Windows Store, we’ll need to find an alternative way to obtain the WinDbg files.

   ![WinDbg Image](https://cdn.porotnikov.com/media/2024/01/30111628/image-1024x775.png)
   
2. Open the .appinstaller file in Notepad or any text editor of your choice. Within this file, you will find a URL linking directly to the .msixbundle file. This file is the actual package containing the WinDbg software. Download it.

   ![AppInstaller](https://cdn.porotnikov.com/media/2024/01/30111724/image-1.png)

3. Before you can install any applications not sourced from the Windows Store, you need to enable Developer Mode on your Windows Server. This can be done by navigating to the Settings menu, selecting ‘Update & Security’, and then choosing ‘For developers’. Here, you can switch on the ‘Developer Mode’ to allow sideloading of apps.

   ![Developer Mode](https://cdn.porotnikov.com/media/2024/01/30111804/image-2-1024x672.png)

4. After downloading the .msixbundle file from the URL found in step 2, use the Add-AppxPackage command in PowerShell to install the package. This can be done by opening PowerShell as an administrator and executing the command with the path to your downloaded .msixbundle file.

   ![PowerShell Command](https://cdn.porotnikov.com/media/2024/01/30111901/image-3.png)

By following these steps, you can successfully install WinDbg on a Windows Server environment without access to the Windows Store.
