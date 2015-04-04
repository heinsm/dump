---
layout: post
title: Mounting Physical Disk storage in VirtualBox
categories: [sysadmin]
tags: [virtualbox]
# date: timestamp
disclaimer: true
---

Ref:

* https://www.virtualbox.org/manual/ch09.html
* https://forums.virtualbox.org/viewtopic.php?f=2&t=38124
* https://forums.virtualbox.org/viewtopic.php?f=6&t=38914

I have a dual booting linux / windows lappy.  Sometimes I want a pure linux env better speed, etc.); sometimes I want Windows environment.  and sometimes I want both!.
Wouldn't it be awesome to VM my dual booted linux into a windows hosted VM?  This article shows how to do exactly that.

## Prerequisites

* Existing dual booting system with grub bootloader in the MBR (not first partition).  You may need fiddle the instructions if this isn't the case.
* 1 Drive contains both systems.   Again you may need to fiddle the instructions if this isn't the case

## Specifically - how to wrap a specific physical partition

In windows, use "PhysicalDrive#" to list out the partitions to find the right one (matching size, placement, etc) with Computer Managagment console/Disk Management to get your bearings.

Open command prompt as administrator to the vitrual box install directory...

**Note:** this needs to be ran inside a command prompt opened with the "run as administrator".

```batch
cd C:\Program Files\Oracle\VirtualBox
```

Assuming you have one disk, try:  (incrementing PhysicalDrive# as needed)

```batch
VBoxManage internalcommands listpartitions -rawdisk \\.\PhysicalDrive0

Number  Type   StartCHS       EndCHS      Size (MiB)  Start (Sect)
1       0x27  0   /32 /33  191 /89 /26          1500         2048
2       0x07  191 /89 /27  1023/254/63        459638      3074048
5       0x83  1023/254/63  1023/254/63        452958    944414720
6       0x82  1023/254/63  1023/254/63          3953   1872074752
3       0x17  1023/254/63  1023/254/63         35816   1880170496
```

Notice the Type column - we're interested in 0x83 linux partitions.  0x82 are linux swap partitions.
So then create the raw disk to give guest access only to the partitions of our choosing.  This helps eliminatre the possibility of accidently killing our windows partitions etc...:

```batch
VBoxManage internalcommands createrawvmdk -filename C:\work\vmware\Ubuntu1410\disk1.vmdk -rawdisk \\.\PhysicalDrive0 -partitions 5,6
```

We then open VirtualBox; create a "New" VM.
For me it was ubuntu 64bit -> Next
Select a Disk... browse to your newly created raw disk.  Notice there are two (if you are isolating partition access with --partitions):

1. disk1.vmdk = raw disk
2. disk1-pt.vmdk = some config for the raw disk (not directly mountable in the VM)

Selecting #1 resulted in VERR\_ACCESS\_DENIED for me.

Read some forums about DISKPART utility and Clearing the readonly flags after offlining a disk.  However this doesn't work if

* Its a partition or two i'm raw accessing.
* Offlining the disk thats running windows doesn't sound like a good idea.

Solution was to launch VirtualBox Manager with "Run as Administrator".  Created the VM and mounted the storage no problems.  The VM then booted no problem.

Once booted, used the VM to "Insert Guest Additions" CD; installed the VGA driver, and VM awareness extensions.  Worked like a charm!
