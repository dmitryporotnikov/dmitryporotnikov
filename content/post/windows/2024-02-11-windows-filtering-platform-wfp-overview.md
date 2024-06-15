---
id: 774
title: 'Had some fun today learning about WFP drivers'
summary: 'Had some fun today learning about WFP drivers'
date: '2024-02-11T21:49:07+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=774'
permalink: /2024/02/11/windows-filtering-platform-wfp-overview/
categories:
    - Code
    - Windows
---

## Had some fun today learning about WFP drivers

WFP is a network packet filtering framework provided by Windows. It provides API support for both user mode and kernel mode, and can be used to easily implement network packet interception/editing, commonly used to implement firewalls, speed control and intrusion detection.

![WFP Architecture](https://cdn.porotnikov.com/media/2024/02/11213108/image-7.png)  
[Image source](https://scorpiosoftware.net/2022/12/25/introduction-to-the-windows-filtering-platform/)  
  
The WFP architecture is shown in the figure above. Although it provides user mode and kernel mode interfaces, the actual work is done in the kernel layer.   
  
The main part of the WFP framework is the Filter Engine in the figure, which interacts and operates with the operating system’s network stack, and can process data in the network stack to implement data filtering and other functions. The filter engine can interact with each layer of the seven-layer network model, realizing the processing of network data at different layers. It includes BFE (Base Filtering Engine, user mode) and KM FE (Kernel Mode Filtering Engine, kernel mode), which interact with the system’s network stack. We can register filter rules or data operations with the filter engine, and the filter engine will eventually complete the corresponding operations according to our filter rules.

**The Callouts** on the right side of the figure is the part that specifically operates on network data. Using the API provided by WFP, we can define our own Callout and register it with the Filter Engine. Each Callout can be understood as a collection of network packet processing, which includes its network layer, filtering function (defining which data packets need to be processed), processing method, etc.

After successful registration, a callout\_id will be returned to identify the registered wfp callout object.

```
	// Register a new Callout with the Filter Engine using the provided callout functions
	s_callout.calloutKey = CALLOUT_GUID;
	s_callout.classifyFn = classify;
	s_callout.notifyFn = notify;
	s_callout.flowDeleteFn = flow_delete;
	status = FwpsCalloutRegister((void*)wdm_device, &s_callout, &callout_id);
	if (!NT_SUCCESS(status)) {
		DbgPrint("Failed to register callout functions for example callout, status 0x%08x", status);
		goto Exit;
	}
```

Callout is a very important data structure in WFP, which includes specific data operation functions and a unique identifier GUID. We mainly implement data operations by registering our own defined Callout with the filter engine. Callouts can perform any kind of processing, such as examining and even modifying packet data.

I explored the Windows Filtering Platform by utilizing the WFPStarterKit, an example driver for Windows created by Jared Wright. This kit demonstrates how to set up basic components of WFP. I modified the callout to block the IP address “168.63.129.16”. You can find the compiled driver and source files on [GitHub](https://github.com/dmitryporotnikov/CustomWFPDriver/tree/main).