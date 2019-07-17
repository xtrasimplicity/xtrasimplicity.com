---
title: "Building a Self-Hosted Cloud With NextCloud"
date: 2018-11-17T09:53:14+11:00
draft: true
---

[desired_setup]: /images/93df9a951d683a8d.png

I currently have a NAS running FreeNAS on an old HP MicroServer which exports a number of SMB shares, but after having spent some time playing around with NextCloud in a Docker container, I've been eager to add it to my current setup to make it even easier to remotely access and manage my files.

As with many of my personal projects, I don't really have a budget to purchase additional hardware to do this, so I've decided to take advantage of virtualisation to keep the cost down and to get by with the resources I have at my disposal.

### Physical Hardware
* HP MicroServer N36L w/ 8GB RAM
* 2x 3TB WD RED NAS 3.5" Drives, in a mirrored ZFS Pool
* 2x 2TB Seagate 3.5" Drives, stand-alone
* 1x 250GB WD 3.5" SATA Drive
* All Drives are connected using the built-in SATA controller.

### My current FreeNAS setup
Currently, the bare-metal FreeNAS installation is installed to the 250GB drive, and the other drives are used for storage.

FreeNAS exports the 3TB ZFS volume and the 2x stand-alone 2TB drives as 3 separate SMB shares. _Ideally_, I would have set up a ZFS mirror with these two 2TB drives, but I don't have enough spare drives at the moment.

### Where I'd like to end up
![Image showing the desired virtualisation configuration][desired_setup]

As I won't be presenting these volumes to the server via iSCSI or Fibre-channel, and don't have access to an additional SATA controller (for pass-through), I'll have to settle on mapping the physical drives to VMDK files using VMWares `Raw Device Mapper` (RDM) feature and mounting these to the FreeNAS VM.

As a result, I'll unfortunately lose the ability to read each drive's S.M.A.R.T status from within the FreeNAS VM, which is far from ideal, but unavoidable at this point in time.



--> Multiple Drives, RDM to FreeNAS VMDK
--> 