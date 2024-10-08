# HLD Persistent MLOOP #


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
| 0.1  | 08/10/24 | Sophie Kravitz| Initial version                           |

# Motivation

Verification team needs the ability to set MLOOP per port for testing. It means, that the port will be in LOOP mode. Currently, when setting the port to be in LOOP mode, it only lasts until syncd reset. If syncd is restarted, MLOOP (that was previously set) is disabled. We want to be able to set it to be persistent.

# About this Manual

This document provides an overview of the implementation of adding the ability to set MLOOP persistently.

# Design

This ability is only for verification tests, so it needs to only be available in images for internal tests.
We need to add two components:
1. A service controlled by supervisord
2. A script that sets all the specified ports to MLOOP.

These two files need to be copied to syncd on the tested switch.

How set ports to MLOOP:

`persistent_mloop.py --ports port1,port2,..`

## MLOOP Script
To configure ports to MLOOP persistently, `persistent_mloop.py` will be called with a list of ports to be configured.
The script will translate the ports to logical ports, call the SDK api as was used before, and will also save the ports to a file. 

```
parse_ports()
ports_to_logical_ports()

for logical_port in mloop_ports:
    sx_api_port_phys_loopback.py --cmd 0 --log_port port --loopback_type 2 –force

save_ports()
```

When called from the service, the script will read the saved port list and preform the same flow as before: 
```
check_ports_to_configure()
ports_to_logical_ports()

for logical_port in mloop_ports:
    sx_api_port_phys_loopback.py --cmd 0 --log_port port --loopback_type 2 –force
```

## Supevisord Daemon
A new service will be added that will call the `persistent_mloop.py` script which will configure the saved ports to MLOOP.
The supervisord file will be copied to /etc/supervisord/conf.d/ in `mloop.conf`

```
[program:persistent_mloop]
command=persistent_mloop.py
stdout_logfile=/tmp/mloop.out.log
stderr_logfile=/tmp/mloop.err.log
autostart=true
autorestart=false
```

# Tests

## Manual Tests
Will conduct the following manual tests:
- Configure one port to MLOOP, check if it stays after restart.
- Configure a few ports to MLOOP, check if it stays after restart.
- Configure back to normal, check that it's normal after restart.
- 

# Documentation
A confluence page will be added.