# HLD NVIDIA Component Listing #



## Table of Content 

   -[Revision](#revision)

​	-[Motivation](#motivation)

​	-[About this Manual](#about-this-manual)

​	-[Design](#design)

​	-[Tests](#tests)
   
   -[Documentation](#documentation)


### Revision  

| Rev  |   Date   |    Author     |       Change Description                  |
| :--: | :------: | :-----------: | ------------------------------------------|
| 0.1  | 02/01/24 | Sophie Kravitz| Initial version                           |
| 0.2  | 02/01/24 | Sophie Kravitz| Changed file format and removed service   |

# Motivation

We want to have a script that outputs the versions of all Nvidia SONiC components for two reasons:
- Easily compare actual build vs qualified hash components (listed in SONiC release notes)
- Shorten debug time - one place for the support team to view all relevant information

# About this Manual

This document provides an overview of the implementation of adding a new file that holds NVIDIA SONiC components versions.

# Design

The versions that need to be printed by the script are:
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

All the versions will be printed to the screen by the versions script.
A file will be created in compilation which will contain only the internal Nvidia components - `SDK, FW, SAI, HW MGMT, MFT, Kernel` - since the versions of the platform components will not be known at this stage.

Some of these versions can be changed after the SONiC installation, so there will be another column in the output with the actual version that is installed at that moment on the switch. This column will be added on the fly to the output of the script.
The versions of the platform components - `BIOS, SSD, CPLDs, ONIE` -  will also be added on the fly each time the version generation script is called.

The compilation file and the output of the script will also be collected in techsupport for debugging purposes.

The versions will be printed to the screen when executing the script:
```
component-versions.sh
```

Example of a `component-versions.sh` output:
```
     Component      |       compilation      |     actual
-----------------------------------------------------------------
SDK                 |  4.6.2134              |  4.6.2134  
FW                  |  2012.2134             |  2012.2134
SAI                 |  SAIBuild2311.26.0.28  |  SAIBuild2311.26.0.28        
HW-MGMT             |  7.0030.2008           |  7.0030.2008       
MFT                 |  4.25.0                |  4.25.0    
Kernel              |  5.10.0-23-2-amd64     |  5.10.0-23-2-amd64
BIOS                |                        |  5.6.5 
SSD                 |                        |  0202-000 
CPLD1               |                        |  CPLD000087_REV0600      
CPLD2               |                        |  CPLD000075_REV0600
ONIE                |                        |  2022.08-5.3.0010-9600
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
Each of the internal Nvidia components need to be collected from different places.

`update-component-versions.sh`:
```
fwutil show status > version_string

// format the version_string 
// get the internal component versions

echo version_string
```

## Techsupport
The versions script will also be executed at show techsupport.

`generate_dump`:
```
collect_mellanox() {
   ...
   ...
   save_command component-versions.sh dump
}
```
The file will be placed under the `dump/` directory in the tar.

# Tests

## Manual Tests
We will conduct the following manual tests:
- Install image and check if the file was created correctly.
- Change a component, check that the file was updated.
- Check that techsupport collects the file and the script output.
- Compare techsupport with and without this feature.

# Documentation
We will update the user manual.

