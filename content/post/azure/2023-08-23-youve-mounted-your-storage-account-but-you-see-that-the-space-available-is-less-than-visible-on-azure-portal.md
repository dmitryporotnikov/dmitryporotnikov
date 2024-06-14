---
id: 121
title: "You've mounted your storage account, but you see that the space available is less than visible on Azure Portal"
summary: "You've mounted your storage account, but you see that the space available is less than visible on Azure Portal"
date: '2023-08-23T11:58:07+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=121'
permalink: /2023/08/23/youve-mounted-your-storage-account-but-you-see-that-the-space-available-is-less-than-visible-on-azure-portal/
categories:
    - Azure
---

## You've mounted your storage account, but you see that the space available is less than visible on Azure Portal

If you mounted your Azure Storage account via SMB, but you see that the free space is being limited:

![Limited free space on SMB mount](https://cdn.porotnikov.com/media/2023/08/24235754/image-1024x779.png)

Compared to available capacity on Azure portal:

![Available capacity on Azure portal](https://cdn.porotnikov.com/media/2023/08/24235753/image-1-1024x458.png)

Most likely you are limited by the quota configuration:

![Quota configuration](https://cdn.porotnikov.com/media/2023/08/24235751/image-2-1024x490.png)
