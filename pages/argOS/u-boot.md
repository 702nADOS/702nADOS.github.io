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
This page describes how to build (mainline) U-Boot for various platforms, used by the argos-research project.
Already prepared images can be found at the bottom of the page.

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

## Compile From Source
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
cp ../u-boot-dtb.bin u-boot.bin
sh sd_fusing.sh /dev/null # replace /dev/null with your sd card
```

**Caution:** If you want to use TFTP boot on ODROID-U3, please take a look at...

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

## Prepared Images
- [Hardkernel ODROID-U3](https://nextcloud.os.in.tum.de/s/66hmk6s8kxCjS2x)
- [Raspberry Pi Model 1 B(+)]()
- [Avnet ZedBoard]()
- [Digilent Zybo]()
