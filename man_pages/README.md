Copyright &copy; 2018 Dell Inc. or its subsidiaries. All rights reserved.

# RackManager Toolkit:  utility command MAN Pages
This folder contains MAN pages for ***Utility Commands*** that are implemented as part of the RackManager Toolkit:

## RMTK provides several utilities developed for RackManager:
* RMconfig -- a basic utility to setup and configure the RackManager Toolkit: 
  * includes creating default RM Users: rackmanager_adm, rackmanager_oper, rackmanager_readonly
  * includes creating default RM permission groups:  RM_ADMIN, RM_OPER, RM_READONLY
  * sets-up default RMTK config files for sshd, dhcpd, httpd, etc., and puts the path to the RMTK utilities in the standard shell path
  * creates the network stack that allows the RM to communicate with the internal management network isolated from the external Ethernet interfaces.
* RMg5update -- a utility to update DSS9000 infrastructure firmware: MCs, IM, BC, and G5 Switches
* RMg5cli -- a utility to connect to the DSS9000 MC and execute legacy G5 CLI commands
* RMredfishtool -- a version of redfishtool CLI that is optimized for the RackManager toolkit on DSS9000
* RMbiosupdate -- a utility to update the Stark RackManager's BIOS ROM boot firmware
* RMadmin -- provides several helpful debug and admin subcommands
* RMversion -- displays the RM Toolkit version


## RackManager Toolkit specific Utilities:
* RMversion     `-- see /man_pages/RMversion_MAN.md`
* RMconfig      `-- see /man_pages/RMconfig_MAN.md`
* RMg5cli       `-- see /man_pages/RMg5cli_MAN.md`
* RMredfishtool `-- see /man_pages/RMredfishtool.md`
* RMg5update    `-- see /man_pages/RMg5update_MAN.md`
* RMbiosupdate  `-- see /man_pages/RMbiosupdate_MAN.md`
* RMadmin       `-- see /man_pages/RMadmin_MAN.md`

