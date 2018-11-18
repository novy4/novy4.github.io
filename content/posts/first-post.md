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

{{ if isset .Params "title" }}<h4>{{ index .Params "title" }}</h4>{{ end }}




