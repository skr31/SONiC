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
| 0.1  | 02/01/24 | Sophie Kravitz| Initial version           |

# Motivation

We want to have a file that gathers the versions of all Nvidia SONiC components for two reasons:
- Easily compare actual build vs qualified hash components (listed in SONiC release notes)
- Shorten debug time - one place for the support team to view all relevant information

# About this Manual

This document provides an overview of the implementation of adding a new file that holds NVIDIA SONiC components versions.

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
- CPLDs
- ONIE

All the versions will be listed in a file that will be stored on the switch in `/etc/mlnx/`.
The file will be created in compilation at which point it will contain only the internal Nvidia components - `SDK, FW, SAI, HW MGMT, MFT, Kernel` - since the versions of the platform components will not be known at this stage.
The versions of the platform components - `BIOS, SSD, CPLDs, ONIE` - will be added in the initialization flow.

This file will also be collected in techsupport for debugging purposes.

The file will be viewed with `cat` command:
```
cat /etc/mlnx/component-versions
```

Example of a `component-versions` file:
```
     Component      |       Version
---------------------------------------------
SDK                 |  4.6.2134
FW on image         |  2012.2134
FW burned on asic   |  2012.2134
SAI                 |  SAIBuild2311.26.0.28          
HW-MGMT             |  7.0030.2008                  // compilation
MFT                 |  4.25.0
Kernel              |  5.10.0-23-2-amd64     ________________________
BIOS                |  5.6.5
SSD                 |  0202-000
CPLD1               |  CPLD000087_REV0600           // init flow
CPLD2               |  CPLD000075_REV0600
ONIE                |  2022.08-5.3.0010-9600
```


## Internal NVIDIA Components
The .mk files under `sonic-buildimage/platform/mellanox/` export the versions of each of the Nvidia components: SDK, FW, SAI, HW-MGMT, MFT.
There is also an exported variable for the kernel version, outside of `sonic-buildimage/platform/mellanox/`.
We will add a makefile with a target that creates a file and writes all the versions to it.
The `slave.mk` file will not be affected by this.

The versions file will be created in `sonic-buildimage/platform/mellanox/component-versions/Makefile`:
```
.ONESHELL:
SHELL = /bin/bash
.SHELLFLAGS += -e

MAIN_TARGET = component-versions

$(addprefix $(DEST)/, $(MAIN_TARGET)): $(DEST)/% :
	echo $<COMPONENT_VERSION> >> $(DEST)/$(MAIN_TARGET)
    ...
```

The target above will be added to the compilation process in `sonic-buildimage/platform/mellanox/component-versions.mk`:
```
COMPONENT_VERSIONS_FILE = component-versions
$(COMPONENT_VERSIONS_FILE)_SRC_PATH = $(PLATFORM_PATH)/component-versions

SONIC_MAKE_FILES += $(COMPONENT_VERSIONS_FILE)

MLNX_FILES += $(COMPONENT_VERSIONS_FILE)

export COMPONENT_VERSIONS_FILE
```


## Platform Components
The platform versions can be read using the fwutil command.

We will create a service in `sonic-buildimage/files/build_templates/sonic_debian_extension.j2` that will only be added if the platform is mellanox.
```
{% if sonic_asic_platform == "mellanox" %}
sudo LANG=C chroot $FILESYSTEM_ROOT systemctl enable component-versions.service
{% endif %}
```

`component-versions.service`:
```
[Unit]
Description=Platform component listing
After=pmon.service

[Service]
Type=oneshot
ExecStart=/bin/bash /usr/bin/update-component-versions.sh
```

`update-component-versions.sh`:
```
fwutil show status > version_string

// format the version_string and update

version_string >> /etc/mlnx/component-versions
```

## Techsupport
The versions file will also be collected at show techsupport.

`generate_dump`:
```
collect_mellanox() {
   ...
   ...
   save_file /etc/mlnx/component-versions dump
}
```
The file will be placed under the `dump/` directory in the tar.

# Tests

## Manual Tests
We will conduct the following manual tests:
- Install image and check if the file was created correctly.
- Restart switch and check that the file is still correct.
- Change a component, restart and check that the file was updated.
- Compare fastboot with and without this feature.

# Documentation
We will update the user manual.


