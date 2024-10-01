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
| 0.1  | 23/09/24 | Sophie Kravitz| Initial version                           |

# Motivation

Verification team needs the ability to set MLOOP per port for testing. It means, that the port will be in LOOP mode. Currently, when setting the port to be in LOOP mode, it only lasts until syncd reset. If syncd is restarted, MLOOP (that was previously set) is disabled. We want to be able to set it to be persistent.

# About this Manual

This document provides an overview of the implementation of adding the ability to set MLOOP persistently.

# Design

This ability is only for verification tests, so it needs to only be available in images for internal tests.
We need to add two components:
1. A service controlled by supervisord
2. A script that sets all the specified ports to MLOOP.

## Supevisord Daemon


The supervisord file will be copied to /etc/supervisord/conf.d/ in `mloop.conf`
```
[program:persistent_mloop]
command=persistent_mloop.py
process_name=?
stdout_logfile=/tmp/mloop.out.log
stderr_logfile=/tmp/mloop.err.log
redirect_stderr=false
autostart=true
autorestart=false
startsecs=1?
numprocs=1?
```

## MLOOP Script

```
check_ports_to_configure()
ports_to_logical_ports()

for logical_port in mloop_ports:
    sx_api_port_phys_loopback.py --cmd 0 --log_port port --loopback_type 2 –force
```


# Tests

## Manual Tests
Will conduct the following manual tests:
- Configure one port to MLOOP, check if it stays after restart.
- Configure a few ports to MLOOP, check if it stays after restart.
- Configure back to normal, check that it's normal after restart.
- 

# Documentation
