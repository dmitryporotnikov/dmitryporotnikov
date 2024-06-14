---
id: 179
title: 'Bicep: Execute a RunCommand on already running Azure Virtual Machine'
summary: 'Bicep: Execute a RunCommand on already running Azure Virtual Machine'
date: '2023-08-27T17:09:56+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=179'
permalink: /2023/08/27/bicep-execute-a-runcommand-on-already-running-azure-virtual-machine/
categories:
    - Bicep
---

## Bicep: Execute a RunCommand on already running Azure Virtual Machine

```bicep
param vmName string
param location string
//https://learn.microsoft.com/en-us/azure/templates/microsoft.compute/2022-11-01/virtualmachines/runcommands?pivots=deployment-language-bicep
resource runCommand 'Microsoft.Compute/virtualMachines/runCommands@2022-11-01' = {
  name: '${vmName}/runPowerShellCommand'
  location: location
  properties: {
    source: {
      script: 'Write-Host "Hello World"'
    }
  }
}
```
