---
id: 54
title: 'Azure CLI: Create IPV6 enabled VM'
date: '2023-08-22T13:11:20+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=54'
permalink: /2023/08/22/azure-cli-create-ipv6-enabled-vm/
summary: "How to create IPv6 only enabled VM in Azure"
categories:
    - Code
---

## Azure CLI: Create IPV6 enabled VM

```shell
az group create --name IPv6 --location northeurope
 
az network vnet create --resource-group IPv6 --location northeurope --name IPv6-Net --address-prefixes 10.0.0.0/16 2404:f800:8000:122::/63 --subnet-name myBackendSubnet --subnet-prefixes 10.0.0.0/24 2404:f800:8000:122::/64

az network public-ip create --resource-group IPv6 --name PublicIP-Ipv4 --sku Standard --version IPv4 --zone 1 2 3

az network public-ip create --resource-group IPv6 --name PublicIP-Ipv6 --sku Standard --version IPv6 --zone 1 2 3

az network nsg create --resource-group IPv6  --name IPv6NSG

az network nic create --resource-group IPv6 --name IPv6NIC1 --vnet-name IPv6-Net --subnet myBackEndSubnet --network-security-group IPv6NSG --public-ip-address PublicIP-IPv4

az network nic ip-config create --resource-group IPv6 --name IPv6config --nic-name IPv6NIC1 --private-ip-address-version IPv6 --vnet-name IPv6-Net --subnet myBackendSubnet --public-ip-address PublicIP-Ipv6

az vm create --resource-group IPv6 --name IPv6VM --nics IPv6NIC1 --image Win2022Datacenter --admin-username azureuser
```