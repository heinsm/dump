---
layout: post
title: Dell PowerEdge 1750, Wake-on-lan, and debian
categories: [sysadmin]
tags: [wakeonlan]
# date: timestamp

---

# WHATABOUT

Details how to setup Wake-On-Lan (WOL) with a Dell PowerEdge 1750, Broadcom NetXtreme BCM5704 Gigabit ethernet, and debian (lenny); specifically, dealing with the issue of having the NIC card configured for WOL, but after issuing the standard 'shutdown -h now' command from within linux; the NIC card does not respond to WOL magic packets.

## Broadcom NetXtreme NICs

With the BCM5704 nics, the details of the configuration tools for linux, or dos can be found through the dell website (www.dell.com); in our case, the configuration of the card could be done through BIOS.  To enable the prompt, we temporarily enable the PXE boot option for the primary NIC interface from within the BIOS settings screen;  (one may also take note of the MACs for these NIC interfaces at this point). On the next reboot, the NIC card verbosely declares its configuration can be changed by pressing 'CTRL-S'.  For us, this was just prior to it searching for the PXE boot server.  We note that on Dell Poweredge 1750, there is a user warning to only enable 1 NIC for WOL at a time due to the inability to support many NICs on the standby 3.3v rail.

Pressing 'CTRL-S' brings us to a configuration panel which allows for enabling the Wake-On-LAN (WOL) mode of the card.

# Problem

The WOL performance was as expected in every situation except if we shutdown or suspended from within Linux.  The WOL magic packet woke up the machine from the S5 halt power state in these scenarios:

* If workstation power was disabled half way through BIOS posting (but prior to linux booting)
* When Windows was used to shutdown the machine

# One Solution

It took awhile to determine the blame on power state traversal when linux was running its shutdown / halt script; but in the end the solution was to modify the behavior of linux when entering into a specific power state.  To do this, we used a few command line tools to investigate NIC WOL state (or what linux thought the WOL state was), NIC driver, and PCI bus enumerations.

To get started we installed:

* ethtool
    - apt-get install ethtool
* acpitool
    - apt-get install acpitool
* lspci
    - apt-get install pciutils

## Enable Interface for WOL

First, we needed to ensure the NIC driver supported WOL settings that were configured through the NIC BIOS configuration.  We used ethtool to do this; assuming eth0 is the NIC interface you wish to use for WOL, you would:

```bash
ethtool eth0
```

The response should be somthing similar to the below; note the line item that says 'Wake-on:' - there are a few letters that may appear here; most importantly, 'g' indicates WOL for receiving a magic packet. Check the ethtool for description of other letters for different WOL modes.

```text
Settings for eth0:
        Supported ports: [ TP ]
        Supported link modes:   10baseT/Half 10baseT/Full 
                                100baseT/Half 100baseT/Full 
                                1000baseT/Half 1000baseT/Full 
        Supports auto-negotiation: Yes
        Advertised link modes:  10baseT/Half 10baseT/Full 
                                100baseT/Half 100baseT/Full 
                                1000baseT/Half 1000baseT/Full 
        Advertised auto-negotiation: Yes
        Speed: 100Mb/s
        Duplex: Full
        Port: Twisted Pair
        PHYAD: 1
        Transceiver: internal
        Auto-negotiation: on
        Supports Wake-on: g
        Wake-on: g
        Current message level: 0x000000ff (255)
        Link detected: yes
```

If the 'g' is not present, check the line above - 'Supports Wake-on';  if 'g' is not in the supported modes, then either your driver doesn't recognize the NIC support correctly, or your NIC hardware doesn't support WOL.  If 'g' is supported, but not in 'Wake-on' line, then use ethtool to enable it:

```bash
ethtool -s eth0 wol g
```

## Modify PCI Power States

Generally, most savy user are able to traverse the forums and reach this point; we have turned on WOL from the NIC config, and checked that the linux driver sees our NIC WOL settings.  At this point, one would thinking halting the machine and sending a magic packet would work just fine... but no. Apparently, the driver for the Broadcom doesn't properly handle our explicit desire for WOL.

We check quickly which driver is being used for our NIC card; we do this by using 'lspci' with '-k' argument; this prints pci information along with drivers loaded detail to stdout

```bash
lspci -k
```

The response should be something similar to the below;  on this machine, we note that there are 2 NICs; both have same driver 'tg3'; and can be found at 02:00.0/02:00.1 or PCI bus 2 function 0 and 1 respectfully.

