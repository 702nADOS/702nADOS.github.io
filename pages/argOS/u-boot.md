---
title: U-Boot
keywords: u-boot, development, install
last_updated: October 09, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: u-boot.html
folder: "argOS"
---

# U-Boot
This page describes how to build (mainline) U-Boot for various platforms used by the argos-research project.
Already prepared images can be found [at the bottom of the page](#prepared-images).

## Preparation
```sh
# get u-boot source code
git clone --depth 1 --branch argos-research https://github.com/argos-research/u-boot.git
cd u-boot
# download and extract toolchain (gcc-linaro-5.4.1-2017.05-x86_64_arm-eabi.tar.xz)
curl https://nextcloud.os.in.tum.de/s/P7fKmyO2u5tRZS1/download | tar xJ
# temporarily add toolchain to path and set CROSS_COMPILE environment variable
export PATH=$(pwd)/gcc-linaro-5.4.1-2017.05-x86_64_arm-eabi/bin:$PATH
export CROSS_COMPILE=arm-eabi-
```

## Compile And Configure
### Hardkernel ODROID-U3
```sh
# select and compile for droid-u3
make odroid_defconfig
make
# download binary blobs (boot.tar.gz)
curl https://nextcloud.os.in.tum.de/s/0NWU5Wznfg5n7zE/download | tar xz
cd boot
# prepare the SD card to leave some space in the front,
# since the binary blobs and u-boot image will be copied to the beginning of it
(
  echo "o"       # create new dos partition layout
  echo "n"       # create new partition
  echo "p"       # primary
  echo "1"       # first partition
  echo "2097152" # first sector after 1 GB (sector size 512 bytes)
  echo "+8G"     # 8 GB size
  echo "t"       # change partition type
  echo "0c"      # w95 fat32 (lba)
  echo "w"       # write changes
) | sudo fdisk /dev/null # replace /dev/null with your SD card
sudo mkfs.fat -n GENODE /dev/null # replace /dev/null with the first partition of your SD card
# finally fuse u-boot onto the SD card
cp ../u-boot-dtb.bin u-boot.bin
sh sd_fusing.sh /dev/null # replace /dev/null with your SD card
```
#### TFTP boot
```sh
# configure an ethernet address
setenv usbethaddr 02:DE:AD:BE:EF:FF
# change bootcmd to boot via dhcp
# workaround for the usb ethernet - seems to work that way
setenv bootcmd 'usb reset; usb reset; dhcp ${scriptaddr}; usb reset; usb stop; bootelf ${scriptaddr}'
# save settings on the SD card
saveenv
```

### Raspberry Pi Model 1 B(+)
```sh
make rpi_defconfig
make
```

### Avnet ZedBoard
```sh
make zynq_zed_defconfig
make
```

### Digilent Zybo
```sh
make zynq_zybo_defconfig
make
```

## <a name='prepared-images'></a>Prepared Images
- [Hardkernel ODROID-U3](https://nextcloud.os.in.tum.de/s/66hmk6s8kxCjS2x/download)
- [Raspberry Pi Model 1 B(+)]()
- [Avnet ZedBoard]()
- [Digilent Zybo]()
