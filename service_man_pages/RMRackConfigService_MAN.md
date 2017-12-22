Copyright 2017 Dell, Inc. All rights reserverd
 
# RMRackConfigService -- RackManager's Rack Configuration Service

---

## About
The ***RMRackConfigService*** is a systemd integrated service that will apply rack configuration changes across the entire rack,
and perform the neccessary tasks to make sure all utilities and services are in sync with the changes.

***RMRackConfigService*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMRackConfigService.log`

### Dependencies
* RMconfig must have already properly configured all it's subsystems in order for the RMRackConfigService to
  communicate will all the mgt components and services

### Description
* Upon start of ***RMRackConfigService***, first the user conf file `/etc/opt/dell/rm-tools/RackConfig.conf` will be validated
* If the user conf is invalid, a warning will be logged, and the service will continue to use the previously validated conf
* Next, RMRackConfigService will check each setting in the validated conf file to see if has changed from the previous value
* If it is the same, RMRackConfigService will move on to the next setting
* If it has changed, RMRackConfigService will save the new value to data files, and then take the neccessary steps to apply
  that change across the entire rack
  * e.g. Changing RackNumber will result in the RMRackConfigService stopping some RM services, applying the new value to the
    managed MC's configuration, and then restarting the RM services with the new value in place
* All events will be logged to `/var/log/rackmanager/RMRackConfigService.log`

### RackConfig.conf
* `/etc/opt/dell/rm-tools/RackConfig.conf` is the editable user conf file for RMRackConfigService
* Any edits to this file require a restart of RMRackConfigService in order for the new configuration rules to take effect
* The current default conf file looks like this:

```
[RackConfig]
# Configures the RackNumber used by G5MC and RMRedfishService to create IDs - eg. Rack1-Block1-Sled1
# RackNumber is a string of form "1", "2", "3", ...
RackNumber="1"

[PowerAndManagementDomains]
# used to tell RMconfig and RMRedfishSerivce how many G5 Mgt Domains to expect
# only one is supported for RMTK 1.0 and 1.1
PowerDomains="1"

[JbodConfig]
# used to configure the MC Jbod Associations
# G5MC and RMRedfishService use this data
# not used by RMTK 1.0 or 1.1  -- for 1.0 and 1.1, jbod associations must be done in MC config

[Location]
# stores the U location of blocks and 1U servers in the rack
# RMRedfishSerivce uses this data to implement chassis Location structures
# not used by RMTK 1.0 or 1.1
```

### Limitations
* Currently, RackNumber and PowerDomains are the only RackConfig.conf settings supported:
  * RackNumber - can be set to any value between 1 and 9999
  * PowerDomains - currently can only be set to 1
  
## Usage
* ` systemctl {restart, start, stop} RMNodePortMapService`
