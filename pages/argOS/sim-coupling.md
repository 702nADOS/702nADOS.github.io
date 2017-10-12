---
title: Simulator Coupling
keywords: vagrant, development, install
last_updated: April 24, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: sim-coupling.html
folder: "argOS"
---

# Quick Step Guide

In order to download, compile and start the simulation environment, run the following commands.

**Caution:** The following dependencies need to be installed on Ubuntu, in order to successfully run the script:
```sh
# General
sudo apt-get install tmux
# SUMO
sudo apt-get install -q=2 make g++ libxerces-c-dev libfox-1.6-dev automake libtool
# SpeedDreams 2
sudo apt-get install -q=2 make cmake g++ \
alsa-base alsa-utils pulseaudio pulseaudio-utils \
libopenscenegraph-dev libsdl2-dev libexpat1-dev libjpeg9-dev \
libplib-dev libopenal-dev libvorbis-dev libpng12-dev \
libenet-dev libboost-all-dev protobuf-compiler libprotobuf-dev
# SimCoupler
sudo apt-get install -q=2 make cmake g++ \
libboost-all-dev protobuf-compiler libprotobuf-dev
```

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

# File Structure
```
+-- src
|   +-- main.cc
|   +-- sd2.cc
|   +-- sumo.cc
|   +-- ...
+-- inc
|   +-- master_sim.hh
|   +-- sd2.hh
|   +-- sumo.hh
|   +-- ...
+-- protobuf-messages
+-- CMakeLists.txt
```

The `src` directory contains all modules of the SimCoupler and the loop itself.
If you want to add additional modules, just add them to the `src` directory.
To add them to the loop, you need to change the `main.cc` file,
which should only contain a minimum of code.
The protobuf-messages submodule contains all message definitions exchanged by the SimCoupler.

# Compilation
The SimCoupler can be compiled by issuing the following commands
in the directory of the SimCoupler:
```sh
mkdir -p build
cd build
cmake ../
make -j
```

# Dependencies
To run the simulation, a few repositories are needed

## General
Independent of the choice of the simulators you are using,
you need two repositories for the simulation:

- **SimCoupler:** The simulation coupler (SimCoupler) contains code for the main simulation loop.
- **protobuf-messages:** This repository contains the definition of all protobuf messages.

## Simulators
In our case there are currently two scenarios available:

### Autonomous driving
For the autonomous driving scenario currently two simulators are used
and are needed for the simulation:

- **SD2:** SpeedDreams2 (SD2) is our main simulator and performs the simulation steps.
- **SUMO:** SUMO provides a top-down view on the simulation
and is planned to be used as a middleware for the Veins simulator.

SUMO provides its own API, which only allows one client at a time.
We therefore wrote a Proxy to allow multiple connections to it,
which is accessible via the **TraCI-Proxy** repository
and will also be used by the SimCoupler.

### Robotics
For the robotics scenario the **V-REP** simulator is used.
By now *no* additional simulators are used by this scenario.

# Network connections
In order to work the SimCoupler opens a few Sockets
and/or connects to a few sockets.

In the current state the SimCoupler opens the following sockets:
* 127.0.0.1:9000 for SpeedDreams2 (SD2)

Additionally the SimCoupler connects to the following sockets:
* 127.0.0.1:2002 for TraCI-Proxy (SUMO)

Changes to addresses need to be done in the source code for now.

# Startup-Routine
In order to start the SimCoupler,
a few programs need to be started in order:

*For the autonomous driving scenario:*
1. SUMO
2. TraCI-Proxy
3. SimCoupler
4. SD2

# Program Flow
As an example, the flow for the autonomous driving scenario:
1. SD2 module is started:
  - opens a socket for a SD2 connection
  - waits for SD2 to transmit setup information (i.e. track)
2. SUMO module is started:
  - connects to the TraCI-Proxy
3. Now a loop is started:
  - SD2: perform simulation step and receive new simulation state
  - SUMO: perform simulation step and transmit information via TraCI-Proxy to SUMO

This loop can be easily changed to run a different main simulator
and add additional simulators to the coupler.

# Open points
- S/A VMs are not integrated into the SimCoupler
- Logging routine is missing
