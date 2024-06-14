---
id: 26
title: 'Capture network traffic using windows built in tools'
summary: 'Capture network traffic using windows built in tools'
date: '2023-08-22T10:36:34+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=26'
permalink: /2023/08/22/capture-network-traffic-using-windows-built-in-tools/
categories:
    - Troubleshooting
---

## Capture network traffic using windows built in tools

Start the capture:  
**netsh trace start capture=yes level=5 tracefile=c:\\temp\\%computername%\_nettrace.etl scenario=netconnection**

&lt;reproduce the issue&gt;

Stop the capture:  
**netsh trace stop**  
  
Convert the resulting file to wireshark format:  
[GitHub – microsoft/etl2pcapng: Utility that converts an .etl file containing a Windows network packet capture into .pcapng format.](https://github.com/microsoft/etl2pcapng)
