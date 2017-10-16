---
title: U-Boot
keywords: u-boot, development, install
last_updated: October 16, 2017
tags: [development]
summary: "This page describes how to build (mainline) U-Boot for various platforms used by the argos-research project.
Already prepared images can be found [at the bottom of the page](#prepared-images)."
sidebar: main_sidebar
permalink: u-boot.html
folder: "argOS"
---

# Preparation
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

# Compile And Configure
## Hardkernel ODROID-U3
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
**No** files need to be copied on the SD card afterwards!
U-Boot was written at a specific location on the SD card by the script.

### TFTP boot
```sh
# configure an ethernet address
setenv usbethaddr 02:DE:AD:BE:EF:FF
# change bootcmd to boot via dhcp
# workaround for the usb ethernet - seems to work that way
setenv bootcmd 'usb reset; usb reset; dhcp ${scriptaddr}; usb reset; usb stop; bootelf ${scriptaddr}'
# save settings on the SD card
saveenv
reset
```

## Raspberry Pi Model 1 B(+)
```sh
# select and compile for rpi model 1 b(+)
make rpi_defconfig
make
# prepare SD card
echo "o\nn\np\n1\n\n+8G\nt\n0c\nw\n" | sudo fdisk /dev/null # replace /dev/null with your SD card
sudo mkfs.fat -n GENODE /dev/null # replace /dev/null with the first partition of your SD card
```

Plase copy the generated `u-boot.bin` file onto the SD card.

Afterwards create `config.txt` on the SD card with the following content:
```ini
kernel=u-boot.bin
enable_uart=1
init_uart_baud=115200
init_uart_clock=3000000 # this line is important
```

Additionally some files from the binary blobs need to be copied onto the SD card:
- bootcode.bin
- start.elf

Please get them from [here](https://github.com/raspberrypi/firmware/tree/1.20170811/boot).

### TFTP boot
```sh
# configure an ethernet address
setenv ethaddr 02:DE:AD:BE:EF:FF
# change bootcmd to boot via dhcp
setenv bootcmd 'usb reset; dhcp ${loadaddr}; bootelf ${loadaddr}'
# save settings on the SD card
saveenv
reset
```

## Avnet ZedBoard
```sh
# select and compile for rpi model 1 b(+)
make zynq_zed_defconfig
make
# prepare SD card
echo "o\nn\np\n1\n\n+8G\nt\n0c\nw\n" | sudo fdisk /dev/null # replace /dev/null with your SD card
sudo mkfs.fat -n GENODE /dev/null # replace /dev/null with the first partition of your SD card
```

Please copy the generated `spl/boot.bin`, `spl/u-boot-spl.bin` and `u-boot.img` files onto the SD card.

### TFTP boot
```sh
# configure an ethernet address
setenv ethaddr 02:DE:AD:BE:EF:FF
# change bootcmd to boot via dhcp
setenv bootcmd 'dhcp ${scriptaddr}; bootelf ${scriptaddr}'
# save settings on the SD card
saveenv
reset
```

## Digilent Zybo
```sh
# download and extra base_system_design.zip
curl -o zybo_base_system.zip https://nextcloud.os.in.tum.de/s/MprLJQ43K5vpgzw/download
unzip zybo_base_system.zip
# copy the init files into the u-boot file tree
cp zybo_base_system/source/vivado/SDK/hw_platform/ps7_init.h board/xilinx/zynq/zynq-zybo/ps7_init.h
cp zybo_base_system/source/vivado/SDK/hw_platform/ps7_init.c board/xilinx/zynq/zynq-zybo/ps7_init_gpl.c
# select and compile for rpi model 1 b(+)
make zynq_zybo_defconfig
make
# prepare SD card
echo "o\nn\np\n1\n\n+8G\nt\n0c\nw\n" | sudo fdisk /dev/null # replace /dev/null with your SD card
sudo mkfs.fat -n GENODE /dev/null # replace /dev/null with the first partition of your SD card
```

Please copy the generated `spl/boot.bin`, `spl/u-boot-spl.bin` and `u-boot.img` files onto the SD card.

### TFTP boot
```sh
# configure an ethernet address
setenv ethaddr 02:DE:AD:BE:EF:FF
# change bootcmd to boot via dhcp
setenv bootcmd 'dhcp ${scriptaddr}; bootelf ${scriptaddr}'
# save settings on the SD card
saveenv
reset
```

# Prepared Images<a name='prepared-images'></a>
Install the images via
```sh
# extract the *.tgz
tar xfz file.img.tgz
# dd' it onto the SD card
dd of=/dev/null if=file.img # replace /dev/null with your SD card
```
Every image is configured to use the `02:DE:AD:BE:EF:FF` ethernet address and boot via dhcp (expects a `*.elf` file!).

- [Hardkernel ODROID-U3](https://nextcloud.os.in.tum.de/s/66hmk6s8kxCjS2x/download)
- [Raspberry Pi Model 1 B(+)](https://nextcloud.os.in.tum.de/s/vVRFCh74B499P2W/download)
- [Avnet ZedBoard](https://nextcloud.os.in.tum.de/s/TsZNontFhgigiBg/download)
- [Digilent Zybo](https://nextcloud.os.in.tum.de/s/uNhs1VhfSsM0OdF/download)
