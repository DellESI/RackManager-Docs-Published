Copyright &copy; 2017-2018 Dell Inc. or its subsidiaries. All rights reserved.
 
# RMMgmtPortMonitor -- RackManager's Management Port Monitoring Service

---

## About
The ***RMMgmtPortMonitor*** is a systemd integrated service that monitors the state of external facing mgmt links on the
management network as required based on the network topology.

The key feature of the service monitors the status of the Mgmt1 and Mgmt2 external links, which connect internally to network
interfaces on the RackManager. As the Mgmt1 and Mgmt2 links go UP and DOWN, ***RMMgmtPortMonitor*** will detect the
changes, and make the neccessary required changes to the internal network interfaces.

***RMMgmtPortMonitor*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMMgmtPortMonitor.log`

### Dependencies
* RMMgtNetworkStart
  * ***RMMgmtPortMonitor*** uses RMMgtNetworkStart's functionality to apply specific namespace networking changes.
* Communication with the Interconnect Module (IM)


### Description
* Upon start of ***RMMgmtPortMonitor***, there is an initial delay of a default of 10 seconds.
* A periodic poll will then begin at a default rate of 20 seconds.
* At each poll, ***RMMgmtPortMonitor*** will first determine if mgmt1 and mgmt2 are RMMgmtPortMonitor controlled.
  * This is determined by the `RM_PORTMONITOR_CONTROLLED` field in the mgmt interface network scripts.
  * To turn off this port monitor feature for mgmt1 or mgmt2, set `RM_PORTMONITOR_CONTROLLED=no` in the following files:
    * mgmt1 --> /etc/sysconfig/network-scripts/ifcfg-enp1s0.4021
    * mgmt2 --> /etc/sysconfig/network-scripts/ifcfg-enp1s0.4022
* If the mgmt port is RMMgmtPortMonitor controlled, ***RMMgmtPortMonitor*** will then determine the state of the external link.
* If the external link just went down, ***RMMgmtPortMonitor*** will down the internal interface and arm it for the next link up.
* If the external link just went up, ***RMMgmtPortMonitor*** will up the internal interface and establish the internal management network.
* All events and state changes will be logged.

### Current Limitation:
* None

## Usage
* ` systemctl {restart, start, stop} RMMgmtPortMonitor`
