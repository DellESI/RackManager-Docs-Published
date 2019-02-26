Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMg5update -- a BASH utility used to update the G5 infrastructure firmware
---

## About

***RMg5update*** is a BASH command script that allows a user to update firmware and config files for the G5.5/DSS9000 infrastructure controllers (MCs, BC, IMs, and G5switches) from the RackManager.

* depends on bash, tftp, and a stable communication with the Managed MC.
* parses command arguments, and performs FW updates of the G5.5/DSS9000 infrastructure controllers
* tightly integrated with the G5.5/DSS9000 internal management infrastructure
* logs events to `/var/log/rackmanager/RMg5update.log`

***RMg5update*** supports updates using a few different update package options:
* A generic update package - can be used to update all controllers (MCs, BC, IMs, and G5switches)
  * File name must end with .tar, .tgz, or .tar.gz
  * Use of a generic update package also requires a Rackconfig file unique to the Rack in use
    * Rackconfig file name can be anything
* A custom update package - can be used to update MCs, BC, and IMs
  * File name must be of format `G5_<packageName>-<ver>.tgz`
* A switch firmware .flash file - can be used to update the G5switches
  * File name must be of format `G5_stark_fw_v<ver>.flash`

## Usage

```
Usage:
     RMg5update -V       -- display version and return
     RMg5update -h       -- display usage and return
     RMg5update [OPTIONS] -R <rackcfg> <RMpackage>
                          -- update G5 Firmware from generic RM package based on rack configuration <rackcfg>,
                          <RMpackage> is path to a generic RM G5 FW package: /path/RM_G5FW_VERSION-<ver>.tgz,
                          <rackcfg> is name of a Rackconfig file in /opt/dell/rm-tools/RMg5update/Rackconfigs/
     RMg5update [OPTIONS] [-C] <G5Package>
                          -- update G5 Firmware from a custom G5 package
                          <G5package> is path to a custom G5 FW package: /path/G5_VERSION-<ver>.tgz
     RMg5update -D 1 -t G5ALLSWITCH <RMpackage>
                          -- update all switches in mgt domain 1 with switch firmware from generic RM package,
                          <RMpackage> is path to a generic RM G5 FW package: /path/RM_G5FW_VERSION-<ver>.tgz
     RMg5update -t G5IMSWITCH <SwitchFW>
                          -- update the IM switch in default mgt domain (1) with specified firmware file,
                          <SwitchFW> is path to Switch FW file: /path/G5_stark_fw_v<ver>.flash
                          NOTE: in order to specify the switch fw file directly, the target must
                                only include switches

   Options:
        -h              --- display usage and exit
        -V              --- display version and exit
        -I              --- displays info about the update and package specified and verifies that options
                            and dependencies are met, but does not update any actual firmware
        -R <rackcfg>    --- include Config files based on <rackcfg>,
                            <rackcfg> must be file located in /opt/dell/rm-tools/RMg5update/Rackconfigs/
        -C              --- include Config files if in package - always done if -R option specified,
                            thus should not be used with -R
        -t <target>     --- specifies the update target - the default is G5MCIMBC.
                            <target>={ G5ALL, G5MCIMBC, G5ALLSWITCH, G5IMSWITCH, G5BLKSWITCH}
                            - G5ALL -- updates all MCs, BCs, IM, and G5switches in the mgt domain.
                            - G5MCIMBC -- default, only update the MCs, BCs, and IMs
                            - G5ALLSWITCH -- updates all G5switches in the mgt domain
                            - G5IMSWITCH -- only update the G5 IM switch
                            - G5BLKSWITCH -- updates all G5 Block Switches
        -D <mgtDom>    --- specifies the management domain: <mgtDom>=[1:4], default is 1
        -r <rackNum>   --- specifies the specific rack in the management domain, default is 1
        -v             --- verbose output - can repeat for additional level of verbosity
                            -v     -- verbose level-1 shows general progress as the update proceeds
                            -vv    -- verbose level-2 shows detailed progress as the update proceeds
                            -vvv   -- verbose level-3 dumps detailed debug info during execution
        -M             --- update the mc.conf and redfish.conf on the MC from MC.conf and G5Redfish.conf on
                           the RM after all other updates to insure that the managed MC has RM config settings
                           
   Notes:
       * Following any update with RMg5update, RMconfig of the RMg5mc subsystem should be executed in order
         to re-sync the RM with the MC.
         * RMconfig [-vvv] -F -s RMg5mc config
```

### To run:

