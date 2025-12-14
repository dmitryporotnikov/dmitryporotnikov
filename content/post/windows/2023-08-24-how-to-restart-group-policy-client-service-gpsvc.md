---
id: 171
title: 'How to restart Group Policy Client Service (GPSVC)'
summary: 'How to restart Group Policy Client Service (GPSVC)'
date: '2023-08-24T14:54:07+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=171'
permalink: /2023/08/24/how-to-restart-group-policy-client-service-gpsvc/
categories:
    - Troubleshooting
    - Windows
---

## How to restart Group Policy Client Service (GPSVC)

Are you facing difficulties in restarting the Group Policy Client service? It’s a common issue that many have run into, especially when they find the option grayed out in the services list or receive an “Access is Denied” error from the command line. Let’s dive into why this happens and how you can get around it.

The Group Policy Client service requires specific permissions to manage, specifically System account permissions. So, if you’re running commands as a normal user account, you’ll likely run into Access Denied message.

Fortunately, there’s a tool that can help us get around this obstacle. Here’s what you need to do:

**Download the Sysinternals Suite**:
<https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite>  
This suite contains a utility called `psexec.exe`, which will be instrumental in solving our problem.

**Extract the Suite**: Once downloaded, make sure to unzip it and find the `psexec.exe` file.

**Run the Command to Open an Interactive Window**: By executing the following command, you can open an interactive window as the system account:

```shell
psexec.exe -s -i cmd.exe
```

**Restart the Group Policy Client Service**: With the system account’s command window, you can now stop and start the Group Policy Client service using:

```shell
net stop gpsvc
net start gpsvc
```

The process outlined above helps you gain control over the service using the system account.  
