---
id: 70
title: 'C#: Check if service exists via WMI'
summary: 'C#: Check if service exists via WMI'
date: '2023-08-22T13:55:49+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=70'
permalink: /2023/08/22/c-check-if-service-exists-via-wmi/
categories:
    - Code
---

## C#: Check if service exists via WMI

```csharp
using System;
using System.Management;

namespace debug
{
    internal class Program
    {
        static void Main(string[] args)
        {
            string query = "select * from Win32_Service where name='W3svc'";
            try
            {
                using (ManagementObjectSearcher searcher = new ManagementObjectSearcher(query))
                 {
                    ManagementObjectCollection services = searcher.Get();
                    if (services.Count == 0)
                    {
                                Console.WriteLine("IIS Service not found.");
                    }
                    else
                    {
                        foreach (ManagementObject service in searcher.Get())
                            {
                                string name = service["Name"].ToString();
                                string displayName = service["DisplayName"].ToString();
                                string status = service["State"].ToString();

                                Console.WriteLine("Service Name: " + name);
                                Console.WriteLine("Service Display Name: " + displayName);

                                //Additional checks if status is not running?
                                Console.WriteLine("Service Status: " + status);
                                if (status == "Running")
                                {
                                // Do stuff
                                }
                        }
                    }
                }
            }
            catch
            {
                Console.WriteLine("Unable to connect to the WMI service");
            }
        }
    }
}
```