* login as a user with Admin permissions on the RackManager
* copy the package file to a temp file location on the RM using scp
  * ex: from the RM, copy the package from another server to the RM
  *   `scp <mylogin>@<myServer>:/path/to/myPkgFiles/<packagename>  /var/g5updates/.`
* enter `RMg5update [OPTIONS] /var/g5updates/<packagename>`
* once complete, run RMconfig on the RMg5mc subsystem
  * `RMconfig [-vvv] -F -s RMg5mc config`

### Examples:

```
   RMg5update -V                  --- prints the version and exits
           > Version: 1.0

   RMg5update -h                  --- prints the usage similar to MAN page usage section

   RMg5update -v -R G55_HW_10BLK_2PB  $myFwPkgs/RM_G5FW_VERSION-3.32.tgz
                                  --- update G5 with level-1 verbose messages for rack config G55_HW_10BLK_2PB
                                      to ver 3.32 firmware

   RMg5update -I -R G55_HW_10BLK_2PB  $myFwPkgs/RM_G5FW_VERSION-3.32.tgz 
                                  --- verifies that the specified pkg and rackcfg exists
                                      before starting the update, will not perform the update
                                      
   RMg5update -vvv -t G5ALLSWITCH G5_stark_fw_v1.10.0.flash
                                  --- update all G5 switch fw using the G5_stark_fw_v1.10.0.flash fw file,
                                      and display verbose output
```

## Updating with Generic G5 Firmware Packages

A generic G5 firmware package should contain:
* All IM firmware and conf files (for all rack configs)
* All BC firmware and conf files (for all rack configs)
* All MC firmware and conf files (for all rack configs)
* All G5 switch firmware
  
The package itself can have any file name that ends with .tar, .tar.gz, or .tgz

When using the generic package, the `-R <Rackcfg> ` option must be specified
* With the `-R <Rackcfg> ` option, the RM will create a specific HW-dependent "G5-package" that is ultimately sent to the MC
  and used to perform the update
  * This HW-dependent package is created based on the Rackcfg specified with the -R option
    * Note that the specified <Rackcfg> must be located in `/opt/dell/rm-tools/RMg5update/Rackconfigs/`
  * Once created, RMg5update will then copy the package to the `/tftpboot/` directory
  * Then a ssh command is sent to the MC telling it to download the package via tftp and perform the update
    * Note that the MC does the actual update process here

## Updating with Custom G5 Firmware Packages

A custom G5 firmware package must contain only the exact files required for that specific rack configuration, as it will be used "as is" to perform the update

The package itself must have a file name in the form of: `G5_<packageName>-<ver>.tgz`

When using the custom package:
* the `-R <Rackcfg> ` option should never be used
* the -C option may or may not be used
  * The `-C` option tells RMg5update to include any Hardware Config files that were in the manually created package. Otherwise config files will be stripped before sending the package to the MC. 

## Updating with Switch Firmware Directly

Updating G5 switches with the switch firmware directly requires the fw file in use to be the desired fw version.

The fw file itself must have a file name in the form of: `G5_stark_fw_v<ver>.flash`

When using the switch fw directly:
* the `-R <Rackcfg> ` option should never be used
* the -C option should never be used
* the -t <target> must be used to specify switch targets
  * <target> must be one of:
    * G5ALLSWITCH - all switches (including IM switch and all BCDB switches) will be updated to the specified firmware
    * G5IMSWITCH - just the IM switch will be updated to the specified firmware
    * G5BLKSWITCH - all BCDB switches will be updated to the specified firmware

## Installation, Path, and Dependencies:
* ***RMg5update*** is included in the rackmanager-tools RPM, and is installed by default as part of the RackManager Toolkit
* It is installed on the RM at `/opt/dell/rm-tools/RMg5update/*`
* The first line of the program includes a sheebang line `#!/usr/bin/bash`
* RMg5update specific logs can be found at `/var/log/rackmanager/RMg5update.log`
* Following any update with RMg5update, RMconfig of the RMg5mc subsystem should be executed in order
  to re-sync the RM with the MC.
  * RMconfig [-vvv] -F -s RMg5mc config

## See Also:
* RMconfig

## Known Issues:
* When doing an update with target G5ALL or G5MCIMBC, during the update process of the IM the RackManager will lose power twice. This causes loss of output of the update progression; however, the update process, as it is being done by the MC, will still continue on as expected and the update should still take effect for the MC/IM/BC firmware. If target is G5ALL though, because of this loss of power, RMg5update will not get to the switch updates. Thus, you should then update just the switches afterwards.
  * The status of the MC/IM/BC update can be checked afterwards using RMg5cli to look at the Rack's LastUpgradeStatus value
    * e.g. `RMg5cli show rack1`
