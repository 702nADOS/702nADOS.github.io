---
title: DHCP/TFTP/PXE Configuration
keywords: vagrant, development, install
last_updated: April 04, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: dhcp.html
folder: "argOS"
---


# DHCP and TFTP

The vagrant machine comes with an already install DHCP service called isc-dhcpd ( https://wiki.ubuntuusers.de/ISC-DHCPD/ ).
ISC-DHCP helps you to boot up your network and fill it with ip addresses and the distribution of genode images among your devices.

The DHCP service as well as the TFTP service needed is installed at [here](install.html#operating-system-on-local-machine).
If it is not yet install please do so with:
```
sudo apt install isc-dhcp-server tftpd-hpa
```


If the services are installed there are normally started at boot up of the virtual machine within the script [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh).
Please mind that the TFTP service is started automatically. You can crosscheck this by searching in [htop](http://hisham.hm/htop/) for it for example.

Before starting the vagrant machine and the DHCP and TFTP services you can edit/configure the [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) in the config-data folder at git root folder, which is copied into the right directory after boot within the [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh) execution.

The important part of the configuartion, which can be [edited](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf#L32-L206) is:

```

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
```
...
```
# this is the PXE-Boot for this subnet
next-server 10.200.40.1;
filename "/image.elf";
```


This describes the subnet which is used for the virtual/real network the dhcp should polute with addresses and TFTP/PXE information.

The subnet 10.200.40.0/21 is chosen which means that IP-addresses in the range of 10.200.40.1-10.200.47.254 are possible.
Due to certain internal restrictions this subnet is chosen it can be changed to your preferences but mind the static IP of the bridge interface defined in /etc/network/interfaces in vagrant or your local machine has to be adapted as well. For vagrant it is defined within [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L15).

##Fixed Leases & TFTP Boot

ISC-DHCP allows you to set fixed leases where certain MAC-addresses can be assigned certain IP-addresses even with aditional options like TFTP servers and directories.

As embedded devices do often not provide a MAC-address we come up with a MAC-address schemata that is fitting for us based on [this](https://serverfault.com/questions/40712/what-range-of-mac-addresses-can-i-safely-use-for-my-virtual-machines):

```
  0A        -       xx        -      xx       -      xx       -      xx       -      xx
  \/                \/               \                                                 /
Project             Plattform                               Device
argos-research      Identifier                               Identifier

```
The Project itself is identified with the first 0A due to restrictions described in [here](https://serverfault.com/questions/40712/what-range-of-mac-addresses-can-i-safely-use-for-my-virtual-machines). 
The plattform identifier can be anything from 00 to FF, nevertheless we use:
-  01 = reserved for virtual machines, e.g. any vagrant/VM in the network
-  02 = used for Pandaboards
-  03 = used for Raspberry Pis
-  04 = used for Odroid devices
-  05 = used for Zed or Zybo boards
-  06 = used for virtual PBXA9 or Zybo boards / qemuVM

For every MAC-address section described above there is a certain /24 IP-Network reserved within the 10.200.40.0/21.
The partition is as follows:
-  01 -> 10.200.40.20-240 (default,fallback)
-  02 -> 10.200.41.0/24
-  03 -> 10.200.42.0/24
-  04 -> 10.200.43.0/24
-  05 -> 10.200.44.0/24
-  06 -> 10.200.45.0/24

The first /24 network 10.200.40.0/24 is used to host the virtual machines as well as the vagrant machine itself on 10.200.40.1, see [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L15).
The other subnets are only assigned if they are defined withing the Fixed Leases Block of the configuration [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) file.
The [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) already contains 5 entrys for every plattform defined above so MAC-Addresses from 0A:XX:00:00:00:00 to 0A:XX:00:00:00:04 . IP-Addresses which are assigned are 10.200.4X.01 to 10.200.4X.05.

Each device will also be provided its unique boot image which is defined via the line:

``` filename "<plattform>/<deviceNumberForPlattform>/<imagefile>"; ```

The next-server meaning the TFTP server where the file(name) can be found is provided [here](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf#L204) via

``` next-server 10.200.40.1; ```


This means that every device will get its boot image from 10.200.40.1 but they are situated in different folders as they might be a different hardware plattform or require different MAC-addresses (!Please keep in mind: MAC-Address might be changed during Genode startup!).

If you want to specify different TFTP servers for each device edit the per-device section with the next-server line (not tested) or see [isc-dhcp ubuntuusers](https://wiki.ubuntuusers.de/ISC-DHCPD/).


The folder structure that is announced in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf)  is implemented in [bootstrap.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/bootstrap.sh#L19) with [tftp-folder.sh](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/tftp-folder.sh).

The TFTP services will start automatically during system startup and will take /var/lib/tftpboot/ as root directory so files dropped there will be accessable through TFTP for device startup. 

Using this dynamic IP-Address setup you only need to have a running [UBOOT](platforms.html) for your device plattform which starts with dhcp enabled and has the corresponding MAC-Address set. It will automatically get the image file defined in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) for its MAC-Address and boot.
If you do not want to specify a MAC-Address and start right away you will be provided with an IP-Address in range of 10.200.40.20-10.200.40.240 and Lines 204 and 205 in [dhcpd.conf](https://github.com/argos-research/operating-system/blob/d700f0251ab2f1102272c821bb0d67f56ecd09ed/config-data/dhcpd.conf) will be your TFTP setting.
If there is something wrong with your config file but your DHCP server started normal you will also end up with this setting.








