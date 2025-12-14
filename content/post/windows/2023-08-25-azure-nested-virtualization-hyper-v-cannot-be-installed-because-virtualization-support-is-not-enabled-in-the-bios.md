---
id: 173
title: 'Azure Nested Virtualization: Hyper-V cannot be installed because virtualization support is not enabled in the BIOS.'
summary: 'Azure Nested Virtualization: Hyper-V cannot be installed because virtualization support is not enabled in the BIOS.'
date: '2023-08-25T16:54:41+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=173'
permalink: /2023/08/25/azure-nested-virtualization-hyper-v-cannot-be-installed-because-virtualization-support-is-not-enabled-in-the-bios/
categories:
    - Azure
    - Troubleshooting
    - Windows
---

## Azure Nested Virtualization: Hyper-V cannot be installed because virtualization support is not enabled in the BIOS

Nested virtualization is an exciting and powerful feature that allows you to run a virtual machine within another virtual machine. This capability has numerous applications in the fields of development, testing, and security. However, when trying to enable it in Azure, you may encounter a specific error related to nested virtualization:

![Error Screenshot](https://cdn.porotnikov.com/media/2023/08/24235750/image-4-1024x660.png)

### “Hyper-V cannot be installed because virtualization support is not enabled in the BIOS.”

Fortunately, the fix for this problem is relatively straightforward and can be done directly through the Command Prompt. Here’s a step-by-step guide:

1. **Open Command Prompt**
2. **Enter the Command**:

    ```plaintext
    bcdedit /set hypervisorlaunchtype auto
    ```

3. **Restart the Computer**: After executing the command, you’ll need to restart your VM for the changes to take effect. You can do this manually or use the following PowerShell command:

    ```plaintext
    Restart-Computer -Force
    ```

After rebooting, you can check that Hyper-V can be installed as usual:

![Installation Confirmation](https://cdn.porotnikov.com/media/2023/08/24235748/image-5-1024x723.png)
