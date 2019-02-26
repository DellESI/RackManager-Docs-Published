Copyright &copy; 2017-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMbiosupdate -- utility to flash RackManager's BIOS ROM

---

## About

***RMbiosupdate*** is a python 3.4 based utility that makes it easy to flash the RackManager's BIOS ROM with a new firmware image.
* depends on Python 3.4, Flashrom, and flash_util
* parses command arguments, and flashes the specified image to the BIOS ROM
* currently only supports flashing of Dell ESI's DSS 9000 RackManager
* logs events to `/var/log/rackmanager/RMbiosupdate.log`

## Usage

```
RMbiosupdate -V              -- display version and return
RMbiosupdate -h              -- display usage and return
RMbiosupdate -i <image>      -- flash the specified image to the RM's BIOS ROM
RMbiosupdate -r -i <image>   -- flash the specified image to the RM's BIOS ROM and auto-reboot
```

### To run:

* Login as root on RackManager
* Enter `RMbiosupdate [options]`
* Output:
  * -h option will print the usage
  * -V option will print the RMbiosupdate version
  * -i option will attempt to flash the specified image, displaying progress and completion status
  * -r option will auto-reboot (power cycle) the RackManager upon a successful flash

### [OPTIONS]:

```
-h, --help                    --- display usage and exit
-V, --Version                 --- display version and exit
-i <image>, --image=<image>   --- flash the specified image to the RM's BIOS ROM
-r, --reboot                  --- auto reboot (power cycle) after successful flash
```
---

## Installation, Path, and Dependencies:

* ***RMbiosupdate*** is included in the rackmanager-tools RPM, and is installed by default when the Dell ESI RackManager Toolkit is installed on the RackManager
* The utility is part of the rackmanager-tools development repo
* It is installed on the RM at `/opt/dell/rm-tools/RMbiosupdate/*`
* RMbiosupdate specific logs can be found at `/var/log/rackmanager/RMbiosupdate.log`
* The first line of the program includes a shebang line `#!/usr/bin/env python3` that will direct execution using the python3.4 executable
* The following dependent libraries are imported:
  *  python 3.4 built-in sys, getopt, subprocess, time, logging, pwd, and os modules

---

## Examples:

```
   RMbiosupdate -V
           > RMbiosupdate: Version: 0.3
           
   RMbiosupdate -h
           > usage: RMbiosupdate [-V][-h][-i <image>]
           >   OPTIONS:
           >     -h, --help                   # prints usage and exit
           >     -i <image>, --image=<image>  # flashes the specified image
           >     -V, --Version                # prints program version and exits
           >
           > Warning: If flashing a new bios image without using the -r option to auto power cycle,
           >          the new image will not take effect until the RackManager is power cycled. Please
           >          be careful as this is not the same as a reboot. Rebooting will cause a shutdown
           >          but the RackManager will never boot back up until fully power cycled.
           >
           > Warning: If using RMbiosupdate utility in a PXE environment, please be careful
           >          that you do not have a shared folder between PXE nodes. This may
           >          cause the update to scramble DMI information.
           
   RMbiosupdate -r -i SILVERSHADOW-01.00.00.04-nodebug.rom
           > RMbiosupdate: Using BIOS ROM Image = SILVERSHADOW-01.00.00.04-nodebug.rom
           > RMbiosupdate: Please do not remove power from the system during flash.
           > RMbiosupdate: RackManager will auto reboot upon successful flash.
           > RMbiosupdate: Flashing...
           > RMbiosupdate: Flash completed successfully.
           > RMbiosupdate: Rebooting RackManager...
```

---

## Limitations:
* currently only supports flashing of Dell ESI's DSS9000 RackManager
* requires root user
* requires Flashrom
* requires flash_util
* requires a full power cycle after execution if the -r option is not used

## Known Issues:
* If you do not use the -r option to auto-reboot (power cycle) during execution, and then attempt to do
  a normal reboot afterwards, the RM will shutdown as expected, but will not boot back up until a full
  power cycle is performed.
