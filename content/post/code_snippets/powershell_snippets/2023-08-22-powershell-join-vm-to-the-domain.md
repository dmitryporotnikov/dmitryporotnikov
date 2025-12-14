---
id: 56
title: 'Powershell: Join VM to the Domain'
summary: 'Powershell: Join VM to the Domain'
date: '2023-08-22T13:12:58+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=56'
permalink: /2023/08/22/powershell-join-vm-to-the-domain/
categories:
    - Code
---
## Powershell: Join VM to the Domain

```powershell
## Description: This script will join a computer to a domain and force a reboot
$domain = "lab.local"
$username = "$domain\username" 
$password = Read-Host 'What is your password?' -AsSecureString
$credential = New-Object System.Management.Automation.PSCredential($username,$password)
Add-Computer -DomainName $domain -Credential $credential
Write-Host ("Computer joined domain " + $domain)
gpupdate /force
shutdown -r -f -t 60
```
