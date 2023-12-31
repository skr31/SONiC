# HLD NVIDIA Component Listing #



## Table of Content 

   -[Revision](#revision)

​	-[Motivation](#motivation)

​	-[About this Manual](#about-this-manual)

​	-[Design](#design)

​	-[Tests](#tests)
   
   -[Documentation](#documentation)


### Revision  

| Rev  |   Date   |    Author     | Change Description        |
| :--: | :------: | :-----------: | ------------------------- |
| 0.1  | 17/05/23 | Sophie Kravitz| Initial version           |

# Motivation

We want to have a file that gathers all the versions of the Nvidia components in an image in one place, that would be easy to access when needed.

# About this Manual

This document provides an overview of the implementation of adding file containing all the NVIDIA components.

# Design

The versions that need to be added to the file are:
- SDK
- FW
- SAI
- HW MGMT
- MFT
- Kernel
- BIOS
- SSD
- CPLD(s)

All the versions will be listed in a file that will be stored on the switch in /etc/mlnx/.
The file will be created in compilation at which point it will contain only the internal Nvidia components - SDK, FW, SAI, HW MGMT, MFT, Kernel - since the versions of the platform components will not be known at this stage.
The versions of the platform components will be added in the initialization flow.

The file will be accessed with cat command:
```
cat /etc/mlnx/component-versions
```

Example of a component-versions file:
```
     Component      |       Version
-------------------------------------------
SDK                 |  4.6.2134
FW                  |  2012.2134
SAI                 |  SAIBuild2311.26.0.28
HW-MGMT             |  7.0030.2008
MFT                 |  4.25.0
Kernel              |  5.10.0-23-2-amd64
BIOS                |  5.6.5
SSD                 |  
CPLD(s)             |  CPLD000087_REV0600_CPLD000075_REV0600
```

## Internal NVIDIA Components
The .mk files under sonic-buildimage/platform/mellanox/ export the versions of each of the Nvidia components: SDK, FW, SAI, HW-MGMT, MFT. There's also a kernel version variable that is accessable during compilation.
We will add an makefile with a target that outputs a file and write all the versions to it.

component-versions/Makefile:
```
.ONESHELL:
SHELL = /bin/bash
.SHELLFLAGS += -e

MAIN_TARGET = component-versions

$(addprefix $(DEST)/, $(MAIN_TARGET)): $(DEST)/% :
	echo $<COMPONENT_VERSION> >> $(DEST)/$(MAIN_TARGET)
```

component-versions.mk
```
COMPONENT_VERSIONS_FILE = component-versions
$(COMPONENT_VERSIONS_FILE)_SRC_PATH = $(PLATFORM_PATH)/component-versions

SONIC_MAKE_FILES += $(COMPONENT_VERSIONS_FILE)

MLNX_FILES += $(COMPONENT_VERSIONS_FILE)

export COMPONENT_VERSIONS_FILE
```

## Platform Components
The platform versions can be read using the fwutil command.

We will create a one shot service in `sonic_debian_extension.j2` that will only be added if the platform is mellanox.
```
{% if sonic_asic_platform == "mellanox" %}
sudo LANG=C chroot $FILESYSTEM_ROOT systemctl enable component-versions.service
{% endif %}
```

We need to handle the case of component upgrade as well.

## Techsupport
The versions file will also be collected at show techsupport.
```
collect_mellanox() {
   ...
   ...
   save_file /etc/mlnx/component-versions dump
}
```
The file will be placed under the `dump/` directory in the tar.

# Tests

## Unit Tests
The following unit tests will be added:


## Manual Tests

# Documentation


