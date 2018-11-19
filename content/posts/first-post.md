+++ 
draft = false
date = 2018-11-12T16:02:23+02:00
title = "Azure Disk Encryption for Linux VMs"
slug = "ade-for-linux" 
tags = [
    "azure",
    "ade",
    "linux",
    "encryption"
]
categories = [
    "cloud"
]
+++

Azure provides disk encryption option for Iaas virtual machines, this is a perfect solution for those who want to keep data encrypted with own keys. Some aspects I would like to clarify here to have a more transparent solution. I have tested and implemented this solution. Ok, lets have some fun...

## There are some prerequests before we can start to encrypt our disks.
* Azure Disk Encryption is only supported on specific Azure Gallery based Linux server distributions and versions. For the list of currently supported versions, refer to the Azure Disk Encryption FAQ.
* Azure Disk Encryption requires that your key vault and VMs reside in the same Azure region and subscription.
* Before enabling encryption, the data disks to be encrypted need to be properly listed in /etc/fstab. Use a persistent block device name for this entry, as device names in the "/dev/sdX" format can't be relied upon to be associated with the same disk across reboots, particularly after encryption is applied.

## Supported scenarios
* Integration with Azure Key Vault
* Enable encryption on Windows and Linux IaaS VMs, managed disk, and scale set VMs from the supported Azure Gallery images
* Backup and restore of encrypted VMs, for both no-KEK and KEK scenarios (KEK - Key Encryption Key)
* Enable encryption on Linux VM OS and data disks
* Disable encryption on Linux VM data disks
* Enable encryption on volumes with mount paths

## The solution doesn't support the following scenarios
* Basic tier IaaS VMs
* Disabling encryption on an OS drive for Linux IaaS VMs
* Disabling encryption on a data drive if the OS drive is encrypted for Linux Iaas VMs
* IaaS VMs that are created by using the classic VM creation method
* Enabling encryption on Linux IaaS VMs customer custom images
* Network File System (NFS)

## ADE Workflow diagram

The below diagram details the ADE workflow for IaaS VMs
![ADE Workflow Diagram](/images/ade-diagram.jpg)

## ADE WORKFLOW FOR WINDOWS/LINUX VM
We are using the az vm encryption enable command to enable encryption on a running IaaS virtual machine in Azure.
It is mandatory to snapshot and/or backup a managed disk based VM instance outside of, and prior to enabling Azure Disk Encryption. A snapshot of the managed disk can be taken from the portal, or Azure Backup can be used. Backups ensure that a recovery option is possible in the case of any unexpected failure during encryption.
Encrypting or disabling encryption may cause the VM to reboot.

#### Make sure we have access to keyvault then let’s enable encryption option for it and then let’s create a KEK.
```
az keyvault list
az keyvault update --name "MySecureVault" --resource-group "MySecureRg" --enabled-for-disk-encryption "true"
az keyvault key create --name “ADEkey” --vault-name “MySecureVault”
```

### Encrypt a running VM using BEK+KEK with Azure CLI

This method works only if key-vault and the VM are in the same resource group.
```
az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type [All|OS|Data]
```

The syntax for the value of disk-encryption-keyvault parameter is the full identifier string: /subscriptions/[subscription-id-guid]/resourceGroups/[resource-group-name]/providers/Microsoft.KeyVault/vaults/[keyvault-name] 
The syntax for the value of the key-encryption-key parameter is the full URI to the KEK as in: https://[keyvault-name].vault.azure.net/keys/[kekname]/[kek-unique-id]


### Encrypt a running VM using BEK+KEK with PowerShell

This method is recommended in case when key-vault and the VM are in the different resource groups.
Lets login to our subscription:
```
Login-AzureRmAccount
```

#### Lets set access policies for the keyvault:
```
Set-AzureRmKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'KeyVaultRg' -EnabledForDiskEncryption
Set-AzureRmKeyVaultAccessPolicy -VaultName 'MySecureVault' -ResourceGroupName 'KeyVaultRg' -EnabledForDeployment 
```

#### Script that is doing all the magic:
```
$rgName = 'MySecureRg';
$vmName = 'MySecureVM';
$KeyVaultName = 'MySecureVault';
$KeyVault = Get-AzureRmKeyVault -VaultName $KeyVaultName -ResourceGroupName "KeyVaultRg" ;
$diskEncryptionKeyVaultUrl = $KeyVault.VaultUri;
$KeyVaultResourceId = $KeyVault.ResourceId; 
Set-AzureRmVMDiskEncryptionExtension -ResourceGroupName $rgname -VMName $vmName -DiskEncryptionKeyVaultUrl $diskEncryptionKeyVaultUrl -DiskEncryptionKeyVaultId $KeyVaultResourceId -VolumeType "All"
```
#### Verify that disks are encrypted with Azure CLI

```
az vm encryption show --name "MySecureVM" --resource-group "MySecureRg"
```

#### Enable encryption on a newly added disk with Azure CLI
The Azure CLI command will automatically provide a new sequence version for you when you run the command to enable encryption. The example uses "All" for the volume-type parameter. You may need to change the volume-type parameter to OS if you're only encrypting the OS disk. In contrast to Powershell syntax, the CLI does not require the user to provide a unique sequence version when enabling encryption. The CLI automatically generates and uses its own unique sequence version value.
```
az vm encryption enable --resource-group "MySecureRg" --name "MySecureVM" --disk-encryption-keyvault  "MySecureVault" --key-encryption-key "MyKEK_URI" --key-encryption-keyvault "MySecureVaultContainingTheKEK" --volume-type All
```

#### Disable encryption with Azure CLI

To disable encryption, use the az vm encryption disable command. Disabling data disk encryption on Windows VM when both OS and data disks have been encrypted doesn’t work as expected. Disable encryption on all disks instead.
```
az vm encryption disable --name "MySecureVM" --resource-group "MySecureRg" --volume-type [ALL, DATA, OS]
```








