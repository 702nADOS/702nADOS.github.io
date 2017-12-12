---
title: DHCP/TFTP/PXE Configuration
keywords: Vagrant, development, install
last_updated: December 12, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: dhcp.html
folder: "argOS"
---

# DHCP and TFTP

The Vagrant machine already comes with an installed DHCP service called [isc-dhcpd](https://wiki.ubuntuusers.de/ISC-DHCPD/).
ISC-DHCP helps you to bootstrap your network, assign IP-addresses and distribute Genode images among your devices.

The DHCP service, as well as the TFTP service needed, is installed [here](install.html#operating-system-on-local-machine).
If it is not yet installed, please do so by executing:
```bash
sudo apt install isc-dhcp-server tftpd-hpa
```

<div class="alert alert-danger">
<p><b>Caution:</b></p>
<p>Please make sure, that you select a NIC device for bridging, which is only connected to a closed network - or even better just the hardware where you want to deploy Genode to. Otherwise the DHCP service running inside your Vagrant machine could influence the DHCP service of your network.</p>
</div>

If the services are installed, they are started while booting up the virtual machine within the script [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh).
You can crosscheck this by e.g. looking for the services in [htop](http://hisham.hm/htop/).

Before starting the Vagrant machine and the DHCP & TFTP services you can edit/configure the [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) in the config-data folder of the git root folder, which is automatically copied into the right directory after boot by the [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh) script.

The important parts of the configuration, which can be [edited](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf#L32-L206) are:

```conf
subnet 10.200.40.0 netmask 255.255.248.0 {
	range 10.200.40.10 10.200.40.240;
	default-lease-time 600;
	max-lease-time 7200;
	interface enp0s8;
}
### Fixed Dynamic Leases

# Pandaboard Section
host PandaboardES1  {
	hardware ethernet 0a:02:00:00:00:00;
	fixed-address 10.200.41.01;
	option host-name "pandaboardES1";
	filename "pandaboard/1/image.elf";
}

[...]

# this is the PXE-Boot for this subnet
next-server 10.200.40.1;
filename "/image.elf";
```

Theses parts describe the subnet which is used for the virtual/real network the DHCP should provide with addresses and TFTP/PXE information.

The subnet `10.200.40.0/21` was chosen, which means that IP-addresses in the range of `10.200.40.1` to `10.200.47.254` are possible.
This was done due to certain internal restrictions. Nonetheless it can be changed to your preferences but mind the static IP of the bridge interface defined in `/etc/network/interfaces` in Vagrant or your local machine, which has to be adapted as well. For Vagrant it is defined within [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L15).

## Fixed Leases & TFTP Boot

ISC-DHCP allows you to set fixed leases where certain MAC-addresses can be assigned certain IP-addresses even with additional options like TFTP servers and directories.

As embedded devices often do not provide a fixed MAC-address, we came up with a MAC-address schemata that is fitting for us based on [this](https://serverfault.com/questions/40712/what-range-of-mac-addresses-can-i-safely-use-for-my-virtual-machines):

```
  0A        -       xx        -      xx       -      xx       -      xx       -      xx
  \/                \/               \                                                 /
Project             Platform                                 Device
argos-research      Identifier                               Identifier

```

The Project itself is identified with the first 0A due to restrictions described in [here](https://serverfault.com/questions/40712/what-range-of-mac-addresses-can-i-safely-use-for-my-virtual-machines). 
The platform identifier can be anything from 00 to FF, nevertheless we use:
-  01 = reserved for virtual machines, e.g. any Vagrant/VM in the network
-  02 = used for Pandaboards
-  03 = used for Raspberry Pis
-  04 = used for Odroid devices
-  05 = used for Zed or Zybo boards
-  06 = used for virtual PBXA9 or Zybo boards / qemuVM

For every MAC-address section described above there is a certain /24 IP-Network reserved within the 10.200.40.0/21.
The partition is as follows:
-  01 -> `10.200.40.20-240` (default,fallback)
-  02 -> `10.200.41.0/24`
-  03 -> `10.200.42.0/24`
-  04 -> `10.200.43.0/24`
-  05 -> `10.200.44.0/24`
-  06 -> `10.200.45.0/24`

The first `/24` network `10.200.40.0/24` is used to host the virtual machines as well as the Vagrant machine itself on `10.200.40.1`, see [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L15).
The other subnets are only assigned if they are defined within the Fixed Leases block of the configuration [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) file.
The [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) already contains 5 entries for every platform defined above, meaning MAC-addresses from `0A:XX:00:00:00:00` to `0A:XX:00:00:00:04` are assigned to IP-addresses from `10.200.4X.01` to `10.200.4X.05`.

Each device will also be provided its unique boot image which is defined via the line:

``` filename "<platform>/<deviceNumberForPlatform>/<imagefile>"; ```

The next-server specifies the TFTP server's ip-address, where the file(name) can be found and is provided [here](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf#L204) via

``` next-server 10.200.40.1; ```

This means that every device will get its boot image from `10.200.40.1` but they are situated in different folders as they might be a different hardware platform or require different MAC-addresses
<div class="alert alert-info">
<p><b>Information:</b></p><p>MAC-address might be changed by NIC driver during Genode startup.</p>
</div>

If you want to specify different TFTP servers for each device, edit the per-device section with the next-server line (not tested) or see [isc-dhcp ubuntuusers](https://wiki.ubuntuusers.de/ISC-DHCPD/).

The folder structure that is announced in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf)  is implemented in [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L19) with [tftp-folder.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/tftp-folder.sh).

The TFTP services will start automatically during system startup and will take `/var/lib/tftpboot/` as root directory, such that files dropped there will be accessible through TFTP for device startup, when described in `tftpd.conf`.

Using this dynamic IP-address setup you only need to have a running [U-Boot](platforms.html) for your device platform, which starts with DHCP enabled and has the corresponding MAC-address set. It will automatically get the image file defined in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) for its MAC-address and boot it.
If you do not want to specify a MAC-address and start right away you will be provided with an IP-address in range of 10.200.40.20-10.200.40.240 and Lines 204 and 205 in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) will be your TFTP setting.
If there is something wrong with your config file but your DHCP server started normally you will also end up with this setting.
