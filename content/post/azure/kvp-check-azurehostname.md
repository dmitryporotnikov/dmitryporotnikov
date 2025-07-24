---
title: "Check if two Azure VMs are on the same physical host"
summary: "Check if two Azure VMs are on the same physical host"
date: '2025-07-24T00:00:00'
draft: false
---

## Check if two Azure VMs are on the same physical host

Have you ever wondered if your Azure workloads are located on the same physical host? Imagine you're facing a strange issue, and after checking everything on your end, you notice that only a subset of virtual machines is affected. Could the Azure host be the culprit? Maybe, maybe not. Should you re-deploy?

Fortunately, you can check whether two VMs are located on the same host using the good old Hyper-V Data Exchange protocol.
👉 [Microsoft Documentation](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/dn798287%28v=ws.11%29)

**For Windows:**

```powershell
$regPath = "HKLM:\SOFTWARE\Microsoft\Virtual Machine\Guest\Parameters"
$hostName = Get-ItemProperty -Path $regPath -Name "PhysicalHostName"
$hostName.PhysicalHostName
```

![](https://cdn.porotnikov.com/media/2025/7/24/1.png)

**For Linux:**

```bash
strings /var/lib/hyperv/.kvp_pool_1 | grep "server-name"
```

![](https://cdn.porotnikov.com/media/2025/7/24/2.png)

