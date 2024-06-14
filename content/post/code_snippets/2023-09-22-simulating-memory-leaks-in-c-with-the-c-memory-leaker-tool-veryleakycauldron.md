---
id: 410
title: 'C++: Memory Leaker Tool &#8211; VeryLeakyCauldron'
summary: 'C++: Memory Leaker Tool &#8211; VeryLeakyCauldron'
date: '2023-09-22T14:09:42+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=410'
permalink: /2023/09/22/simulating-memory-leaks-in-c-with-the-c-memory-leaker-tool-veryleakycauldron/
categories:
    - Code
---

## VeryLeakyCauldron

Memory leaks are a common problem in programming. They can occur when memory is allocated and then not properly deallocated, leading to a waste of resources and potentially causing performance problems.

This C++ Memory Leaker tool is a simple tool that can be used to simulate memory leaks. It works by allocating a specified amount of memory and then not deallocating it. This can be done manually or automatically at a specified interval.

```C
#include <iostream>
#include <thread>
#include <chrono>
#include <cstring>
#include <string>

void allocateMemory(size_t sizeInMB) {
    size_t size = sizeInMB * 1024 * 1024 / sizeof(int);
    int* leakyMemory = new int[size];
}

int main(int argc, char* argv[]) {
    std::string mode = "";
    size_t leakSize = 0;

    // Parse command line arguments
    for (int i = 1; i < argc; i++) {
        if (strcmp(argv[i], "-mode") == 0 && i + 1 < argc) {
            mode = argv[i + 1];
            i++;
        }
        else if (strcmp(argv[i], "-leak") == 0 && i + 1 < argc) {
            leakSize = std::stoi(argv[i + 1]);
            i++;
        }
    }

    if (mode == "manual") {
        while (true) {
            allocateMemory(leakSize);
            std::cout << "Memory allocated. Press Enter to allocate more." << std::endl;
            std::cin.get();
            if (std::cin.get() == '\n') continue;
        }
    }
    else if (mode == "auto") {
        while (true) {
            allocateMemory(leakSize);
            std::cout << "Memory allocated automatically." << std::endl;
            std::this_thread::sleep_for(std::chrono::seconds(1));
        }
    }
    else {
        std::cout << "Pass parameters: -manual or -auto. For -auto use desired leak size in MB." << std::endl;
    }

    return 0;
}
```

To use the C++ Memory Leaker tool, compile the source code provided above and then run the program with the desired command line arguments. Compiled binary for Windows is available on GitHub. Link is at the bottom of this page. The following table shows the available command line arguments:

| Argument | Description |
|---|---|
| `-mode` | Specifies the mode of operation. The two possible modes are `manual` and `auto`. In `manual` mode, the user is prompted to press Enter to allocate more memory. In `auto` mode, the tool allocates memory automatically at a specified interval. |
| `-leak` | Specifies the amount of memory to leak in megabytes. This argument is only required in `auto` mode. |

For example, to simulate a memory leak of 100 megabytes every second, you would run the program with the following command:

```shell
VeryLeakyCauldron.exe -mode auto -leak 100
```

To simulate a memory leak of 100 megabytes that must be manually allocated, you would run the program with the following command:

```shell
VeryLeakyCauldron.exe -mode manual -leak 100
```

## GitHub

Here is the link:
[https://github.com/dmitryporotnikov/VeryLeakyCauldron/tree/main](https://github.com/dmitryporotnikov/VeryLeakyCauldron/tree/main)
