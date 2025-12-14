---
id: 731
title: 'PowerShell: Create a New Domain Admin User'
summary: 'PowerShell: Create a New Domain Admin User'
date: '2023-12-01T09:05:10+00:00'
author: admin
layout: post
guid: 'https://porotnikov.com/?p=731'
permalink: /2023/12/01/recover-azure-vm-domain-admin-access/
categories:
    - Code
    - Windows
---

## PowerShell: Create a New Domain Admin User

If you have deployed a Windows Virtual Machine from the Azure Marketplace that comes with a domain controller pre-installed, and your local domain admin password has expired, you will not be able to recover access using Azure’s built-in password recovery tool, as it is not supported on domain controllers. However, you can use the script provided below as a Run Command to create a new domain admin user and regain access:

```bicep
Import-Module ActiveDirectory

# Define new user parameters
$newUsername = "RecoveryUser"
$password = "YourPassword" | ConvertTo-SecureString -AsPlainText -Force
$userProperties = @{
    SamAccountName = $newUsername
    UserPrincipalName = "$newUsername@yourdomain.com"
    Name = $newUsername
    GivenName = "FirstName"
    Surname = "LastName"
    Enabled = $true
    DisplayName = "FirstName LastName"
    AccountPassword = $password
    ChangePasswordAtLogon = $false
}

# Create the new user
New-ADUser @userProperties

# Add the user to the Domain Admins group
Add-ADGroupMember -Identity "Domain Admins" -Members $newUsername

Write-Host "User $newUsername created and added to Domain Admins."
```
