---
title: Platform Instructions
keywords: vagrant, development, install
last_updated: April 04, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: platforms.html
folder: "argOS"
---

# General Instructions (local)
1. Build the OS depending on your platform
2. Get any prepared files if required (see *Pre-compiled files* section of each platform)
3. Prepare the SD card (see [Setup SD-Card](platforms.html#setup-sd-card))
4. Copy your project files to the SD Card

## Build the OS depending on your platform

According to the project, the following project parameters should be modified in [Makefile](https://github.com/argos-research/operating-system/blob/master/Makefile):

* `GENODE_BUILD_DIR` points to genode's building directory (e.g. `genode-focnados_panda`)
* `PROJECT` is the name of the project (e.g. `demo`)
* `GENODE_TARGET` is the target platform (e.g. `focnados_panda`) (see *Setup correct target device* section of each platform)

After modification, execute the following commands (unless otherwise specified):

```sh
$> make build_dir
$> sudo make run
```

## Setup SD-Card

To prepare a working SD card you can use the following example script (similar to [eewiki](https://eewiki.net/display/linuxonarm/PandaBoard#PandaBoard-SetupmicroSDcard)).

**We are assuming DISK=/dev/mmcblk0 as your sd card device**.

```sh
#!/bin/bash

if [[ "$1" == "--help" ]]; then
   echo "Please choose as first parameter the location of your MLO (./MLO)"
   echo "Please choose as second parameter the location of your 'u-boot.img'(./u-boot.img)"
   echo "If you dont choose any parameters the default ones(brackets) will be choosen"
   echo -e "\033[31m";
   echo "Use with care no input validation is implemented! This can damage your sd card!"
   echo -e "\033[30m";
   exit
fi

DISK=/dev/mmcblk0

echo "zero ${DISK}"
sudo dd if=/dev/zero of=${DISK} bs=1M count=10

#.MLO is now $1
echo "Install bootloader"
if [[ -z "$1" ]]; then
	sudo dd if=./MLO of=${DISK} count=1 seek=1 bs=128k
else
	sudo dd if=$1 of=${DISK} count=1 seek=1 bs=128k
fi

#./u-boot.img is now $2
if [[ -z "$2" ]]; then
	sudo dd if=./u-boot.img of=${DISK} count=2 seek=1 bs=384k
else
	sudo dd if=$2 of=${DISK} count=2 seek=1 bs=384k
fi

echo "Create Partition"
#uncomment the following lines if sfdisk <= 2.25.x
#sudo sfdisk --in-order --Linux --unit M ${DISK} <<-__EOF__
#1,,0x83,*
#__EOF__

#we're using  sfdisk >= 2.26.x
sudo sfdisk ${DISK} <<-__EOF__
1M,,0x83,*
__EOF__

echo "Format Partition"
sudo mkfs.vfat ${DISK}p1

echo "Sync"
sudo sync

echo "READY"

if [[ -z "$S1" ]]; then
  echo -e "\033[31m";
  echo "Caution: Standard parameters selected: if you did not intend to do this please check if everything is ok!"
  echo -e "\033[00m";
fi
```

## Copy your project files to the SD Card

* Mount your SD Card (e.g. `/dev/mmcblk0`) on `/mnt`

* Execute the following steps

```sh
$> cp $(GENODE_BUILD_DIR)/var/run/$(PROJECT)/image.elf /mnt
$> cp $(GENODE_BUILD_DIR)/var/run/$(PROJECT)/modules.list /mnt
$> cp -R $(GENODE_BUILD_DIR)/var/run/$(PROJECT)/genode /mnt
```

# Platform Specific Instructions

## PandaBoard ES

### Select correct target device

Change `GENODE_TARGET` inside the `Makefile` to `focnados_panda`

```make
GENODE_TARGET = focnados_panda
```

### Pre-compiled files
  * [U-boot Image](https://github.com/argos-research/operating-system/tree/master/Panda%20SD)


### References
Further (deeper) information about this topic can be found under the following links:

* [MLO & u-boot](http://elinux.org/Panda_How_to_MLO_%26_u-boot)

* [uboot PandaBoard ES](http://elinux.org/PandaBoard_ES_uboot_howto)

* [Linux on ARM](https://eewiki.net/display/linuxonarm/PandaBoard)


## Raspberry PI Model B+

### Select correct target device
Change `GENODE_TARGET` inside the `Makefile` to `focnados_rpi`

```make
GENODE_TARGET = focnados_rpi
```

### Flash (PXE Version)
**We are assuming DISK=/dev/mmcblk0 as your sd card device**.
```sh
sudo dd if=./RaspberryPiU-Boot.img of=/dev/mmcblk0 bs=4M
```

For further PXE/tftp instructions [pxe/tftp](pxe.html) section.

### Pre-compiled files
* [RaspberryPiU-Boot.img](https://nextcloud.os.in.tum.de/s/xAxEQYA57SIhnhz)

### References
* [Building U-Boot](http://wiki.beyondlogic.org/index.php?title=Compiling_uBoot_RaspberryPi)



## QEMU (PBXA9)
### Select correct target device

```make
GENODE_TARGET = focnados_pbxa9
```

### Building
Execute the following steps
```sh
sudo make vde
sudo make run
```

## Digilent Zybo
### Building u-boot
* Follow the following guide: [Booting Linux on the ZYBO](http://www.dbrss.org/zybo/tutorial4.html)

### Building Genode
**Zybo support is not yet integrated in the focnados-1608 branch, but in the focnados-zybo branch! **
```sh
./tool/create_builddir focnados_zybo
cd build/focnados_zybo
echo "RUN_OPT += --include image/uboot" >> etc/build.conf
echo "REPOSITORIES += \$(GENODE_DIR)/repos/hello_tutorial" >> etc/build.conf
echo "MAKE += -j4" >> etc/build.conf # increase number of jobs for make
make run/hello
```

### Running Genode
* Copy **boot.bin** and **uImage** to the SD-Card
* Boot up the Zybo with the SD-Card

**The following steps need to be done only once:**
* Interrupt the auto boot of u-boot and execute the following commands
```sh
sf probe
sf erase 0 0x1000000
env default -a
setenv bootcmd "echo Copying Fiasco.OC/Genode from SD to RAM... && fatload mmc 0:1 0x00100000 uImage && echo Booting Fiasco.OC/Genode... && bootm 0x00100000"
saveenv
boot
```

### Pre-compiled files
* [genode.img.tgz](https://nextcloud.os.in.tum.de/s/tyTDcIWpCMCnZ0J)
