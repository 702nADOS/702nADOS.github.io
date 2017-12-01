---
title: Install
keywords: vagrant, development, install
last_updated: March 10, 2017
tags: [getting_started]
summary: ""
sidebar: main_sidebar
permalink: /docs/install/
folder: "argOS"
---

# Prerequisites

## System

* [ubuntu 16.04 lts](https://www.ubuntu.com/download/desktop)

## Packages

The following packages need to be installed (either from the corresponding
  website or via repository):

* [git](https://git-scm.com/)

* [oracle virtualbox 5.1.30](https://www.virtualbox.org)

* [vagrant 2.0.0](https://www.vagrantup.com)

# Quick Steps

## Operating System
The following steps bootstrap the virtual machine.
```sh
$> git clone https://github.com/argos-research/operating-system.git
$> cd operating-system
$> git checkout 1608
$> git submodule init
$> git submodule update
$> vagrant up
```

With the following commands we will login to the virtual machine and start the build process for our default target ([PandaBoard ES](http://pandaboard.org/)).
```sh
$> vagrant ssh
$> cd /vagrant
$> sudo make run
```

## Operating System on local machine
```sh
$> sudo apt-get update
$> sudo apt-get upgrade
$> sudo apt-get install libncurses5-dev texinfo autogen autoconf2.64 g++ \
   libexpat1-dev flex bison gperf cmake libxml2-dev libtool zlib1g-dev \
   libglib2.0-dev make pkg-config gawk subversion expect git libxml2-utils \
   syslinux xsltproc yasm iasl lynx unzip qemu tftpd-hpa libpixman-1-dev
$> git clone https://github.com/argos-research/operating-system.git
$> cd operating-system
$> git checkout 1608
$> git submodule init
$> git submodule update
$> make toolchain
$> make ports
$> make jenkins_build_dir
$> make jenkins_run
```

## QEMU/VDE
```sh
$> git clone https://github.com/mwilliamson/vde.git
$> cd vde/vde-2
$> autoreconf --install
$> ./configure
$> make
$> sudo make install
$> cd ../..
$> git clone https://github.com/qemu/qemu.git
$> cd qemu
$> mkdir build
$> cd build
$> ../configure
$> make -j
$> sudo make install
$> sudo ldconfig
```

## Testbed
For more information see [Simulatour Coupling](/sim-coupling.html) section
```sh
# clone the testbed repository
git clone https://github.com/argos-research/testbed.git
# change directory to testbed
cd testbed
# checkout the simcoupler branch
git checkout simcoupler
# change directory to startup-scripts
cd startup-scripts
# run the shell script
./sim_env.sh
```

# Troubleshoot

## Vagrant memory error (if physical memory is less than 4096 MB)

This exception arises from the configuration of virtualbox. Change the following settings in `Vagrantfile` to your needs.

```ruby
config.vm.provider "virtualbox" do |vb|
# Display the VirtualBox GUI when booting the machine
  vb.gui = false
# Customize the amount of memory on the VM:
  vb.memory = 4096
  vb.cpus = 2
```

# Next Steps

See [Plaftorm Instructions](/platforms.html)
