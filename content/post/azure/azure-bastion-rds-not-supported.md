---
title: "Azure Bastion and RDS Session Hosts - Compatibility Issue"
summary: "Azure Bastion and RDS Session Hosts - Compatibility Issue"
date: '2024-06-18T17:50:01+01:00'
draft: false
---

## Azure Bastion and RDS Session Hosts - Compatibility Issue

If you attempt to connect to a VM that is configured as an RDS session host using Azure Bastion, you will encounter an error message indicating invalid credentials. This issue arises because Bastion does not support RDS session hosts.

https://learn.microsoft.com/en-us/azure/bastion/bastion-faq
