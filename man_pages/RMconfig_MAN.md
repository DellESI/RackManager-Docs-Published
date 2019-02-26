Copyright &copy; 2016-2018 Dell Inc. or its subsidiaries. All rights reserved.

# RMconfig -- RackManager Configuration Utility
---

## About

***RMconfig*** is a BASH utility used to setup (or configure) the RackManager (RM) Toolkit (RMTK) services and utilities (referred to as subsystems) after the toolkit is installed. After installation or update of a RMTK, RMconfig should always be immediately executed in order to ensure all utilities and services are properly setup and configured.

Setup and configuration includes: 
* creating the RMTK network stack (ethernet configs specific to G5 and RM with VLANs, /etc/hosts, and namespaced network stacks) to isolate the internal management network from the external network
* setting up default RM users, groups, and credentials used for communications w/ G5 MCs and G5 Mgt Network Switches
* creating the RM default configs for standard linux services: dhcpd, httpd, sshd, tftp, rsyslog, redis, etc.
* creating the RM default configs for the RMTK utilities/services: RackConfig.conf, RM.conf, mc.conf, redfish.conf, etc.
* configuring linux startup services to start the proper services upon boot
* configuring the G5 MCs and internal Switches

***RMconfig*** will initially configure the "RMbase" subsystem which will make sure that the paths to various utilities are setup, and that a default RM.conf file is created at `/etc/opt/dell/rm-tools/RM.conf` which is required to configure the other subsystems.

Once the "RMbase" subsystem has been configured, ***RMconfig*** by default will read the RM.conf config file to determine which other subsystems to configure (eg dhcpd, httpd, RMRedfishService, RMUsersGroupsPaths, etc.) and for each which configuration profile to use.

However, note that users can run the command with specific options/arguments to configure a single subsystem, or to start, stop, or restart a service for debug (e.g. `RMconfig -s <subsystem> config` or `RMconfig -s <subsystem> restart` ).

Additionally, note that certain subsystem configurations (such as RMMgtNetwork) may require a system reboot in order to fully take affect. If you want the system to auto-reboot following completion of the RMconfig action, you can use the "-R" option.

All events will be logged to `/var/log/rackmanager/RMconfig.log`

## Usage:

```
RMconfig  -V                       -- display version and exit
RMconfig  -h                       -- display overall RMconfig usage and exit
RMconfig  -h -s <subSys>           -- display usage help for a specific subsystem and exit
RMconfig  [-v][-F] config          -- config all subsystems listed in RM.conf using the assigned profiles in RM.conf
RMconfig  [-v][-F][-p <cfgProfile>] -s <subSys> <action>  -- run <action> on specified <subSys>
	Note that if <action>="config" above, -p <cfgProfile> can be used to specify the profile. Otherwise, by
	default the profile specified in RM.conf for the subsystem will be used with config.
	
OPTIONS:
        -h          -- display usage help
        -F          -- force reconfig -- without -F, subsystems previously configured will not configure again
        -s <subSys> -- specify the subsystem - if -s option is used, the <action> must also be specified --
                       if -s <subSys> is not specified, RMconfig runs "config" action on all subsystems 
                       in RM.conf with a valid profile name set other than "None".
	-R          -- will auto-reboot the RM once action is completed.
        -v          -- verbose output - can repeat for additional level of verbosity
              -v    -- verbose level 1 is for high-level debug info from the main script - including progress
              -vv   -- verbose level 2 is for subsystem specific flow progress
              -vvv  -- verbose level 3 is for dumping detailed variable info at any level

<subSys>: one of: { RMbase, sshd, dhcpd, httpd, RMMgtNetwork, RMUsersGroupsPaths, tftp, redis, rsyslog, celery, RMTime,
                    RMG5MCPortMapService, RMNodePortMapService, RMRackConfigService, RMCredentials, RMg5mc, RMRedfishService,
		    RMNodeDiscoveryService, RMPortMonitorService } 

<action>: one of: { config, restart, stop, start }
          note: enter 'RMconfig -h -s <subSys>' to get specific usage and actions supported for that subsystem
```

## Installation, Path, and Dependencies:
* ***RMconfig*** is included in the rackmanager-tools RPM, and is installed by default when a RackManager Toolkit is installed
* The utility and its data is installed at `/opt/dell/rm-tools/RMconfig/*`
* The rackmanager-tools RPM install creates the RMconfig executable script at `/opt/dell/rm-tools/bin/RMconfig`
* The rackmanager-tools RPM install also places `/opt/dell/rm-tools/bin/` in the default bash/sh path for all RM users
* The first line of the program includes a shebang line `#!/usr/bin/bash` that will direct execution using standard linux bash
* RMconfig specific logs can be found at `/var/log/rackmanager/RMconfig.log`

## Examples:

```
   RMconfig -V                   --- prints the version and exits
   
   RMconfig -h                   --- prints help info and exits
   
   RMconfig -vv config           --- runs a full configuration based on the subsystems and config profiles in RM.conf,
                                     also prints some verbose information about the execution process
   
   RMconfig -F -s RMg5mc config  --- does a force config of just the RMg5mc subsystem only using the assigned config
                                     profile in RM.conf
```

## Subsystems managed by RMconfig

* Under RMconfig/, there is a subsystem-specific directory for each subsystem
* Under the subsystem-specific directories, there is a profile-specific directory for each config profile
  with a profile-specific bash config script (`subSystem_config.sh`) and config files for the subsystem/profile
  * RMconfig will use these to setup the default configurations per subsystem/profile
