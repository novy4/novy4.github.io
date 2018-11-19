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







