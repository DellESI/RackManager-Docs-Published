Copyright 2017 Dell, Inc. All rights reserverd
 
# RMNodeDiscoveryService -- RackManager's BMC Node Discovery Service

---

## About
The ***RMNodeDiscoveryService*** is a systemd integrated service that discovers BMC nodes, and creates associated hosts entries.

***RMNodeDiscoveryService*** specific logs can be found on the RackManager: `/var/log/rackmanager/RMNodeDiscoveryService.log`

### Dependencies
* RMRedfishService
  * ***RMNodeDiscoveryService*** uses RMRedfishService's collection of nodes to determine which nodes exists.
  * If RMRedfishService is not running, or has failed, the `/etc/hosts` will not reflect any nodes existing.

### Description
* Upon start of ***RMNodeDiscoveryService***, a periodic poll will begin at a default rate of 30 seconds.
* At each poll, ***RMNodeDiscoveryService*** will pull the current collection of nodes from RMRedfishService.
* ***RMNodeDiscoveryService*** will then compare this information with the list of nodes in `/etc/hosts`.
* Any new nodes will be added to the end of the `/etc/hosts` file with names and aliases that map to the rack, block, and slot:
```
10.253.4.44    Rack1-Block1-Sled2-Node1    node-1.1.2   R1-B1-S2
10.253.4.42    Rack1-Block1-Sled1-Node1    node-1.1.1   R1-B1-S1
10.253.4.43    Rack1-Block2-Sled1-Node1    node-1.2.1   R1-B2-S1
10.253.4.41    Rack1-Block3-Sled3-Node1    node-1.3.3   R1-B3-S3
```
* Any nodes that were replaced with a new physical node will be updated with the correct IP address.
* Any missing nodes will be removed from the /etc/hosts file if they no longer exists.
* All events including additons, updates, and subtractions will be logged.

### Current Limitation:
* None

## Usage
* ` systemctl {restart, start, stop} RMNodeDiscoveryService`
