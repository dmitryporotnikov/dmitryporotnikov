---
title: "Simple ARG query to find a NIC by private ip via Azure Resource Graph"
summary: "Simple ARG query to find a NIC by private ip via Azure Resource Graph"
date: '2025-04-23T00:00:00+00:00'
draft: false
---

# Simple ARG query to find a NIC by private ip via Azure Resource Graph

```

resources
    | where type == 'microsoft.network/networkinterfaces'
    | mv-expand ipconf = properties.ipConfigurations
    | extend privateIp = tostring(ipconf.properties.privateIPAddress)
    | extend nicId = tolower(id)
    | where privateIp has "REPLACE WITH IP"
    | project nicId, privateIp

```

