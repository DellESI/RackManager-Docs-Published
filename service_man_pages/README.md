Copyright &copy; 2018 Dell Inc. or its subsidiaries. All rights reserved.

# RackManager Toolkit:  Service MAN Pages
This folder contains MAN pages for ***Services*** that are implemented as part of the RackManager Toolkit:
* These services are integrated with systemd and configured by RMconfig
* Users generally should not need to modify them
* Typically, these services should be managed using systemd APIs as described in the MAN pages here
* The MAN pages herein describe any configuration settings, interaction and dependencies with other services, and systemd commands 
used to start, stop, restart, etc the services.

# The RackManager Toolkit Services include:
  * RMRedfishService -- a rack-level implementation of the industry standard Redfish RESTful hardware management API. The RMRedfishService runs behind the Apache httpd (as either a reverse proxy or using the Apache mod-wsgi). The service provides:
    * a rack-level Redfish service implementation from which one can manage the entire rack
    * caches that select data for speed
    * node-specific data from the sled BMCs directly over the internal management network
    * chassis, fan and power data from the managed MC over the internal management network
  * RMNodeDiscoveryService -- discovers BMC nodes and creates hosts entries with names that map to block.slot
  * RMMgmtPortMonitor -- an internal service that monitors the state of ports on the internal management network as required based on the network topology
    * The key feature is monitoring the status of the Mgmt1 and Mgmt2 external links that connect to RM via VLANS, so if the link ever drops, the RM will know to re-bringup the link
  * RMTimeService -- used to get localtime from the DSS9000 MC if RM has lost its localtime due to a poweroff.
    * When RM starts if it has lost localtime, it re-initializes its localtime from the MC
    * it routinely synces its RM localtim to the MC so that the MC localtime if valid
    * Note that timezine is not synced, but timezone on the MC is not generally visable to  a user
  * RMMgtNetworkStart -- is a script used to startup the namespaced network stacks on the RM.  It is not a full service.
    * this subsystem is called by systemd as part of normal network start so that the namespaced network stacks on RM used to implement the vlan tunnel from RM to the physical Mgmt1 and Mgmt2 ports on the DSS9000 IM switch is configured correctly.

# MAN Pages (***coming soon***)
  * RMRedfishService  `-- see ./RMRedfishService_MAN.md`
  * RMNodeDiscoveryService  `-- see ./RMNodeDiscoveryService_MAN.md`
  * RMMgmtPortMonitor `-- see ./RMMgmtPortMonitor_MAN.md`
  * RMMgtNetworkStart `-- see ./RMMgtNetworkStart_MAN.md`
  * RMTimeService  `--see ./RMTimeService_MAN.md`
