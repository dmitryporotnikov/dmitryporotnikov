---
id: 72
title: 'C#: Working with Azure IMDS'
summary: 'C#: Working with Azure IMDS'
date: '2023-08-22T13:58:28+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=72'
permalink: /2023/08/22/c-working-with-azure-imds/
categories:
    - Code
---

## C#: Working with Azure IMDS

```csharp
class Program
{
    static void Main(string[] args)
    {
        //Identify Storage Type Using Azure Metadata Service
        string url = "http://169.254.169.254/metadata/instance?api-version=2021-02-01";
        WebClient client = new WebClient();
        client.Headers.Add("Metadata", "true");

        try
        {
        string response = client.DownloadString(url);
        dynamic data = JsonConvert.DeserializeObject(response);

        string storageProfile = data.compute.storageProfile.ToString();
        dynamic storageData = JsonConvert.DeserializeObject(storageProfile);

        string osDiskName = storageData.osDisk.name;
        string osDiskCaching = storageData.osDisk.caching;
        string osDiskSize = storageData.osDisk.diskSizeGB;
        string writeAcceleratorEnabled = storageData.osDisk.writeAcceleratorEnabled;

        string datadisks = storageData.dataDisks.ToString();
        

        Console.WriteLine("OS Disk Name: " + osDiskName);
        Console.WriteLine("OS Disk Caching: " + osDiskCaching);
        Console.WriteLine("OS Disk Size: " + osDiskSize);
        Console.WriteLine("writeAcceleratorEnabled: " + writeAcceleratorEnabled);

        Console.WriteLine("Data disks: " + datadisks); //Parse Datadisks and check caching / write acceleration / SKU type same as OS disk above, by deserializing JSON response.


        }
        catch (WebException ex)
        {
            //If we are running on non Azure VM or there are networking restrictions the connection will timeout:
            Console.WriteLine("Error: " + ex.Message);
        }
    }
}
```