```text
00:00.0 Host bridge: Broadcom CMIC-LE Host Bridge (GC-LE chipset) (rev 33)
        Kernel modules: sworks-agp
00:00.1 Host bridge: Broadcom CMIC-LE Host Bridge (GC-LE chipset)
        Kernel modules: sworks-agp
00:00.2 Host bridge: Broadcom CMIC-LE Host Bridge (GC-LE chipset)
        Kernel modules: sworks-agp
00:0e.0 VGA compatible controller: ATI Technologies Inc Rage XL (rev 27)
00:0f.0 Host bridge: Broadcom CSB5 South Bridge (rev 93)
        Kernel modules: sworks-agp, i2c-piix4
00:0f.1 IDE interface: Broadcom CSB5 IDE Controller (rev 93)
        Kernel driver in use: Serverworks_IDE
        Kernel modules: serverworks
00:0f.2 USB Controller: Broadcom OSB4/CSB5 OHCI USB Controller (rev 05)
        Kernel driver in use: ohci_hcd
        Kernel modules: ohci-hcd
00:0f.3 ISA bridge: Broadcom CSB5 LPC bridge
00:10.0 Host bridge: Broadcom CIOB-E I/O Bridge with Gigabit Ethernet (rev 12)
        Kernel modules: sworks-agp
00:10.2 Host bridge: Broadcom CIOB-E I/O Bridge with Gigabit Ethernet (rev 12)
        Kernel modules: sworks-agp
00:11.0 Host bridge: Broadcom CIOB-X2 PCI-X I/O Bridge (rev 05)
        Kernel modules: sworks-agp
00:11.2 Host bridge: Broadcom CIOB-X2 PCI-X I/O Bridge (rev 05)
        Kernel modules: sworks-agp
01:04.0 RAID bus controller: LSI Logic / Symbios Logic MegaRAID (rev 01)
        Kernel driver in use: megaraid
        Kernel modules: megaraid_mbox
02:00.0 Ethernet controller: Broadcom Corporation NetXtreme BCM5704 Gigabit Ethernet (rev 02)
        Kernel driver in use: tg3
        Kernel modules: tg3
02:00.1 Ethernet controller: Broadcom Corporation NetXtreme BCM5704 Gigabit Ethernet (rev 02)
        Kernel driver in use: tg3
        Kernel modules: tg3
04:05.0 SCSI storage controller: LSI Logic / Symbios Logic 53c1030 PCI-X Fusion-MPT Dual Ultra320 SCSI (rev 07)
        Kernel driver in use: mptspi
        Kernel modules: mptspi
04:05.1 SCSI storage controller: LSI Logic / Symbios Logic 53c1030 PCI-X Fusion-MPT Dual Ultra320 SCSI (rev 07)
        Kernel driver in use: mptspi
        Kernel modules: mptspi
```

Assumingly, we could go forum hunting for patches to the tg3 driver (which there are lots of), and find a patch.  But a less evasive way is to simply force the PCI bus of the NIC card to remain powered when placed into the S5 halt state.  To do this, we use the knowledge acquired from above to match the PCI bus of the NIC and the 'acpitool'.  Since the 'acpitool' toggles the powered state of a specified PCI bus, we wrote a small bash script to first query the current S5 power state of the bus, and only change it if required.  This ensures that we don't accidently undo the setting if we happen to call this twice / or when the tg3 driver gets fixed, we don't prevent it from performing this change internally.  The script is as follows:

```bash
#!/bin/bash
 
#ensure acpi knows to keep device enabled during S5
#since acpitool acts as a toogle
#we only proceed if it needs enabling
wolKeyword="disabled"
pciNIC="pci0000:02"

wolEnabled=$(acpitool -w | grep "$pciNIC" | grep -o "$wolKeyword")
if [ "$wolEnabled" = "$wolKeyword" ]; then
  id=$(acpitool -w | grep "$pciNIC" | awk -F. '{print $1}' | sed 's/^[ t]*//')
  acpitool -W $id
fi
```

After running this script, running acpitool in query mode should detail the S5 powerstates of the PCI bus; something similar to the following:

```bash
acpitool -w
   Device       S-state   Status   Sysfs node
  ---------------------------------------
  1. PCI0         S5     disabled  no-bus:pci0000:00
  2. RTC          S5     disabled  pnp:00:08
  3. PCI3         S5     disabled  no-bus:pci0000:03
  4. PCI2         S5     enabled   no-bus:pci0000:02
  5. PCI1         S5     disabled  no-bus:pci0000:01
```

## Making this Automatic

So if you've been following along doing all this - shutting down linux, and sending the magic WOL packet will now wake your workstation properly; however, the next startup/shutdown will not keep your hard work.  To make this config automatic, we need to do a few simple things: 

1. Save the acpitool bash script into a new file at "/etc/init.d/wakeonlanconfig"
2. Add a post-up / post-down event for the ethenet card you are trying to setup WOL for; in our case it is 'eth0'

    ```bash
    nano /etc/network/interfaces
    ```

    ```text
    # This file describes the network interfaces available on your system
    # and how to activate them. For more information, see interfaces(5).

    # The loopback network interface
    auto lo
    iface lo inet loopback

    # The primary network interface
    #allow-hotplug eth0
    auto eth0
    #iface eth0 inet dhcp
    iface eth0 inet static
            address 192.168.0.252
            netmask 255.255.255.0
            network 192.168.0.0
            gateway 192.168.0.1
            broadcast 192.168.0.255
            post-up /usr/sbin/ethtool -s eth0 wol g
            post-down /usr/sbin/ethtool -s eth0 wol g
    ```

3. Use update-rc.d to add the wakeonlanconfig script to specific runlevels; specifically on halt

    ```bash
    update-rc.d wakeonlanconfig defaults
    ```

4. Reboot your workstation and verify that the acpi S5 pci bus is now enabled
5. Shutdown and verify that a WOL magic packet now works correctly
