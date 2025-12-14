---
title: "Quick custom script extention test via Azure CLI"
summary: "Quick custom script extention test via Azure CLI"
date: '2025-07-16T00:00:00+01:00'
draft: false
---

## Quick custom script extention test via Azure CLI

If you are unsure, if it is an issue with your script or with custom script extention itself, you can always test with a short Azure CLI one-liner, and primitive script, to exlude CSE issue.
Like so:

```
  az vm extension set \
  --resource-group CSE \
  --vm-name CSETEST \
  --name CustomScriptExtension \
  --publisher Microsoft.Compute \
  --version 1.10 \
  --protected-settings '{ "commandToExecute": "powershell -ExecutionPolicy Unrestricted -Command \"Write-Host hello-world\"" }'

```
