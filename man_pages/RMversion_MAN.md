Copyright 2017 Dell, Inc. All rights reserved.

# RMversion -- utility to display RackManager Toolkit version

---

## About

***RMversion*** is a python 3.4 based utility that will display the version of the RackManager Toolkit installed.
* depends on Python 3.4
* parses command arguments, and displays version
* reads version from `/etc/opt/dell/rm-tools/rmtk_version.txt`

## Usage

```
RMversion -h              -- display usage and return
RMversion                 -- display version of installed RackManager Toolkit
```

### To run:

* enter `RMversion [-h]`

### [OPTIONS]:

```
-h, --help                    --- display usage and exit
```
---

## Installation, Path, and Dependencies:

* ***RMversion*** is included in the rackmanager-tools RPM, and is installed by default when the Dell RackManager Toolkit is installed on the RackManager
* The utility is part of the rackmanager-tools dev repo
* It is installed on the RM at `/opt/dell/rm-tools/RMversion/*`
* The first line of the program includes a sheebang line `#!/usr/bin/env python3` that will direct execution using the python3.4 executable
* The following dependent libraries are imported:
  *  python3.4 builtin sys, getopt, and os modules

---

## Examples:

```        
   RMversion -h
           > usage: RMversion   [OPTIONS]
           >    [OPTIONS]
           >       -h, --help              # prints usage and exits
           
   RMversion
           > RackManager Toolkit 1.0.0
```

---

## Limitations:
* `/etc/opt/dell/rm-tools/rmtk_version.txt` must exist and contain valid data

## Known Issues:
* None
