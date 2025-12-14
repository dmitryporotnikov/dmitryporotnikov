---
title: "Disable automatic reboots, for VM with AutomaticByPlatform patching mode"
summary: "Disable automatic reboots, for VM with AutomaticByPlatform patching mode"
date: '2024-07-10T00:58:07+00:00'
draft: false
---

# Disable automatic reboots

```
az vm update --resource-group yourRGname --name yourVMname --set osProfile.windowsConfiguration.patchSettings.automaticByPlatformSettings.rebootSetting=Never
```

# Check

```
az vm show --resource-group yourRGname --name yourVMname --query "osProfile.windowsConfiguration.patchSettings.automaticByPlatformSettings.rebootSetting"
```
