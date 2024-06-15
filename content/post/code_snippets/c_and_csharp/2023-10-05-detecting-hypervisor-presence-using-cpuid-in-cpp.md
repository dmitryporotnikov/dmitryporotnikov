---
id: 589
title: 'C++: Detecting Hypervisor Presence Using CPUID Timing'
summary: 'C++: Detecting Hypervisor Presence Using CPUID Timing'
date: '2023-10-05T21:12:44+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=589'
permalink: /2023/10/05/detecting-hypervisor-presence-using-cpuid-in-cpp/

categories:
    - Code
    - Virtualization
---

## C++: Detecting Hypervisor Presence Using CPUID Timing

A VM-exit is a mechanism that allows a VM to transfer control to its hypervisor. This can be triggered by various events such as hardware interrupts, software exceptions, and unsupported instructions. Once a VM-exit is triggered, the hypervisor temporarily halts the VM’s execution to carry out specific tasks or services that are required for the smooth operation of the virtualized environment. And in the case of the CPUID instruction, a VM-exit is triggered when the guest operating system attempts to execute the instruction.

The CPUID instruction is a processor supplementary instruction for the x86 architecture. It is used to query the CPU for information about its features and capabilities.

### How VM-Exits Work with CPUID

1. The guest operating system executes the CPUID instruction.
2. The CPU generates a VM-exit interrupt.
3. The hypervisor intercepts the VM-exit interrupt and takes control of the VM.
4. The hypervisor executes the CPUID instruction and returns the results to the guest operating system.
5. The hypervisor transfers control back to the guest operating system.

![Image 11](https://cdn.porotnikov.com/media/2023/10/05205407/image-11-1024x568.png)

The CPU calls for VM-exit, so the hypervisor can:

- Provide accurate information about the CPU’s features and capabilities.
- Emulate CPU features not supported by the underlying hardware.
- Prevent the guest OS from accessing certain CPU features.

To demonstrate how VM-exits can be used for VM detection, let’s look at a C++ code snippet that measures the time it takes for the CPUID instruction to execute:

```cpp
#include <iostream>
#include <string>
#include <intrin.h>
#include <chrono>

bool IsRunningInVM() {
    int cpuInfo[4] = { -1 };
    char vendor[13];

    auto start = std::chrono::high_resolution_clock::now();

    __cpuid(cpuInfo, 0);

    auto stop = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::nanoseconds>(stop - start);

    *((int*)vendor) = cpuInfo[1];
    *((int*)(vendor + 4)) = cpuInfo[3];
    *((int*)(vendor + 8)) = cpuInfo[2];
    vendor[12] = '\0';

    std::cout << "Time taken by cpuid: " << duration.count() << " nanoseconds" << std::endl;

    // Check if the time taken is above a certain threshold (this is just an example threshold)
    if (duration.count() > 1000) {
        return true;
    }

    return false;
}

int main() {
    if (IsRunningInVM()) {
        std::cout << "The code is likely running within a VM based on timing." << std::endl;
    }
    else {
        std::cout << "The code is likely running on bare-metal hardware based on timing." << std::endl;
    }

    return 0;
}
```

On my laptop, this call takes around 500-600 nanoseconds:

![Image 12](https://cdn.porotnikov.com/media/2023/10/05210614/image-12-1024x793.png)

Microsoft Azure VM:

![Image 13](https://cdn.porotnikov.com/media/2023/10/05210724/image-13-1024x541.jpg)

P.S. I stumbled upon this method while I was reverse engineering a certain malware. I think what the creators did was pretty smart, so I wanted to see if certain CPU instructions are indeed a tad slower in a virtualized environment, and I see that they are 🙂