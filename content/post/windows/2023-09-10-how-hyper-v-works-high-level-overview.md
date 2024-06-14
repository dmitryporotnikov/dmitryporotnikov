---

id: 238
title: 'How Hyper-V Works, High Level Overview'
summary: 'How Hyper-V Works, High Level Overview'
date: '2023-09-10T19:28:13+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=238'
permalink: /2023/09/10/how-hyper-v-works-high-level-overview/
categories:
    - Azure
    - Virtualization
    - Windows

---

## How Hyper-V Works, High Level Overview

Hyper-V stands as Microsoft’s first standalone hypervisor, offering extensive support for the x86-64 architecture. It has established its own space in the virtualization market, going toe-to-toe with competitors like VMware ESXi, Xen, and KVM. One of its distinct advantages is its native integration with Microsoft-centric environments.

Contrary to many Type 2 hypervisors that function atop an existing operating system, Hyper-V is a Type 1 hypervisor that interacts directly with the hardware. This distinction can lead to initial confusion, given that the setup starts with a customary Windows Server installation. However, once Hyper-V is activated as an additional role, the Windows Server OS converts into a privileged partition, coming under the immediate control of the hypervisor after a system reboot.

![image](https://cdn.porotnikov.com/media/2023/09/24235740/image-1024x514.png)

In the nomenclature of Microsoft, guest virtualized operating systems are designated as partitions. The architecture follows a hierarchical model where a privileged root partition holds the reins, governing all other child partitions. This root partition is entrusted with full hypervisor permissions and oversees the configuration and management of lesser-privileged guest partitions. It has access other partition’s physical memory.

“Unenlightened” operating systems (OSes that are unaware of their virtualized environment) can also operate smoothly on Hyper-V. This is accomplished by Hyper-V’s emulation of universally compatible standard devices, allowing these operating systems to function as if they were running on physical hardware.

In contrast, operating systems tailored for Hyper-V can use extra capabilities to boost performance. For example, these specialized systems have the ability to make Hypercalls, allowing them to interact directly with the hypervisor. Hypercalls work through [publicly available Hyper-V APIs](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/tlfs/hypercall-interface) and serve a role similar to traditional system call interfaces present in operating system kernels.

![image](https://cdn.porotnikov.com/media/2023/09/24235739/image2-1024x502.png)
![image](https://cdn.porotnikov.com/media/2023/09/24235737/image3-1024x741.png)

Additionally, the VMBus framework in Hyper-V paves the way for quick and direct data exchanges between the root and child partitions. In default configuration VMBus communication is allowed only between guest and root partitions. This feature is particularly beneficial for synthetic devices, which outperform their emulated equivalents in speed. A noteworthy instance of such an optimized device operating in an enlightened partition is Azure’s [Accelerated Networking feature](https://learn.microsoft.com/en-us/azure/virtual-network/accelerated-networking-overview). We will discuss the difference between emulated vs synthetic devices in “Types of Virtual Devices in Hyper-V” section.

### Memory Virtualization

---

Hyper-V is using EPT (Extended Page Tables) that serves as an additional layer of memory virtualization, enabling more direct and efficient mapping between the guest’s virtual memory and the host’s physical memory. The EPT feature uses its own set of tables (similar to page tables in standard memory management) to map the guest’s physical address space directly to the host’s physical address space. When a memory access occurs, the hardware checks the EPT tables to find the corresponding mapping to the host’s physical memory, without requiring the hypervisor to perform this translation manually. Because the EPT hardware can handle memory translations, the number of calls to the hypervisor for address translation is reduced. This minimizes “context switches” between the hypervisor and guest OS, improving performance.

### Device virtualization

---

Each child partition represented by a corresponding Virtual Machine Worker Process (VMWP). Running as a user-space process in the root partition, VMWP handles management tasks such as snapshots and migrations. It also emulates certain devices and implements synthetic devices that are not performance-critical. While the user-space implementation offers lower privilege levels for security, it’s unsuitable for performance-critical tasks like networking and storage. You will find high level overview of the architecture below. If you are not sure, what User Mode/Kernel Mode and Hypervisor modes are for, [this wiki article](https://en.wikipedia.org/wiki/Protection_ring) explains it very well.

![image](https://cdn.porotnikov.com/media/2023/09/24235729/image5-1024x709.png)

### Virtualization Service Providers (VSP)

---

To present performance-critical devices to the guest OS, Hyper-V employs Virtualization Service Providers. VSPs are kernel drivers running in the root partition.

### Virtualization Infrastructure Driver (VID)

---

The VID acts as a bridge between the hypervisor and the operating system. It manages virtualization resources, including memory management and CPU scheduling. VID uses the Hypercall interface in order to send management commands to the hypervisor, e.g. partition suspend/resume, partition create/delete, device visibility, e.t.c. It also emulates ROMs.

![image](https://cdn.porotnikov.com/media/2023/09/24235723/image7-1024x749.png)

### Windows Hypervisor Interface Library (WinHv)

---

WinHv provides an interface that enables drivers to communicate with the hypervisor using standard Windows calling conventions. It essentially acts as a bridge between an operating system’s drivers and the hypervisor.

### Types of Virtual Devices in Hyper-V

---

#### VDev: The Virtual Device

Virtual Devices, or VDev, represent the virtual hardware stack a Hyper-V VM utilizes. VDev can include various components like network adapters, disk drives, and CPU cores.

#### Emulated vs Synthetic Devices

**Emulated Devices:** These virtual devices mimic real hardware components and are universally supported by most operating systems. However, they consume more CPU resources due to the emulation process, making them slower.

**Synthetic (Enlightened) Devices:** These are optimized for virtualized environments, offering better performance. They communicate directly with the hypervisor via the Virtual Machine Bus (VMBus).

Example of "Enlightened" IO shown below. Source – old Microsoft Technet article.

![image](https://cdn.porotnikov.com/media/2023/09/24235717/old_technet.png)

#### How VDev Functions

---

**Resource Allocation:** When setting up a VM, you can allocate resources by adding various virtual devices managed by VDev.

**Driver Interaction:** The guest OS communicates with these virtual devices via drivers. Emulated devices require standard drivers, while synthetic ones need specialized drivers provided by Integration Services (older OSes) or to be already bundled with the OS.

**Data Transfer:** VDev also oversees data transfer between the guest operating system and virtual devices. For synthetic devices, this happens efficiently through the VMBus.

The Virtualization Service Clients (VSC) in child partitions interact with VSPs. The communication is done via VMBus. While modern Windows versions include these kernel drivers for supporting synthetic devices, Linux and FreeBSD also have open-source VSC implementations, enhancing cross-platform compatibility.

![image](https://cdn.porotnikov.com/media/2023/09/24235712/video1-1024x572.png)
![image](https://cdn.porotnikov.com/media/2023/09/24235703/video2-1024x562.png)

**Source:** A Dive in to Hyper-V Architecture & Vulnerabilities

### Generation1 Vs Generation2 Guests

---

Gen1 is heavily reliant on emulated devices like IDE controllers and legacy network adapters, leading to higher CPU overhead and utilizes legacy BIOS for the booting process.

Gen2 utilizes Unified Extensible Firmware Interface (UEFI) which supports Secure Boot, a feature that helps to prevent unauthorized code execution during the boot process. Uses VMBus directly, resulting in less CPU overhead and better performance and supports booting from a SCSI controller, allowing for larger boot volumes.

### CPU Virtualization

---

Hyper-V takes advantage of AMD-V and Intel VT-x features to partition physical CPUs into multiple virtual CPUs.

Both AMD-V and Intel VT-x are sets of processor extensions that add hardware virtualization support to x86 architecture processors. These technologies facilitate running multiple operating systems on a single physical machine by enhancing the CPU’s ability to manage and execute multiple instruction sets. Root partition will assign certain logical processors to certain children partitions, those processors called VPs (virtual processors), and the same logical processor on the root partition can be shared between multiple guest os children partitions. Than Hyper-V scheduler implements the [scheduling logic, that can be adjusted](https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/manage/manage-hyper-v-scheduler-types). Nested virtualization is also available for Hyper-V or Azure Virtual Machines.

### VM Exit Handler

When a virtual machine is running, it occasionally performs operations that the hypervisor needs to intercept for management, safety, or emulation purposes. (e.g. non-maskable interrupts) These could be things like sensitive instructions or privileged operations that a guest VM shouldn’t execute directly on the host hardware.

When such an operation occurs, a “VM exit” is triggered, essentially pausing the VM and transferring control to the hypervisor. The VM exit handler is the part of the hypervisor code responsible for dealing with this event. It figures out why the VM exit occurred, performs whatever action is needed (like emulating an instruction or modifying VM state), and then resumes the VM.

**More?**  
If you want to learn more in depth technical details, I recommend official documentation:  
<https://learn.microsoft.com/en-us/windows-server/administration/performance-tuning/role/hyper-v-server/architecture>
