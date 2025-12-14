+++
title = 'How to Convert a BEK File for Disk Decryption'
summary = 'How to Convert a BEK File for Disk Decryption'
date = 2024-06-16T00:11:27+02:00
draft = false
+++
## How to Convert a BEK File for Disk Decryption

When working with Azure Key Vault, you might need to download a BitLocker Encryption Key (BEK) to decrypt a disk. However, the downloaded BEK file isn't in a usable format for decryption until it’s properly converted.

Below is a PowerShell script that converts a base64 encoded BEK file to a proper binary format:

```powershell
md "C:\BEK\"                            
$base64Secret = "PASTE_SECRET_HERE"
$binaryFileBytes = [Convert]::FromBase64String($base64Secret)
$filePath = "C:\BEK\decrypted.BEK"
[System.IO.File]::WriteAllBytes($filePath, $binaryFileBytes)
```

The BEK file from Azure Key Vault is base64 encoded, which is not directly usable for BitLocker decryption. By converting it to a binary format, the file becomes compatible and can be used to unlock encrypted disks.