* The current list of subsystems include:
  * RMbase -- sets up default RM.conf config file, RM utility executables, and required initial data files
  * RMUsersGroupsPaths -- creates default rackmanager users, groups, and paths as defined by the specified profile
  * sshd -- sets up sshd service config as defined by the specified profile
  * dhcpd -- sets up dhcpd service config as defined by the specified profile
  * RMMgtNetwork -- sets up RMMgtNetworkStart service config as defined by the specified profile, creates static
      hosts entries, ethernet configs, and namespaced network stacks for mgmt1 and mgmt2
  * httpd -- sets up httpd config as defined by the specified profile
  * RMRedfishService -- sets up RMRedfishService service config as defined by the specified profile
  * RMCredentials -- creates/configures ssh keys and redfish auth credentials for communication with the Managed MC
  * RMg5mc -- configures G5 Managed MCs with ssh keys and required dependent MC configurations
  * tftp -- sets up tftp service config as defined by the specified profile
  * redis -- sets up redis service config as defined by the specified profile
  * rsyslog -- sets up rsyslog service config as defined by the specified profile
  * RMNodeDiscoveryService -- sets up RMNodeDiscoveryService service config as defined by the specified profile
  * celery -- sets up celery service config as defined by the specified profile
  * RMTime -- sets up RMTimeService service config as defined by the specified profile
  * RMG5MCPortMapService -- sets up RMG5MCPortMapService service config as defined by the specified profile
  * RMNodePortMapService -- sets up RMNodePortMapService service config as defined by the specified profile
  * RMRackConfigService -- sets up RMRackConfigService service config as defined by the specified profile
   
### High level description of what lower-level config does for each subsystem (by subsystem)
  * RMbase
    * create the `/etc/opt/dell/rm-tools/` directory with default RM.conf, .RMconfig_times, and .RMg5update_times data files
    * copy's executable wrapper scripts for other rackmanager-tools utilities to the `/opt/dell/rm-tools/bin` dir so that they will be in the user's path
    
  * sshd, dhcpd, httpd, tftp, rsyslog, redis:
    * copy the correct RM config files depending on the specified profile to their proper locations for the service
    * execute the standard linux command for each of these services to configure it to auto start on boot
      * tftp will be disabled by default
    * additional actions that can be specified by targeting the service: start, stop, restart
  
  * RMMgtNetwork:
    * create a default /etc/hosts file with default management network entries (based on profile in RM.conf)
    * create default `/etc/sysconfig/network-scripts/ifcfg-*` files (with proper vlan eth devices) based on specified profile
    * create VLAN eth devices to tunnel the two ext mgt ports to ext ports on the IM switch
    * configure the RM network stack to totally isolates the internal mgt network from external ports using namespaces
    * configure firewall rules to allow access only to the proper services on the various RM subnets
    * configures the RMMgmtPortMonitor service to monitor the status of mgmt1/mgmt2

  * RMUsersGroupsPaths
    * add the default RM Linux OS groups for admin, operator, and readonly roles (RM_ADMIN, RM_OPER, and RM_READONLY)
    * add the default RM Linux OS users for each role (rackmanager_adm, rackmanager_oper, rackmanager_readonly)
    * setup the default path for all users to pickup `/opt/dell/rm-tools/bin`
    * NOTE: Only Linux OS users will be created and configured by this subcommand.  Redfish Service users are managed in the RMRedfishService AccountsDB.

  * RMCredentials
    * generate ssh keys and ssh config files for use when communicating with the MCs using ssh passwordless authentication
    * configure the RM credential vault w/ proper credentials that RMg5mc can use to communicate w/ managed MCs
    * configure the RM credential vault w/ proper credentials that RM utilities and services can use to communicate w/ iDracs
    * configure the RM credential vault w/ proper credentials that RM utilities and services can use to communicate w/ RM Management Network Switches
    * NOTE: only Linux OS users' credentials are configured by this subcommand

  * RMg5mc
    * scp the ssh public key files used by RMg5cli to the the Managed MCs
    * copy the required MC configuration files to the Managed MC
    * reset the MC for these changes to take effect
    
  * RMRedfishService
    * copy the Redfish.conf file indicated by the specified profile to `/etc/opt/dell/rm-tools/Redfish.conf`
    * copy the RedDrum.conf file indicated by the specified profile to `/etc/opt/dell/rm-tools/RedDrum.conf`
    * setup Redfish user accounts in the AccountsDB with default passwords if the AccountsDB does not already exist
    * config Linux boot script to auto start RMRedfishService
    
  * RMNodeDiscoveryService
    * config Linux boot script to auto start RMNodeDiscoveryService
    
  * RMG5MCPortMapService
    * create default RMG5MCPortMap.conf in `/etc/opt/dell/rm-tools/`
    * create default service data files in `/var/opt/dell/rm-tools/`
    * config Linux boot script to auto start RMG5MCPortMapService
    
  * RMNodePortMapService
    * create default RMNodePortMap.conf in `/etc/opt/dell/rm-tools/`
    * create default service data files in `/var/opt/dell/rm-tools/`
    * config Linux boot script to auto start RMNodePortMapService
  
  * RMRackConfigService
    * create default RackConfig.conf in `/etc/opt/dell/rm-tools/`
    * create default service data files in `/var/opt/dell/rm-tools/`
    * config Linux boot script to auto start RMRackConfigService
  
  * celery
    * config Linux boot script to auto start celery service
  
## Limitations:
* No known limitations

## Known Issues:
* A full config of all subsystems can sometimes take several minutes, but should not take more than 15 minutes  


