---
title: "Qnap TS-412 and Time Machine on Modern Mac OS"
summary: "How to configure Time Machine backup on a legacy NAS"
date: '2025-11-26T00:00:00+00:00'
draft: false
---

### Intro

I have a lecacy nas (QNAP TS 412) and I wanted to use it for Time Machine backups for my Macbook. Challenge is, it is a legacy hardware, and despite still getting updates from QNAP, the packages like HBS for backup, are quite old. 

Apple decided to deprecate the [AFP protocol](https://en.wikipedia.org/wiki/Apple_Filing_Protocol), and modern Time Machine backups now use SMB instead. 

### The issue

If you will try to configure the backup via GUI, you will be greeted with **I"The selected network backup disk does not support the required capabilities."** message. This is most likely due to the fact, that the GUI tries to configure backup via SMB, but the QNAP is offering AFP by default.

### How to solve this?

```bash
sudo tmutil setdestination -p afp://TimeMachine@192.168.1.101/TMBackup
```

Set the IP to IP of your NAS, the backup target and username accordingly. The OS will complain about AFP, but at least for the time being backup and restore works. 

![Time Machine backup using AFP on QNAP TS-412](https://cdn.porotnikov.com/media/2025/11/26/afp_backup.png)

