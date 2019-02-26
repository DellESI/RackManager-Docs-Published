Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMg5cli -- Legacy G5 Commandline Utility
---

## About
***RMg5cli*** is a BASH utility that allows a user to run G5 CLI commands from the RackManager.

* If a specific MC CLI sub-command is not specified, an interactive MC CLI shell is started.
* The privilege of the user depends on the RackManager Role Group that the user is a member of:
  * users with Administrator privilege on the RM will execute commands with Admin privilege on the MC
  * users with Operator privilege on the RM will execute commands with Operator privilege on the MC
  * users with ReadOnly privilege on the RM will execute commands with ReadOnly privilege on the MC 
* Commands are executed on the targeted G5 Managed MC (MMC)
  * The default MMC is MMC1 - if no MMC is specified
    * in many cases, there is only one MMC in a rack, so MMC1 is the only MMC
    * MMC1 will always be valid
  * MMCs are numbered from bottom up in the rack:  MMC1, MMC2, MMC3,...

## Usage
```
RMg5cli -V                                        -- display version and returns
RMg5cli -h                                        -- display help and usage info and exit
RMg5cli  [-m <mmc>] [-v]                          -- connects to MMC <mmc> and starts an interactive MC CLI shell
                                                     <mmc> is MMC1 by default
RMg5cli  [-m <mmc>] [-v] <sub-command and args>   -- run the specified sub-command and args on <mmc> and return
```

### To run:
* Login to the Rackmanager via ssh
* enter `RMg5cli [options]`

### Options:
```
-V         --- display the version and exit
-h         --- display help and usage info and exit
-m <mmc>   --- <mmc> is the alias in the RackManager hosts file for the targeted MC:
               valid values:   MMC1, MMC2, MMC3, MMC4, ...
-v         --- verbose flag.  can be repeated multiple times for more verbose output:  
          -v      ---gives the RM user group that this user is a member of
          -vv     ---adds the ssh command sent to the MC,   
          -vvv    ---adds progname, version, & verboseLvl
```

## Installation, Path, and Dependencies:
* ***RMg5cli*** is included in the rackmanager-tools RPM, and is installed by default when the RackManager Toolkit is installed
  * The utility is part of the `rackmanager-tools` development repo
  * The utility and data is installed at `/opt/dell/rm-tools/RMg5cli/*`
  * A wrapper script `RMg5cli` is placed in `/opt/dell/rm-tools/bin` by the RMconfig subsystem RMbase
  * The first line of the program includes a shebang line `#!/usr/bin/bash` that will direct execution using standard linux bash

## Examples:
```
   RMg5cli -V                --- prints the version and exit
   RMg5cli -h                --- prints help info and exit
   RMg5cli                   --- start interactive legacy G5 CLI on MMC1
   RMg5cli -m MMC2           --- start interactive legacy G5 CLI on MMC2 (the managed MC in management domain 2)
   RMg5cli SHOW /DEVICEMANAGER/RACK1/Block1/Sled1  --- display properties and targets for Block1/Sled1 managed by MMC1
   RMg5cli -m MMC2 SHOW /DEVICEMANAGER/Rack1/Block2  -- display properties and targets for Block2 managed by MMC2
```
  
## Limitations:
* Users must be root, or in one of the RM user groups RM_ADMIN, RM_OPER, or RM_READONLY
  * THese groups will be mapped to MC users:  rackmanager_adm, rackmanager_oper, and rackmanager_readonly
* Only "managed" MCs can be targeted. You cannot target an unmanaged MC that monitors a 2nd-ary powerbay
  * use raw ssh user@<MCx_y> to connect to a non-managed MC for development or debug
* When using RMg5cli with no arguments, and using the interactive CLI, if you reset the MC during this session
  you will lose connection to the MC and may see a broken pipe error, but the reset will complete as desired
* RMg5cli does not create any logs on the RackManager since the actual events happen on the MC

## Known Issues:
* None

