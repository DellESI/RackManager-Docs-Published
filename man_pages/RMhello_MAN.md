Copyright 2016 Dell, Inc. All rights reserved.

# RMhello -- an example python utility 
---

## About
***RMhello*** is a simple python utility used as a dev / document example for the Dell-ESI-RM-Toolkit development.

***RMhello*** depends on Python34, parses options, and prints a Hello World message that is customizable based on options.

## Usage
```
RMhello -V   --Version            -- display version and return
RMhello -h   --help               -- display usage and return
RMhello [OPTIONS]                 -- display message customized by optional [OPTIONS] (see below)
```

### To run:
* Login as a user with Admin role on RackManager
* enter `RMhello [options]`
* Output:

  * By default, without options, the output message is:  `Hello World`
  * The output message can be customized with `-m` and `-U` option

### [OPTIONS]:
```
-h, --help                  --- display usage and exit
-V, --Version               --- display version and exit
-m <msg>, --message=<msg>   --- appends <msg> to the default message:   Hello World -- <msg>
-U, --User                  --- Includes username in the response message:  Hello World, <user>
```
---
## Installation, Path, and Dependencies:
(this  should be 4-5 short one-lineer bullets like below)
* ***RMhello*** is included in the rm-tools RPM, and is installed by default when the "Dell EMC RackManager Toolkit" group install package is installed on the RackManager
* The utility is part of the rackmanager-tools dev repo
* It is installed on the RM at `/opt/dell/rm-tools/RMhello/*`
* The first line of the program includes a sheebang line `#!/usr/bin/env python3` that will direct execution using the python3.4 executable
* The following dependent libraries are imported:

  *  python3.4 builtin sys, getopt, and os modules
  *  pwd module (if linux)
  *  getpass module (if pwd is not available-eg for Windows) 

---
## Examples:
(list the most common usage syntax using fenced code block.)
```
   RMhello -V                --- prints the version
           > Version: 1.0
   RMhello                   --- prints default hello message:
           > Hello World
   RMhello -m "my message"   --- prints message with custom message appended
           > Hello World -- my message
   RMhello -U                --- prints user name with message
           > Hello World, Admin
```

---
## See Also:
(list other utilities, commands, documents that might help the user do similar functions)
(eg the general man page for predefined roles, or underlying programs used eg racadm with link to manual)

  
## Limitations:
(list anything to note re limitations eg "you must be admin to run")
* Requires Admin level role
* if platform is windows, the username comes for getpass module and based on environment variables. it is spoofable

## Known Issues:
(list known issues (one liner). reference github issue number if there is one)
* See RackManager Toolkit-V1 Issues, label `RMhello`

