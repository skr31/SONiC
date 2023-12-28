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

When a costumer receives an Nvidia switch, he needs to be able to check that the versions of all the components are correct.
We want these versions to be listed in one place, making them more accessible to our costumers. 

# About this Manual

This document provides an overview of the implementation of adding file containing all the NVIDIA components.

# Design

The versions that need to be added to the file are:
- SDK
- SAI
- FW
- HW MGMT
- MFT
- Kernel
- BIOS
- SSD
- CPLD(s)

All the versions will be listed in a file that will be stored on the switch in /etc/mlnx/.
The file will be created in compilation at which point it will contain only the internal Nvidia components, since the versions of the platform components will not be known at this stage.
The versions of the platform components will be added in the initialization flow.

## Internal NVIDIA Components
The .mk files under sonic-buildimage/platform/mellanox/ export the versions of each of the Nvidia components: SDK, SAI, FW, HW-MGMT, MFT. There's also a kernel version variable that is accessable during compilation.
We will add a target that outputs a file and write all the versions to it.

```
component-versions:
   touch /etc/mlnx/component-versions
   echo $SDK_VERSION >> component-versions
   ...
   ...
```

## Platform Components
The platform versions can be read using the fwutil command.

We will create a one shot service in sonic_debian_extension.j2 that will only be added if the platform is mellanox.

## Techsupport
The versions file will also be collected at show techsupport.
```
collect_mellanox() {
   ...
   ...
   save_file component-versions /etc/mlnx/
}
```

# Tests

## Unit Tests
The following unit tests will be added:


## Manual Tests

# Documentation


