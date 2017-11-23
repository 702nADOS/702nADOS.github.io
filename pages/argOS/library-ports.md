---
title: Library ports for Genode
keywords: genode, library, application
last_updated: June 13, 2017
tags: [development]
summary: ""
sidebar: main_sidebar
permalink: library-ports.html
folder: "argOS"
---

# Ports
All library ports we are using are either available directly in the [genode](https://github.com/argos-research/genode) repository or are part of the [genode-world](https://github.com/argos-research/genode-world) repository.
In order to use the world repository, simply copy the contents of the genode-world repository to `genode/repos/world/`.

## Prepare library
Since ports are based on the original source code of the library, we need to prepare the libraries we want to use:

```
cd <genode-dir>
./tool/ports/prepare_port libmosquitto
./tool/ports/prepare_port libprotobuf
```

This automatically downloads the original source code and prepares the libraries, meaning extracts the code if necessary and applies any required patches.

Since both libraries do have some dependencies on other libraries, we need to prepare some additional libraries:

```
cd <genode-dir>
# for libprotobuf
./tool/ports/prepare_port libc
./tool/ports/prepare_port stdcxx
./tool/ports/prepare_port zlib
# for libmosquitto
./tool/ports/prepare_port libc
./tool/ports/prepare_port stdcxx
./tool/ports/prepare_port lwip
./tool/ports/prepare_port openssl
```

## Testing a library
Normally libraries come with examples or tests, which can be executed in order to see if the library compiles successfully.
For the build system to find the genode-world repo, we first must introduce it in the `build.conf` file in `<build-dir>/etc/build.conf`, therefore comment out or add the following lines:

```
REPOSITORIES += $(GENODE_DIR)/repos/world
REPOSITORIES += $(GENODE_DIR)/repos/libports
```

To execute the test files for the libraries, use the following operations:
```
cd <build-dir>
make test/libprotobuf
make test/libmosquitto
```

If no errors occur, we are good to go.

## Using a library

One can now use the libraries more or less the same way one would use them on linux or windows. This also applies to libmosquitto, but not to libprotobuf.

In general one needs to make the following changes in its Genode application repository:

- Add **libprotobuf** and/or **libmosquitto** to the `LIBS` variable in `<application-dir>/src/<application-name>/target.mk` along with any other needed dependencies
    - For **libprotobuf**: **stdcxx pthread**
    - For **libmosquitto**: **stdcxx pthread lwip**
- Add **libprotobuf.lib.so** and/or **libmosquitto.lib.so** to `build_boot_image` in `<application-dir>/run/<application-name>.run` along with any other needed dependencies
    - For **libprotobuf**: **ld.lib.so libc.lib.so stdcxx.lib.so libm.lib.so pthread.lib.so**
    - For **libmosquitto**: **ld.lib.so libc.lib.so stdcxx.lib.so libm.lib.so pthread.lib.so lwip.lib.so libssl.lib.so libcrypto.lib.so**
- Add includes as needed to the src files in `<application-dir>/src/<application-name>/` e.g. `#include <mosquittopp.h>` for libmosquitto.

### libprotobuf
Protobuf normally comes with a compiler **protoc**, which takes \*.proto files and turns them into \*.pb.h and \*.pb.cc files - basically just \*.cc and \*.h files. For now if one wants to use protobuf, one has to compile the \*.proto files on his host machine and add them to the source tree of the Genode application. The files are then compiled into the binary and linked against the protobuf library.

### lwip
The lwip library may need the following addition/fix in `<application-dir>/src/<application-name>/target.mk`:
```
INC_DIR += $(REP_DIR)/../libports/include/lwip
```

# References
- [Writing an application for genode](https://genode.org/documentation/developer-resources/client_server_tutorial)
- [Genode Porting Guide: Overview](https://genode.org/documentation/developer-resources/porting)
- [argos-research/genode Repository](https://github.com/argos-research/genode)
- [argos-research/genode-world Repository](https://github.com/argos-research/genode-wold)
