---
id: 810
title: 'Setting Up Chrony for Time Synchronization in Azure VMs'
summary: 'Setting Up Chrony for Time Synchronization in Azure VMs'
date: '2024-06-10T08:46:23+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=810'
permalink: /2024/06/10/azure-linux-vm-time-synchronization/
categories:
    - Azure
    - Linux
---

## Setting Up Chrony for Time Synchronization in Azure VMs

Azure VMs have several options for maintaining accurate time:

1. VMs can inherit time from their host (host time).  
2. VMs can synchronize directly with an external time server.  
3. A combination of both methods can be used.  
  
When VMs restart, experience network traffic, or undergo maintenance, their clocks might drift. Azure helps manage this through the VMICTimeSync service and the integration of the Precision Time Protocol. Azure hosts are synced to Microsoft’s internal time servers, connected to GPS-antenna-equipped Stratum 1 devices. During maintenance or reboots, the VMICTimeSync updates the VM clocks to correct any discrepancies.


### Setting Up Chrony on Azure Linux VMs

For newer Linux distributions in Azure, `chronyd` is preferred over `ntpd` due to its ability to sync more directly and reliably with the Azure host time. Here’s how to set it up:


#### 1. **Install Chrony**

Typically, `chronyd` might already be installed on newer distributions. If not, you can install it using your package manager:

```bash
sudo apt update  
sudo apt install chrony
```

#### 2. **Configure Chrony**

Edit the Chrony configuration file to use the Azure host as a time source:

```bash
sudo nano /etc/chrony/chrony.conf
```

Add this line to configure the PTP hardware clock source provided by Azure:

```bash
refclock PHC /dev/ptp_hyperv poll 3 dpoll -2 offset 0 stratum 2
```

This configuration treats the Azure host as a highly accurate Stratum 2 time source.

#### 3. **Restart Chrony**

Apply the new settings by restarting the Chrony service:

```bash
sudo systemctl restart chronyd
```

### Verifying the Configuration

Check the time synchronization status with:

```bash
chronyc tracking
```

This command will show if Chrony is properly syncing with the configured time source.

### Additional Configuration and Troubleshooting

- For systems using `systemd` or cloud-init, additional configuration steps might be required to ensure that `chronyd` is the primary time synchronization service.
- In case of time drift issues, adjusting the `makestep` directive in the Chrony configuration can force a time correction when the drift exceeds a certain threshold.

For more detailed information and troubleshooting, refer to the [Microsoft documentation on time synchronization for Azure Linux VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/linux/time-sync).
