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
* RMversion -- displays the RM Toolkit version

