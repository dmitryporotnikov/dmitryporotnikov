---
title: "How to move Windows ADE encrypted drive to a different subscription or tenant without removing encryption from the original VM"
summary: "How to move Windows ADE encrypted drive to a different subscription or tenant without removing encryption from the original VM"
date: '2025-07-07T00:00:00+00:00'
draft: false
---

## How to move Windows ADE encrypted drive to a different subscription or tenant without removing encryption from the original VM

1. Create a snapshot of the disk.
2. Create a new disk out of this snapshot.
3. Attach this disk to an existing windows VM as a data drive. 
4. You’ll need to grab a decryption key from the keyvault.
Easiest way to do it, is by using an automation created by one of the MSFT ADE engineers:
_https://github.com/gabriel-petre/ADE/blob/main/GetSecret/GetSecret_1.0.ps1_

`./GetSecret_1.0.ps1 -Mode "local" -subscriptionId "sub ID" -DiskName "Encryppted disk name"`

Should do the trick, and you’ll get a BEK file with decryption key.

5. Unlock the drive with this key, if needed copy it to the temporary VM where disk is attached as data drive:

`manage-bde -unlock G: -RecoveryKey C:\key.BEK`

6. Turn off the bitlocker drive encryption:

`manage-bde -off G:`

7. Monitor the status until the drive is fully decrypted:

`manage-bde -status G:`
 
8. When drive is fully decrypted, detach the disk from a temporary VM. You’ll need to remove the ADE metadata from the disk. First check if metadata is present:


```
$rgName = "resourcegroupname"
$diskName = "diskName"
$disk = Get-AzDisk -ResourceGroupName $rgName -DiskName $diskName

$disk.EncryptionSettingsCollection.EncryptionSettings | convertto-json -depth 10
```


9. Then remove it:


```
$disk.EncryptionSettingsCollection = @{}
$disk | Update-AzDisk
```


10. If the metadata removal is successful, this command should return **null** as the result:

`$disk.EncryptionSettingsCollection.EncryptionSettings | convertto-json -depth 10`

11. Move the disk to a new subscription / tenant using azcopy or azure storage explorer. 
This simple snippet will move the disk export directly, by azcopy:


```
$osDisk = 'DISK EXPORT URL'
$destination = 'https://<>.blob.core.windows.net/vhds?YOURKEY'

$blobName = ($osDisk -split '/')[-1].Split('?')[0]
if (-not $blobName.EndsWith('.vhd')) {
    $blobName += '.vhd'
}
$destinationBase = $destination.Substring(0, $destination.IndexOf('?'))
$sasPart = $destination.Substring($destination.IndexOf('?'))
$newDestination = $destinationBase.TrimEnd('/') + '/' + $blobName + $sasPart
.\azcopy.exe cp $osDisk $newDestination
```


12. Create a new managed disk out of the exported VHD.
13. Create a new VM from this managed disk and verify if it is booted via boot diagnostic.
